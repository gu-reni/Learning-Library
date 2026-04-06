## `<linux/seccomp.h>` 头文件详解（Linux 内核接口）

`<linux/seccomp.h>` 是 Linux 内核提供的头文件，定义了 seccomp（secure computing mode）相关的常量、结构体和操作命令。seccomp 允许进程限制自身可调用的系统调用，增强安全性。该头文件配合 `prctl()` 或 `seccomp()` 系统调用使用。

---

## 一、函数详解

### 1. seccomp 系统调用

**函数原型：**
```c
int seccomp(unsigned int operation, unsigned int flags, void *args);
```

**作用：** 设置进程的 seccomp 过滤模式。

**参数：**
- `operation`：操作命令（`SECCOMP_SET_MODE_STRICT` 或 `SECCOMP_SET_MODE_FILTER`）。
- `flags`：标志位，通常为 0。
- `args`：指向操作参数的指针（如 `struct sock_fprog *`）。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1，并设置 `errno`。

**示例用法（加载 BPF 过滤器）：**
```c
#include <linux/seccomp.h>
#include <sys/syscall.h>
#include <unistd.h>

struct sock_fprog prog = {
    .len = 1,
    .filter = (struct sock_filter[]){ BPF_STMT(BPF_RET+BPF_K, SECCOMP_RET_ALLOW) }
};
syscall(SYS_seccomp, SECCOMP_SET_MODE_FILTER, 0, &prog);
```

**实现原理：**
系统调用。内核根据 `operation` 设置进程的 seccomp 模式。`SECCOMP_SET_MODE_STRICT` 仅允许 `read`、`write`、`_exit`、`sigreturn` 等少数系统调用；`SECCOMP_SET_MODE_FILTER` 则加载用户提供的 BPF 过滤器，对每次系统调用进行裁决。

**线程安全提示：**
`seccomp()` 系统调用影响整个进程，所有线程共享相同的过滤器。在已有线程的情况下安装过滤器需谨慎，通常应在创建任何线程之前设置。

---

## 二、结构体详解

### 1. struct sock_fprog

（通常来自 `<linux/filter.h>`，但 seccomp 使用相同结构）
```c
struct sock_fprog {
    unsigned short      len;     // 指令数量
    struct sock_filter *filter;  // BPF 指令数组
};
```

**作用：** 描述一个 BPF 过滤器程序，用于 seccomp 过滤。

### 2. struct seccomp_data

**定义：**
```c
struct seccomp_data {
    int   nr;              // 系统调用号
    __u32 arch;           // 体系结构
    __u64 instruction_pointer; // 触发系统调用的指令地址
    __u64 args[6];        // 系统调用参数
};
```

**作用：** 当使用 `SECCOMP_RET_TRACE` 或 `SECCOMP_RET_USER_NOTIF` 时，内核将该结构体传递给跟踪进程或用户态通知机制，提供系统调用的详细信息。

---

## 三、宏定义详解

### 1. seccomp 操作命令（operation）

| 宏名称 | 作用 |
|--------|------|
| `SECCOMP_SET_MODE_STRICT` | 仅允许 `read`、`write`、`_exit`、`sigreturn` 系统调用（传统严格模式） |
| `SECCOMP_SET_MODE_FILTER` | 使用 BPF 过滤器定义允许的系统调用 |
| `SECCOMP_GET_ACTION_AVAIL` | 查询某个 action 是否可用（用于检查内核支持） |
| `SECCOMP_GET_NOTIF_SIZES` | 获取通知结构体大小 |

### 2. seccomp 过滤器返回值（action）

这些值是 BPF 过滤器的返回值，决定内核如何处理系统调用。

| 宏名称 | 作用 |
|--------|------|
| `SECCOMP_RET_KILL` | 立即终止进程（默认 kill thread） |
| `SECCOMP_RET_KILL_PROCESS` | 终止整个进程（Linux 4.14+） |
| `SECCOMP_RET_KILL_THREAD` | 终止当前线程（Linux 4.14+） |
| `SECCOMP_RET_TRAP` | 向进程发送 `SIGSYS` 信号 |
| `SECCOMP_RET_ERRNO` | 返回错误码（errno），不执行系统调用 |
| `SECCOMP_RET_USER_NOTIF` | 通知用户态监听进程处理（Linux 5.0+） |
| `SECCOMP_RET_TRACE` | 通知 ptrace 跟踪进程 |
| `SECCOMP_RET_LOG` | 记录日志但允许执行（Linux 4.14+） |
| `SECCOMP_RET_ALLOW` | 允许系统调用 |

### 3. seccomp 标志位（flags）

| 宏名称 | 作用 |
|--------|------|
| `SECCOMP_FILTER_FLAG_TSYNC` | 同步所有线程的过滤器（确保所有线程都应用同一过滤器） |
| `SECCOMP_FILTER_FLAG_LOG` | 启用日志记录（记录所有被过滤的系统调用） |
| `SECCOMP_FILTER_FLAG_SPEC_ALLOW` | 允许 CPU 推测执行（禁用 Spectre 缓解） |
| `SECCOMP_FILTER_FLAG_NEW_LISTENER` | 创建新的监听 fd（用于用户通知） |
| `SECCOMP_FILTER_FLAG_TSYNC_ESRCH` | `TSYNC` 时若线程不存在则返回 `ESRCH`（Linux 5.5+） |
| `SECCOMP_FILTER_FLAG_WAIT_KILLABLE_RECV` | 使 `SECCOMP_RET_USER_NOTIF` 等待可被信号中断（Linux 5.19+） |

### 4. 通知相关常量（用于 `SECCOMP_IOCTL_NOTIF_*`）

| 宏名称 | 作用 |
|--------|------|
| `SECCOMP_IOCTL_NOTIF_RECV` | 接收用户通知 |
| `SECCOMP_IOCTL_NOTIF_SEND` | 发送响应 |
| `SECCOMP_IOCTL_NOTIF_ID_VALID` | 验证通知 ID 是否有效 |

---

## 四、类型定义

- `struct seccomp_data`：系统调用详细信息结构。
- `struct seccomp_notif`：用户通知事件结构（定义于 `<linux/seccomp.h>`，包含 `id`、`pid`、`data` 等字段）。
- `struct seccomp_notif_resp`：用户通知响应结构（包含 `id`、`error`、`val`、`flags`）。
- `struct seccomp_notif_sizes`：用于获取通知结构体大小。

---

## 五、模板声明

`<linux/seccomp.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
