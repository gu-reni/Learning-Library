## `<linux/userfaultfd.h>` 头文件详解（Linux 内核接口）

`<linux/userfaultfd.h>` 是 Linux 内核提供的用户态缺页故障处理（userfaultfd）机制的 UAPI 头文件，定义了该系统调用的相关结构体、ioctl 命令和标志位。它允许用户态程序接管进程的缺页异常，实现自定义的按需分页、零拷贝迁移、虚拟化内存管理等高级功能[reference:0]。

---

## 一、函数详解

### 1. userfaultfd 系统调用

**函数原型：**
```c
int userfaultfd(int flags);
```

**作用：** 创建一个 userfaultfd 对象，返回一个文件描述符，该描述符可用于后续的 ioctl 配置和事件读取[reference:1]。

**参数：**
- `flags`：可取 `0` 或以下值的按位或：
  - `UFFD_USER_MODE_ONLY`：该标志确保 userfaultfd 只能在用户模式下运行，这对于某些安全场景很有用（Linux 5.11+）[reference:2]。

**返回值：** 成功返回非负文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <linux/userfaultfd.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int uffd = syscall(SYS_userfaultfd, 0);
    if (uffd == -1) {
        perror("userfaultfd");
        return 1;
    }
    // ... 配置 uffd
    close(uffd);
    return 0;
}
```

**实现原理：**
系统调用进入内核，创建一个 `userfaultfd_ctx` 上下文对象，并返回一个匿名文件描述符。该上下文关联一个等待队列，用于存放因缺页而阻塞的线程。内核在缺页异常处理路径中检查 VMA 是否注册了 userfaultfd，若是则将事件放入队列并唤醒用户态处理线程[reference:3]。

**线程安全提示：**
`userfaultfd()` 系统调用本身是线程安全的。但返回的文件描述符在多线程间共享时，需要用户态自行同步。

---

### 2. 通过 `read()` 读取事件

**函数原型：**
```c
ssize_t read(int fd, void *buf, size_t count);
```

**作用：** 从 userfaultfd 文件描述符读取 `struct uffd_msg` 事件，阻塞直到有事件发生（除非设置了非阻塞）。

**示例用法：**
```c
struct uffd_msg msg;
ssize_t n = read(uffd, &msg, sizeof(msg));
if (n == sizeof(msg)) {
    // 处理缺页事件
}
```

**实现原理：**
当注册区域发生缺页时，内核构建 `uffd_msg` 结构体并放入事件队列。`read()` 将队列中的数据拷贝到用户空间，并唤醒阻塞的缺页线程（等待用户态通过 ioctl 解决缺页后）。

---

### 3. 通过 `ioctl()` 配置和响应

`<linux/userfaultfd.h>` 定义了多个 ioctl 命令，用于配置 userfaultfd 对象、注册/注销内存区域、响应缺页事件。

**函数原型：**
```c
int ioctl(int fd, unsigned long request, ...);
```

**常用 ioctl 命令：**
- `UFFDIO_API`：启用 userfaultfd 并进行 API 版本协商。
- `UFFDIO_REGISTER`：注册一个内存区域，告诉内核哪些地址范围的缺页需要交给用户态处理。
- `UFFDIO_UNREGISTER`：注销一个已注册的内存区域。
- `UFFDIO_COPY`：将用户态提供的数据拷贝到缺页地址，完成缺页处理。
- `UFFDIO_ZEROPAGE`：将缺页地址填充为零页。
- `UFFDIO_CONTINUE`：用于 `UFFD_FEATURE_MINOR_HUGETLBFS` 特性，继续处理次缺页。

**示例用法：**
```c
// 注册内存区域
struct uffdio_register reg = {
    .range.start = (unsigned long)addr,
    .range.len = len,
    .mode = UFFDIO_REGISTER_MODE_MISSING,
};
ioctl(uffd, UFFDIO_REGISTER, &reg);

// 处理缺页事件
struct uffdio_copy copy;
copy.dst = (unsigned long)msg.arg.pagefault.address;
copy.src = (unsigned long)page_data;
copy.len = page_size;
copy.mode = 0;
ioctl(uffd, UFFDIO_COPY, &copy);
```

**实现原理：**
- `UFFDIO_REGISTER`：内核在指定 VMA 上设置标志，标记该区域由 userfaultfd 接管。
- `UFFDIO_COPY` / `UFFDIO_ZEROPAGE`：内核将数据拷贝到缺页地址或填充零页，然后唤醒阻塞的缺页线程。
- `UFFDIO_CONTINUE`：用于次缺页处理，通知内核该页已经可用。

**线程安全提示：**
对同一 `uffd` 文件描述符的并发 ioctl 调用需要用户态同步。

---

## 二、结构体详解

### 1. struct uffd_msg

**定义：**
```c
struct uffd_msg {
    __u8  event;        // 事件类型，如 UFFD_EVENT_PAGEFAULT
    __u8  reserved1;
    __u16 reserved2;
    __u32 reserved3;
    union {
        struct {
            __u64 flags;        // 缺页标志，如 UFFD_PAGEFAULT_FLAG_WRITE
            __u64 address;      // 发生缺页的地址
        } pagefault;
        // ... 其他事件类型
    } arg;
};
```

**作用：** 描述一个 userfaultfd 事件，通过 `read()` 从文件描述符读取得到[reference:4]。

**成员详解：**
- `event`：事件类型，当前主要有 `UFFD_EVENT_PAGEFAULT`（缺页事件）。
- `arg.pagefault.flags`：缺页类型标志，可取：
  - `UFFD_PAGEFAULT_FLAG_WRITE`：写缺页。
  - `UFFD_PAGEFAULT_FLAG_WP`：写保护缺页（当注册了 `UFFDIO_REGISTER_MODE_WP` 时）。
  - `UFFD_PAGEFAULT_FLAG_MINOR`：次缺页。
- `arg.pagefault.address`：触发缺页的虚拟地址。

---

### 2. struct uffdio_api

**定义：**
```c
struct uffdio_api {
    __u64 api;          // 请求的 API 版本（输入）
    __u64 features;     // 请求/返回的特性掩码
    __u64 ioctls;       // 内核支持的 ioctl 位掩码（输出）
};
```

**作用：** 用于 `UFFDIO_API` ioctl，进行 API 版本协商和特性查询[reference:5]。

**成员详解：**
- `api`：输入，必须设为 `UFFD_API`（值为 0xAA）。
- `features`：输入/输出，位掩码指定请求的特性（如 `UFFD_FEATURE_EVENT_FORK`），内核返回实际支持的特性。
- `ioctls`：输出，内核返回支持的操作位掩码，用户程序应据此检查可用功能。

---

### 3. struct uffdio_range

**定义：**
```c
struct uffdio_range {
    __u64 start;    // 起始地址
    __u64 len;      // 长度（字节）
};
```

**作用：** 描述一个内存地址范围，用于 `UFFDIO_REGISTER`、`UFFDIO_UNREGISTER` 等操作[reference:6]。

---

### 4. struct uffdio_register

**定义：**
```c
struct uffdio_register {
    struct uffdio_range range;   // 要注册的内存范围
    __u64 mode;                  // 注册模式，如 UFFDIO_REGISTER_MODE_MISSING
    __u64 ioctls;                // 该区域支持的 ioctl 操作位掩码（输出）
};
```

**作用：** 用于 `UFFDIO_REGISTER` ioctl，注册一个内存区域[reference:7]。

**成员详解：**
- `range`：指定要注册的地址范围。
- `mode`：注册模式，可取 `UFFDIO_REGISTER_MODE_MISSING`（缺页）、`UFFDIO_REGISTER_MODE_WP`（写保护）、`UFFDIO_REGISTER_MODE_MINOR`（次缺页）。
- `ioctls`：输出，内核返回该区域支持的 ioctl 操作掩码（如 `1ULL << _UFFDIO_COPY`）。

---

### 5. struct uffdio_copy / uffdio_zeropage / uffdio_continue

**定义：**
```c
struct uffdio_copy {
    __u64 dst;      // 目标地址（缺页地址）
    __u64 src;      // 源地址（用户态数据缓冲区）
    __u64 len;      // 拷贝长度
    __u64 mode;     // 模式，可取 0 或 UFFDIO_COPY_MODE_DONTWAKE
    __s64 copy;     // 实际拷贝的字节数（输出）
};

struct uffdio_zeropage {
    __u64 dst;      // 目标地址
    __u64 len;      // 长度
    __u64 mode;     // 模式
    __s64 zeropage; // 实际处理的字节数（输出）
};

struct uffdio_continue {
    __u64 range_start;  // 起始地址
    __u64 len;          // 长度
    __u64 mode;         // 模式
    __s64 mapped;       // 实际映射的字节数（输出）
};
```

**作用：** 分别用于 `UFFDIO_COPY`、`UFFDIO_ZEROPAGE`、`UFFDIO_CONTINUE` ioctl，解决缺页事件[reference:8][reference:9]。

**成员详解：**
- `mode`：可取 `0` 或 `UFFDIO_COPY_MODE_DONTWAKE`（不唤醒缺页线程，需后续手动唤醒）。

---

## 三、宏定义详解

### 1. 系统调用标志

| 宏名称 | 作用 |
|--------|------|
| `UFFD_USER_MODE_ONLY` | 限制 userfaultfd 只能在用户模式下运行（Linux 5.11+） |

---

### 2. ioctl 命令

| 宏名称 | 作用 |
|--------|------|
| `UFFDIO` | ioctl 命令的魔数（0xAA）[reference:10] |
| `UFFDIO_API` | API 版本协商[reference:11] |
| `UFFDIO_REGISTER` | 注册内存区域[reference:12] |
| `UFFDIO_UNREGISTER` | 注销内存区域 |
| `UFFDIO_COPY` | 拷贝数据到缺页地址[reference:13] |
| `UFFDIO_ZEROPAGE` | 填充零页[reference:14] |
| `UFFDIO_CONTINUE` | 继续处理次缺页[reference:15] |
| `UFFDIO_WRITEPROTECT` | 写保护操作 |
| `UFFDIO_POISON` | 毒化页面（Linux 6.6+） |

---

### 3. API 常量

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `UFFD_API` | 0xAA | userfaultfd API 版本号[reference:16] |
| `UFFD_API_FEATURES` | (UFFD_FEATURE_...) | 当前内核支持的所有特性位掩码 |

---

### 4. 注册模式（mode）

| 宏名称 | 作用 |
|--------|------|
| `UFFDIO_REGISTER_MODE_MISSING` | 缺页模式：捕获缺页异常 |
| `UFFDIO_REGISTER_MODE_WP` | 写保护模式：捕获写保护异常 |
| `UFFDIO_REGISTER_MODE_MINOR` | 次缺页模式：捕获次缺页异常[reference:17] |

---

### 5. 特性标志（features）

| 宏名称 | 作用 |
|--------|------|
| `UFFD_FEATURE_PAGEFAULT_FLAG_WP` | 支持写保护缺页标志 |
| `UFFD_FEATURE_EVENT_FORK` | 支持 `fork` 事件通知 |
| `UFFD_FEATURE_EVENT_REMAP` | 支持 `remap` 事件通知 |
| `UFFD_FEATURE_EVENT_REMOVE` | 支持 `remove` 事件通知 |
| `UFFD_FEATURE_MISSING_HUGETLBFS` | 支持大页缺页处理 |
| `UFFD_FEATURE_MISSING_SHMEM` | 支持共享内存缺页处理 |
| `UFFD_FEATURE_SIGBUS` | 支持 `SIGBUS` 信号通知 |
| `UFFD_FEATURE_THREAD_ID` | 在缺页事件中报告线程 ID |
| `UFFD_FEATURE_MINOR_HUGETLBFS` | 支持大页次缺页处理 |
| `UFFD_FEATURE_MINOR_SHMEM` | 支持共享内存次缺页处理 |
| `UFFD_FEATURE_EXACT_ADDRESS` | 报告精确的缺页地址 |
| `UFFD_FEATURE_WP_HUGETLBFS_SHMEM` | 支持大页/共享内存写保护 |

---

### 6. 事件类型（event）

| 宏名称 | 作用 |
|--------|------|
| `UFFD_EVENT_PAGEFAULT` | 缺页事件 |
| `UFFD_EVENT_FORK` | 子进程 fork 事件 |
| `UFFD_EVENT_REMAP` | 内存重映射事件 |
| `UFFD_EVENT_REMOVE` | 内存区域移除事件 |
| `UFFD_EVENT_UNMAP` | 内存区域解除映射事件 |

---

### 7. 缺页标志（flags）

| 宏名称 | 作用 |
|--------|------|
| `UFFD_PAGEFAULT_FLAG_WRITE` | 写缺页 |
| `UFFD_PAGEFAULT_FLAG_WP` | 写保护缺页 |
| `UFFD_PAGEFAULT_FLAG_MINOR` | 次缺页 |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `struct uffd_msg` | 事件消息结构体 |
| `struct uffdio_api` | API 参数结构体 |
| `struct uffdio_range` | 地址范围结构体 |
| `struct uffdio_register` | 注册参数结构体 |
| `struct uffdio_copy` | 拷贝参数结构体 |
| `struct uffdio_zeropage` | 零页参数结构体 |
| `struct uffdio_continue` | 继续参数结构体 |
| `struct uffdio_writeprotect` | 写保护参数结构体 |
| `struct uffdio_poison` | 毒化参数结构体 |

---

## 五、模板声明

`<linux/userfaultfd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
