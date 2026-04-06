## `<sys/signalfd.h>` 头文件详解（Linux 特有接口）

`<sys/signalfd.h>` 是 Linux 特有的头文件，提供了 **signalfd** 机制，允许将信号接收转换为文件描述符上的可读事件。通过 signalfd，应用程序可以使用 `read()`、`select()`、`poll()` 或 `epoll()` 来同步地处理信号，避免了传统信号处理函数中的异步安全问题。该机制由 Linux 2.6.22 引入内核。

---

## 一、函数详解

### 1. signalfd / signalfd4

**函数原型：**
```c
int signalfd(int fd, const sigset_t *mask, int flags);
int signalfd4(int fd, const sigset_t *mask, int flags);
```

**作用：** 创建一个与信号集关联的文件描述符，通过该描述符可以读取挂起（pending）的信号。

**参数：**
- `fd`：若为 `-1`，则创建一个新的 signalfd 文件描述符；若为一个已有的 signalfd 文件描述符，则替换其关联的信号掩码。
- `mask`：指向要监听的信号集（阻塞集），信号集中的信号将被 signalfd 捕获。
- `flags`：可取 `0` 或以下值的按位或：
  - `SFD_CLOEXEC`：设置 close-on-exec 标志。
  - `SFD_NONBLOCK`：设置非阻塞模式。
  - `SFD_NO`（signalfd4 特有）：暂无扩展，通常设为 0。

**返回值：** 成功返回文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/signalfd.h>
#include <signal.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    sigset_t mask;
    sigemptyset(&mask);
    sigaddset(&mask, SIGINT);
    sigaddset(&mask, SIGTERM);
    // 阻塞这些信号，避免默认处理
    sigprocmask(SIG_BLOCK, &mask, NULL);

    int sfd = signalfd(-1, &mask, SFD_CLOEXEC);
    if (sfd == -1) {
        perror("signalfd");
        return 1;
    }

    struct signalfd_siginfo info;
    ssize_t s = read(sfd, &info, sizeof(info));
    if (s == sizeof(info)) {
        printf("Received signal: %d\n", info.ssi_signo);
    }
    close(sfd);
    return 0;
}
```

**实现原理：** 系统调用进入内核，创建一个 `signalfd_ctx` 结构体，关联当前进程的信号掩码的副本，并返回匿名文件描述符。当进程收到 `mask` 中的信号时，内核将该信号转换为 `struct signalfd_siginfo` 结构体并放入 signalfd 的队列中。`read()` 操作从队列中读取数据。

**线程安全提示：** 线程安全。多个线程可以对不同 signalfd 实例操作；对同一实例的并发 `read` 需要外部同步。

---

## 二、结构体详解

### struct signalfd_siginfo

**定义：**
```c
struct signalfd_siginfo {
    uint32_t ssi_signo;      // 信号编号
    int32_t  ssi_errno;      // 错误码（如果适用）
    int32_t  ssi_code;       // 信号来源代码
    uint32_t ssi_pid;        // 发送进程 PID
    uint32_t ssi_uid;        // 发送进程 UID
    int32_t  ssi_fd;         // 关联的文件描述符（如 SIGIO 相关）
    uint32_t ssi_tid;        // 线程 ID
    uint32_t ssi_band;       // 事件带外标志（如 SIGIO）
    uint32_t ssi_overrun;    // 定时器超限计数（如 SIGALRM）
    uint32_t ssi_trapno;     // 陷阱号（架构相关）
    int32_t  ssi_status;     // 子进程退出状态（如 SIGCHLD）
    int32_t  ssi_int;        // 发送信号携带的整型数据（如 sigqueue）
    uint64_t ssi_ptr;        // 发送信号携带的指针数据（如 sigqueue）
    uint64_t ssi_utime;      // 用户 CPU 时间（如 SIGCHLD）
    uint64_t ssi_stime;      // 系统 CPU 时间（如 SIGCHLD）
    uint64_t ssi_addr;       // 触发信号的地址（如 SIGSEGV）
    uint16_t ssi_addr_lsb;   // 地址最低有效位（某些硬件故障）
    uint8_t  pad[46];        // 填充，使结构体固定大小
};
```

**作用：** 存储一个信号的全部信息，通过 `read()` 从 signalfd 读取得到。其内容类似于 `siginfo_t`。

**成员详解：**
- `ssi_signo`：信号编号（如 `SIGINT`）。
- `ssi_errno`：如果信号由内核错误触发，则为此错误号。
- `ssi_code`：信号来源（如 `SI_USER`、`SI_KERNEL`）。
- `ssi_pid`、`ssi_uid`：发送进程的 PID 和 UID。
- `ssi_fd`：与信号相关的文件描述符（例如 `SIGIO`）。
- `ssi_status`：子进程退出状态（用于 `SIGCHLD`）。
- `ssi_int`、`ssi_ptr`：通过 `sigqueue()` 发送的附加数据。
- `ssi_addr`：引发内存错误的内存地址（如 `SIGSEGV`）。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `SFD_CLOEXEC` | 设置 close-on-exec 标志，`exec` 时自动关闭描述符。 |
| `SFD_NONBLOCK` | 设置非阻塞模式，`read()` 无信号时返回 `EAGAIN`。 |
| `SFD_NO` | signalfd4 保留，通常为 0。 |

---

## 四、类型定义

- `struct signalfd_siginfo`：如上所述。
- 无其他特有类型。

---

## 五、示例用法

以下示例结合 `epoll` 使用 signalfd 处理信号：

```c
#include <sys/signalfd.h>
#include <sys/epoll.h>
#include <signal.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    sigset_t mask;
    sigemptyset(&mask);
    sigaddset(&mask, SIGINT);
    sigprocmask(SIG_BLOCK, &mask, NULL);

    int sfd = signalfd(-1, &mask, SFD_NONBLOCK);
    int epfd = epoll_create1(0);
    struct epoll_event ev = { .events = EPOLLIN, .data.fd = sfd };
    epoll_ctl(epfd, EPOLL_CTL_ADD, sfd, &ev);

    while (1) {
        int n = epoll_wait(epfd, &ev, 1, -1);
        if (n > 0 && ev.data.fd == sfd) {
            struct signalfd_siginfo info;
            if (read(sfd, &info, sizeof(info)) == sizeof(info)) {
                printf("Caught signal %d\n", info.ssi_signo);
                if (info.ssi_signo == SIGINT) break;
            }
        }
    }
    close(sfd);
    close(epfd);
    return 0;
}
```

---
