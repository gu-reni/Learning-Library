## `<linux/openat2.h>` 头文件详解（Linux 内核接口）

`<linux/openat2.h>` 是 Linux 内核提供的头文件，定义了 `openat2()` 系统调用及其参数结构体 `struct open_how`。`openat2()` 是 `openat()` 的扩展版本，允许更精细地控制文件打开行为（如禁止路径遍历、强制只读等）。该接口是 Linux 5.6+ 引入的，属于 Linux 特有接口。

---

## 一、函数详解

### 1. openat2 系统调用

**函数原型：**
```c
int openat2(int dirfd, const char *pathname, struct open_how *how, size_t size);
```

**作用：** 打开或创建文件，提供比 `openat()` 更丰富的控制选项，通过 `struct open_how` 结构体传递标志和模式。

**参数：**
- `dirfd`：目录文件描述符（与 `openat` 相同，可以是 `AT_FDCWD`）。
- `pathname`：文件路径。
- `how`：指向 `struct open_how` 的指针，包含打开标志、文件模式、解析选项等。
- `size`：`sizeof(struct open_how)`，用于内核兼容性检查。

**返回值：**
- 成功：返回新文件描述符。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
#define _GNU_SOURCE
#include <sys/syscall.h>
#include <linux/openat2.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    struct open_how how = {
        .flags = O_RDONLY | O_NOFOLLOW,
        .mode = 0,
        .resolve = RESOLVE_NO_SYMLINKS | RESOLVE_IN_ROOT,
    };
    int fd = syscall(SYS_openat2, AT_FDCWD, "/etc/passwd", &how, sizeof(how));
    if (fd == -1) perror("openat2");
    else close(fd);
    return 0;
}
```

**实现原理：**
`openat2()` 是系统调用，内核解析 `struct open_how` 中的标志和解析选项，在路径查找过程中应用限制（如禁止符号链接、限制在某个根目录下等），比普通 `open` 更安全。

**线程安全提示：**
系统调用本身是线程安全的。但文件描述符的管理需要用户同步。

---

## 二、结构体详解

### struct open_how

**定义：**
```c
struct open_how {
    __u64 flags;    // 打开标志（如 O_RDONLY, O_CREAT, O_CLOEXEC 等）
    __u64 mode;     // 文件模式（当 O_CREAT 或 O_TMPFILE 时有效）
    __u64 resolve;  // 路径解析标志（RESOLVE_*）
};
```

**作用：** 向 `openat2()` 传递打开参数。

**成员详解：**
- `flags`：与 `openat()` 的 `flags` 相同，支持所有标准打开标志，此外还支持 `O_TMPFILE` 等。
- `mode`：文件权限模式（如 `0644`），仅在 `flags` 包含 `O_CREAT` 或 `O_TMPFILE` 时使用。
- `resolve`：路径解析控制标志，用于限制符号链接、魔法链接、路径遍历等。

---

## 三、宏定义详解

### 1. resolve 字段标志

| 宏名称 | 作用 |
|--------|------|
| `RESOLVE_NO_SYMLINKS` | 禁止解析符号链接，遇到符号链接返回错误 |
| `RESOLVE_NO_MAGICLINKS` | 禁止解析 `/proc/self/fd` 等魔法链接 |
| `RESOLVE_NO_XDEV` | 禁止跨越不同文件系统 |
| `RESOLVE_CACHED` | 仅使用缓存中的路径（不进行磁盘查找），若不在缓存则失败 |
| `RESOLVE_BENEATH` | 限制所有路径在 `dirfd` 之下（类似于 `openat2` 的沙箱） |
| `RESOLVE_IN_ROOT` | 将 `dirfd` 视为根目录，所有路径必须在其下方 |

这些标志可以按位或组合使用，用于实现安全文件打开（如沙箱、容器场景）。

### 2. 其他常量

- `OPEN_HOW_SIZE_VER0`：`struct open_how` 的初始版本大小，用于 `size` 参数。

---

## 四、类型定义

- `struct open_how`：打开参数结构体。

---

## 五、模板声明

`<linux/openat2.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
