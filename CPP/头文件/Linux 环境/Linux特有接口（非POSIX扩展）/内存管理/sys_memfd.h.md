## `<sys/memfd.h>` 头文件详解（Linux 特有接口）

`<sys/memfd.h>` 是 Linux 特有的头文件，定义了 `memfd_create()` 系统调用的相关标志。该系统调用用于创建一个**内存中的匿名文件**，并返回一个指向它的文件描述符[reference:0]。该文件的行为类似于普通文件（可修改、截断、内存映射等），但其存储空间位于 RAM 中，一旦没有进程再引用它，便会自动释放[reference:1][reference:2]。这使其非常适合用于共享内存、进程间通信以及高性能的临时数据处理。

---

## 一、函数详解

### 1. `memfd_create`

**函数原型：**
```c
int memfd_create(const char *name, unsigned int flags);
```

**作用：** 创建一个匿名文件，并返回一个指向它的文件描述符[reference:3]。该文件描述符默认同时支持读写 (`O_RDWR`) 和 `O_LARGEFILE` 标志[reference:4]。默认情况下，其初始大小为 0，需要通过 `ftruncate()` 或 `write()` 等系统调用来设置其大小[reference:5]。

**参数：**
- `name`：一个用于调试的名称字符串。它会在 `/proc/self/fd/` 目录下以 `memfd:` 为前缀的符号链接目标中显示，不影响文件描述符的行为，多个文件可以同名[reference:6][reference:7]。
- `flags`：一个位掩码，用于修改 `memfd_create()` 的行为。可以是下面宏定义中常量的按位或组合（详见“宏定义详解”）。

**返回值：**
- 成功：返回一个非负的文件描述符[reference:8]。
- 失败：返回 `-1`，并设置 `errno` 以指示错误类型[reference:9]。

**示例用法：**
```c
#define _GNU_SOURCE
#include <sys/memfd.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main() {
    // 创建一个内存文件，允许对其进行密封操作，并设置 close-on-exec 标志
    int fd = memfd_create("my_shared_memory", MFD_ALLOW_SEALING | MFD_CLOEXEC);
    if (fd == -1) {
        perror("memfd_create");
        return 1;
    }
    // ... 对文件描述符进行操作 ...
    close(fd);
    return 0;
}
```

**实现原理：**
`memfd_create()` 是一个系统调用，于 Linux 内核 3.17 版本中引入[reference:10]。其核心是创建一个与用户可见的挂载点无关的匿名文件，该文件由 tmpfs 提供存储支持[reference:11]。其设计初衷之一是为了与 `fcntl(2)` 提供的文件密封（file-sealing）API 配合使用，以实现安全的共享内存，防止共享内存区域被意外或恶意地调整大小[reference:12]。

**线程安全提示：**
该系统调用是线程安全的，其创建的文件描述符与普通文件描述符一样，遵循标准的 POSIX 语义，例如在 `fork()` 后被子进程继承[reference:13]。

---

## 二、宏定义详解

`<sys/memfd.h>` 头文件主要定义了 `memfd_create()` 函数的 `flags` 参数及与巨页相关的位掩码常量[reference:14]。若需使用巨页，还需包含 `<asm-generic/hugetlb_encode.h>` 头文件。

| 宏名称 | 作用 |
|--------|------|
| `MFD_CLOEXEC` | 为新创建的文件描述符设置 close-on-exec (`FD_CLOEXEC`) 标志，在执行 `exec()` 类函数时会自动关闭该文件描述符[reference:15]。 |
| `MFD_ALLOW_SEALING` | 允许对该文件进行“密封”操作（通过 `fcntl(2)` 的 `F_ADD_SEALS` 命令）。此标志为后续的密封操作提供了前提[reference:16]。 |
| `MFD_HUGETLB` | 使用巨页（Huge Page）为匿名文件分配内存。这可以提升内存访问性能，尤其是在大内存块操作的场景下。配合 `MFD_HUGE_*` 宏可指定巨页大小[reference:17]。 |
| `MFD_NOEXEC_SEAL` | 创建一个不可执行的内存文件，并为其添加 `F_SEAL_EXEC` 密封，防止之后通过 `chmod` 等方式使其变为可执行[reference:18]。 |
| `MFD_EXEC` | 显式创建一个可执行的内存文件[reference:19]。 |
| `MFD_HUGE_64KB`, `MFD_HUGE_512KB`, `MFD_HUGE_1MB`, `MFD_HUGE_2MB`, `MFD_HUGE_8MB`, `MFD_HUGE_16MB`, `MFD_HUGE_32MB`, `MFD_HUGE_256MB`, `MFD_HUGE_512MB`, `MFD_HUGE_1GB`, `MFD_HUGE_2GB`, `MFD_HUGE_16GB` | 这些宏与 `MFD_HUGETLB` 一同使用时，用于指定巨页的大小。例如，`MFD_HUGETLB | MFD_HUGE_2MB` 表示使用 2MB 的巨页。系统必须支持所请求的巨页大小[reference:20]。 |

---

## 三、结构体详解

`<sys/memfd.h>` 头文件本身不定义任何结构体。

---

## 四、类型定义

`<sys/memfd.h>` 头文件本身不定义任何新的数据类型。其所有接口和常量都基于标准的 C 类型，如 `int` 和 `unsigned int`。

---

## 五、模板声明

`<sys/memfd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口和常量定义保持不变。

---
