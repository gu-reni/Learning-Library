## `<sys/uio.h>` 头文件详解（Linux / POSIX）

`<sys/uio.h>` 是 POSIX 标准定义的头文件，提供了**分散/聚集 I/O**（Scatter/Gather I/O）的接口，允许在一次系统调用中从多个缓冲区读取数据或向多个缓冲区写入数据。它主要定义了 `struct iovec` 结构体以及 `readv()`、`writev()`、`preadv()`、`pwritev()` 等函数，能够减少系统调用次数，提高 I/O 效率。

---

## 一、函数详解

### 1. readv 函数

**函数原型：**
```c
ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
```

**作用：** 从文件描述符 `fd` 读取数据，并将数据分散存储到 `iov` 描述的多个缓冲区中（scatter read）。数据按顺序填充每个缓冲区，填满一个后再填充下一个。

**参数：**
- `fd`：文件描述符（可以是文件、套接字、管道等）。
- `iov`：指向 `struct iovec` 数组的指针，每个元素描述一个缓冲区的地址和长度。
- `iovcnt`：`iov` 数组的元素个数（必须大于 0，且不超过 `IOV_MAX`）。

**返回值：**
- 成功：返回实际读取的字节数（可能小于所有缓冲区总大小，若遇到 EOF 则返回 0）。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
#include <sys/uio.h>
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
    char buf1[64], buf2[128];
    struct iovec iov[2] = {
        { .iov_base = buf1, .iov_len = sizeof(buf1) },
        { .iov_base = buf2, .iov_len = sizeof(buf2) }
    };
    int fd = open("file.txt", O_RDONLY);
    ssize_t n = readv(fd, iov, 2);
    if (n > 0) {
        printf("Read %zd bytes\n", n);
        // buf1 和 buf2 中包含了读取的数据
    }
    close(fd);
    return 0;
}
```

**实现原理：**
1. 系统调用进入内核，从文件描述符对应的文件中读取数据。
2. 内核按照 `iov` 数组的顺序，依次将数据拷贝到用户空间提供的缓冲区中。
3. 当第一个缓冲区填满后，继续填充第二个，依此类推，直到数据读完或所有缓冲区填满。
4. 返回实际拷贝的总字节数。

**线程安全提示：**
`readv()` 是线程安全的。多个线程可以同时调用 `readv` 读取不同的文件描述符，或者对同一个文件描述符并发读取（但需注意文件偏移量的共享问题，建议使用 `preadv` 代替）。

---

### 2. writev 函数

**函数原型：**
```c
ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
```

**作用：** 将 `iov` 描述的多个缓冲区中的数据聚集起来，一次性写入文件描述符 `fd`（gather write）。

**参数：**
- `fd`：文件描述符。
- `iov`：指向 `struct iovec` 数组的指针。
- `iovcnt`：数组元素个数。

**返回值：**
- 成功：返回实际写入的字节数。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
#include <sys/uio.h>
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main() {
    char *part1 = "Hello, ";
    char *part2 = "world!\n";
    struct iovec iov[2] = {
        { .iov_base = part1, .iov_len = strlen(part1) },
        { .iov_base = part2, .iov_len = strlen(part2) }
    };
    int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    ssize_t n = writev(fd, iov, 2);
    printf("Wrote %zd bytes\n", n);
    close(fd);
    return 0;
}
```

**实现原理：**
1. 系统调用进入内核，从用户空间的多个缓冲区中读取数据。
2. 内核将这些数据按顺序组合，一次性写入文件描述符对应的文件或套接字。
3. 返回实际写入的总字节数。

**线程安全提示：**
线程安全。与 `write` 类似，多个线程对同一文件描述符并发 `writev` 可能导致数据交错，建议使用文件锁或 `pwritev`。

---

### 3. preadv 函数（Linux 扩展，POSIX.1-2008）

**函数原型：**
```c
ssize_t preadv(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```

**作用：** 从文件描述符 `fd` 的指定偏移量 `offset` 处读取数据，分散到多个缓冲区。不影响文件描述符的当前偏移量。

**参数：**
- `fd`：文件描述符（必须支持随机访问，如普通文件）。
- `iov`：指向 `iovec` 数组的指针。
- `iovcnt`：数组元素个数。
- `offset`：文件读取起始偏移量。

**返回值：** 同 `readv`。

**示例用法：**
```c
ssize_t n = preadv(fd, iov, 2, 1024);  // 从偏移 1024 处读取
```

**实现原理：** 类似 `readv`，但在读取前临时将文件偏移量设为 `offset`，读取后恢复原偏移量（或根本不改变原偏移量），操作是原子的。

**线程安全提示：**
线程安全。由于使用了显式偏移量，多个线程可以并发调用 `preadv` 操作同一个文件描述符的不同位置，不会相互干扰。

---

### 4. pwritev 函数（Linux 扩展，POSIX.1-2008）

**函数原型：**
```c
ssize_t pwritev(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```

**作用：** 将多个缓冲区的数据写入文件描述符 `fd` 的指定偏移量 `offset` 处。不影响文件描述符的当前偏移量。

**参数：** 同 `preadv`。

**返回值：** 同 `writev`。

**示例用法：**
```c
ssize_t n = pwritev(fd, iov, 2, 0);   // 从文件开头写入
```

**实现原理：** 类似 `writev`，但在写入前临时设置偏移量，写入后恢复。

**线程安全提示：**
线程安全。多个线程可以并发使用 `pwritev` 写入同一文件的不同位置。

---

### 5. preadv2 / pwritev2（Linux 特有，较新）

这些函数在 `<sys/uio.h>` 中也有声明（需要定义 `_GNU_SOURCE`），提供了额外的 `flags` 参数（如 `RWF_DSYNC`、`RWF_HIPRI` 等）。由于属于 Linux 扩展，此处不做详细展开。

---

## 二、结构体详解

### struct iovec

**定义：**
```c
struct iovec {
    void  *iov_base;   // 指向缓冲区的起始地址
    size_t iov_len;    // 缓冲区的长度（字节数）
};
```

**作用：** 描述一个内存缓冲区，用于 `readv`/`writev` 等分散/聚集 I/O 函数。多个 `iovec` 结构组成一个数组，表示多个缓冲区。

**成员详解：**
- `iov_base`：指向缓冲区的指针。对于 `readv`，内核将数据写入该地址；对于 `writev`，内核从该地址读取数据。
- `iov_len`：缓冲区的字节数。不能为 0（虽然某些实现允许 0，但 POSIX 未定义）。

**使用注意事项：**
- 在调用 `readv`/`writev` 之前，必须正确设置每个 `iovec` 的 `iov_base` 和 `iov_len`。
- 缓冲区可以是任意内存区域（栈、堆、静态区）。
- `iov_len` 不应超过系统限制（通常为 `SSIZE_MAX`），否则可能导致未定义行为。
- 对于 `readv`，若某个缓冲区的 `iov_len` 很大，但文件剩余数据不足，则只会填充部分数据，该缓冲区可能不会被填满。

**示例用法：** 见上述函数示例。

---

## 三、宏定义详解

### 1. `IOV_MAX`

**定义：** 一个常量，表示 `readv`/`writev` 等函数允许的 `iovcnt` 最大值。其值至少为 16（POSIX 要求），在 Linux 上通常为 1024。

**作用：** 用于限制 `iov` 数组的大小，避免超出系统限制。

**示例：**
```c
#include <limits.h>  // 或直接使用
#ifdef IOV_MAX
    if (iovcnt > IOV_MAX) {
        // 处理错误或分批次处理
    }
#endif
```

**注意：** `IOV_MAX` 可能定义在 `<limits.h>` 中，但 `<sys/uio.h>` 也会间接包含。

---

## 四、类型定义

- `ssize_t`：有符号整数，用于表示字节数或错误（-1）。定义在 `<sys/types.h>`。
- `size_t`：无符号整数，表示缓冲区大小。
- `off_t`：文件偏移量类型。

---

## 五、模板声明

`<sys/uio.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
