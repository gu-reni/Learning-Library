## `<sys/pidfd.h>` 头文件详解（Linux 特有接口）

`<sys/pidfd.h>` 是 Linux 系统提供的头文件，用于操作 PID 文件描述符（pidfd）。它定义了一组系统调用，让进程可以安全、无竞争地操作其他进程，解决了传统 PID 操作中可能出现的 PID 重用问题。进程可以通过 pidfd 进行发送信号、复制文件描述符等操作，避免在信号发送等场景中出现目标进程已终止且 PID 被重用而导致错误信号发送给无关进程的问题。

---

## 一、函数详解

### 1. pidfd_open

**函数原型：**
```c
int pidfd_open(pid_t pid, unsigned int flags);
```

**作用：** 打开一个进程，返回一个引用该进程的文件描述符。返回的文件描述符默认设置了 `close-on-exec` 标志。

**参数：**
- `pid`：目标进程的 PID。
- `flags`：控制标志，可为 `0` 或 `PIDFD_NONBLOCK`、`PIDFD_THREAD` 等。

**返回值：** 成功返回非负文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/pidfd.h>
#include <errno.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t target_pid = 1234;
    int pidfd = pidfd_open(target_pid, 0);
    if (pidfd == -1) {
        perror("pidfd_open");
        return 1;
    }
    close(pidfd);
    return 0;
}
```

**实现原理：** `pidfd_open()` 是一个系统调用，内核会创建一个新的文件描述符，该描述符与给定的 PID 绑定。只要这个文件描述符没有关闭，内核就会确保对应的进程不会被完全销毁，从而避免了 PID 重用的问题。

**线程安全提示：** 是安全的。它仅作用于单个文件描述符的创建。

---

### 2. pidfd_getfd

**函数原型：**
```c
int pidfd_getfd(int pidfd, int targetfd, unsigned int flags);
```

**作用：** 从 `pidfd` 引用的目标进程中，复制一个已经打开的文件描述符到当前进程。

**参数：**
- `pidfd`：由 `pidfd_open()` 返回的 PID 文件描述符。
- `targetfd`：目标进程中想要复制的文件描述符的数字。
- `flags`：保留供未来使用，必须设为 `0`。

**返回值：** 成功返回当前进程中新的文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
int local_fd = pidfd_getfd(pidfd, 2, 0); // 复制目标进程的 stderr
if (local_fd == -1) perror("pidfd_getfd");
```

**实现原理：** 系统调用。内核会验证调用者是否有足够的权限（通常要求进程间的 ptrace 权限），然后从目标进程的文件描述符表中复制对应的文件表项到当前进程，并分配一个新的文件描述符。

**线程安全提示：** 是安全的。但需要注意，在获取文件描述符时，目标进程可能已经关闭了 `targetfd`，导致调用失败。

---

### 3. pidfd_send_signal

**函数原型：**
```c
int pidfd_send_signal(int pidfd, int sig, siginfo_t *info, unsigned int flags);
```

**作用：** 向由 `pidfd` 引用的进程发送信号。

**参数：**
- `pidfd`：PID 文件描述符。
- `sig`：要发送的信号编号（如 `SIGTERM`）。
- `info`：指向 `siginfo_t` 结构的指针，可用于携带额外数据。如果不需要，可设为 `NULL`。
- `flags`：控制标志，可为 `0`、`PIDFD_SIGNAL_THREAD`、`PIDFD_SIGNAL_THREAD_GROUP` 等。

**返回值：** 成功返回 0，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
if (pidfd_send_signal(pidfd, SIGTERM, NULL, 0) == -1) perror("pidfd_send_signal");
```

**实现原理：** 系统调用，通过 PID 文件描述符直接定位到目标进程的任务结构体，然后向其发送信号。这种方式比传统 `kill()` 更安全，不受 PID 重用影响。

**线程安全提示：** 是安全的。多个线程可以同时对同一个 `pidfd` 调用此函数。

---

### 4. pidfd_getpid

**函数原型：**
```c
pid_t pidfd_getpid(int fd);
```

**作用：** 查询一个 PID 文件描述符所引用的进程的实际 PID。

**参数：**
- `fd`：PID 文件描述符。

**返回值：** 成功返回进程的 PID，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
pid_t actual_pid = pidfd_getpid(pidfd);
if (actual_pid == -1) perror("pidfd_getpid");
```

**实现原理：** 该函数通过解析 `/proc/` 文件系统获取信息，读取 `pidfd` 对应的 `procfs` 条目并提取 PID。

**线程安全提示：** 是安全的。只读操作。

---

## 二、结构体详解

`<sys/pidfd.h>` 本身不定义结构体。它使用标准类型如 `pid_t`、`siginfo_t`（来自 `<signal.h>`）。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `PIDFD_NONBLOCK` | 等同于 `O_NONBLOCK`，在打开 PID 文件描述符时设置，可使后续的 `waitid` 等操作变为非阻塞模式。 |
| `PIDFD_THREAD` | 指定 `pidfd_open()` 返回的文件描述符引用的是特定线程而非整个进程组（线程组 leader）。 |
| `PIDFD_SIGNAL_THREAD` | 用于 `pidfd_send_signal()` 中，确保信号被发送到 `pidfd` 引用的特定线程。 |
| `PIDFD_SIGNAL_THREAD_GROUP` | 用于 `pidfd_send_signal()` 中，确保信号被发送到 `pidfd` 引用的整个线程组。 |
| `PIDFD_SIGNAL_PROCESS_GROUP` | 用于 `pidfd_send_signal()` 中，确保信号被发送到 `pidfd` 引用的进程组。 |

---

## 四、类型定义

- `pid_t`：进程 ID 类型（定义在 `<sys/types.h>`）。
- `siginfo_t`：信号信息结构体（定义在 `<signal.h>` 中）。

---

## 五、模板声明

`<sys/pidfd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
