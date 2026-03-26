# `<signal.h>` 核心函数、结构体与宏详解（Linux / POSIX）

## 说明
`<signal.h>` 是 C 标准库和 POSIX 标准定义的头文件，提供了信号处理机制。它包含了设置信号处理函数（`signal`、`sigaction`）、发送信号（`raise`、`kill`）、阻塞信号（`sigprocmask`）、等待信号（`sigsuspend`）等函数，以及相关的结构体和宏。本文档按照统一格式整理其中的函数、结构体、宏等信息，并说明内部原理与线程安全性。

---

## 一、函数详解

### 1. signal 函数
**函数原型：**
```c
void (*signal(int signum, void (*handler)(int)))(int);
```
**作用：** 设置指定信号的处理方式。  
**参数：**  
- `signum`：信号编号（如 `SIGINT`、`SIGTERM`）。  
- `handler`：信号处理函数指针，可以是：
  - `SIG_IGN`：忽略该信号。  
  - `SIG_DFL`：恢复默认处理。  
  - 自定义函数：函数类型为 `void handler(int)`。  
**返回值：** 返回之前设置的信号处理函数指针（或 `SIG_ERR` 表示错误）。  
**示例用法：**
```c
void sigint_handler(int sig) {
    printf("Caught SIGINT\n");
}
signal(SIGINT, sigint_handler);
```
**实现原理：**  
1. 系统调用进入内核，修改进程的 `sigaction` 表项。  
2. 内核记录该信号的处理方式。  
3. 当信号递送时，根据表项执行相应操作（调用用户函数、忽略或默认动作）。  
**线程安全提示：** `signal()` 本身是线程安全的，但它的行为在不同系统上可能有差异（如是否自动重置处理函数）。多线程环境中推荐使用 `sigaction` 替代。

---

### 2. sigaction 函数
**函数原型：**
```c
int sigaction(int signum, const struct sigaction *act,
              struct sigaction *oldact);
```
**作用：** 设置或获取信号的处理方式（更精细的控制）。  
**参数：**  
- `signum`：信号编号。  
- `act`：指向 `struct sigaction` 的指针，指定新处理方式（可为 `NULL`）。  
- `oldact`：输出参数，返回旧的信号处理方式（可为 `NULL`）。  
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
struct sigaction sa;
sa.sa_handler = sigint_handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = 0;  // 或 SA_RESTART 等
sigaction(SIGINT, &sa, NULL);
```
**实现原理：** 系统调用，内核直接修改进程的 `sigaction` 表项，提供更细粒度的控制（如信号掩码、标志）。  
**线程安全提示：** 线程安全。多线程调用时，修改的是进程级别的信号处理方式，所有线程共享。

---

### 3. raise 函数
**函数原型：**
```c
int raise(int sig);
```
**作用：** 向当前进程发送信号。  
**参数：**  
- `sig`：信号编号。  
**返回值：** 成功返回 0，失败返回非零。  
**示例用法：**
```c
raise(SIGUSR1);
```
**实现原理：** 等价于 `kill(getpid(), sig)`。  
**线程安全提示：** 线程安全。向当前进程发送信号，信号可能被任意线程处理。

---

### 4. kill 函数
**函数原型：**
```c
int kill(pid_t pid, int sig);
```
**作用：** 向指定进程或进程组发送信号。  
**参数：**  
- `pid`：目标进程 ID。  
  - `>0`：发送给指定进程。  
  - `0`：发送给同组所有进程。  
  - `-1`：发送给所有有权限发送的进程（需超级用户）。  
  - `< -1`：发送给 `-pid` 进程组。  
- `sig`：信号编号。  
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
kill(pid, SIGTERM);
```
**实现原理：** 系统调用，内核找到目标进程（或进程组），递送信号。  
**线程安全提示：** 线程安全（只影响目标进程）。

---

### 5. sigprocmask 函数
**函数原型：**
```c
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```
**作用：** 修改或获取当前线程的信号掩码（阻塞的信号集）。  
**参数：**  
- `how`：操作方式  
  - `SIG_BLOCK`：将 `set` 中的信号添加到阻塞集。  
  - `SIG_UNBLOCK`：从阻塞集中移除 `set` 中的信号。  
  - `SIG_SETMASK`：将阻塞集设置为 `set`。  
- `set`：指向新信号集的指针（可为 `NULL`）。  
- `oldset`：输出参数，返回旧的信号掩码（可为 `NULL`）。  
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
sigset_t newmask, oldmask;
sigemptyset(&newmask);
sigaddset(&newmask, SIGINT);
sigprocmask(SIG_BLOCK, &newmask, &oldmask);
// 临界区，SIGINT 被阻塞
sigprocmask(SIG_SETMASK, &oldmask, NULL);
```
**实现原理：** 系统调用，修改线程的 `sigmask`（信号掩码）。被阻塞的信号不会递送，但会挂起等待。  
**线程安全提示：** 线程安全（每个线程有独立的信号掩码）。

---

### 6. sigpending 函数
**函数原型：**
```c
int sigpending(sigset_t *set);
```
**作用：** 获取当前线程已挂起（等待递送）的信号集。  
**参数：**  
- `set`：输出参数，存放挂起信号集。  
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
sigset_t pending;
sigpending(&pending);
if (sigismember(&pending, SIGINT)) {
    printf("SIGINT is pending\n");
}
```
**实现原理：** 系统调用，返回内核中该线程的挂起信号集。  
**线程安全提示：** 线程安全。

---

### 7. sigsuspend 函数
**函数原型：**
```c
int sigsuspend(const sigset_t *mask);
```
**作用：** 原子性地将线程的信号掩码替换为 `mask`，并挂起线程直到收到一个未阻塞的信号。  
**参数：**  
- `mask`：临时信号掩码。  
**返回值：** 总是返回 -1，并设置 `errno` 为 `EINTR`（因信号中断）。  
**示例用法：**
```c
sigset_t newmask;
sigemptyset(&newmask);
// 等待任何信号，除了已阻塞的
sigsuspend(&newmask);
```
**实现原理：** 系统调用，原子地设置掩码并进入可中断睡眠，信号递送后恢复原掩码。  
**线程安全提示：** 线程安全。

---

### 8. sigwait / sigwaitinfo / sigtimedwait 函数（POSIX 扩展）
**函数原型：**
```c
int sigwait(const sigset_t *set, int *sig);
int sigwaitinfo(const sigset_t *set, siginfo_t *info);
int sigtimedwait(const sigset_t *set, siginfo_t *info,
                 const struct timespec *timeout);
```
**作用：** 同步等待信号（阻塞线程直到信号递送）。  
**参数：**  
- `set`：等待的信号集。  
- `sig`：输出参数，接收信号编号（`sigwait`）。  
- `info`：输出参数，接收信号信息（`siginfo_t`）。  
- `timeout`：超时时间（`sigtimedwait`）。  
**返回值：**  
- `sigwait`：成功返回 0，失败返回错误码。  
- `sigwaitinfo`：成功返回信号编号，失败返回 -1。  
- `sigtimedwait`：成功返回信号编号，超时返回 -1（`errno=EAGAIN`）。  
**示例用法：**
```c
sigset_t set;
sigemptyset(&set);
sigaddset(&set, SIGINT);
int sig;
sigwait(&set, &sig);
printf("Received signal %d\n", sig);
```
**实现原理：** 系统调用，将线程挂起直到信号集中的任一信号递送。信号不经过用户处理函数。  
**线程安全提示：** 线程安全。每个线程可独立等待。

---

### 9. 其他常用函数
- `sigemptyset(sigset_t *set)`：清空信号集（宏或函数）。  
- `sigfillset(sigset_t *set)`：填满信号集。  
- `sigaddset(sigset_t *set, int signum)`：向信号集中添加信号。  
- `sigdelset(sigset_t *set, int signum)`：从信号集中删除信号。  
- `sigismember(const sigset_t *set, int signum)`：测试信号是否在集中。  
- `psignal(int sig, const char *s)`：打印信号描述（类似 `perror`）。  
- `strsignal(int sig)`：返回信号描述字符串（线程安全）。

---

## 二、结构体详解

### 1. struct sigaction
**定义：**
```c
struct sigaction {
    void     (*sa_handler)(int);   // 信号处理函数（或 SIG_IGN/SIG_DFL）
    void     (*sa_sigaction)(int, siginfo_t *, void *); // 扩展处理函数
    sigset_t   sa_mask;            // 信号处理期间要阻塞的信号集
    int        sa_flags;           // 标志（SA_RESTART, SA_SIGINFO 等）
    void     (*sa_restorer)(void); // 已废弃
};
```
**作用：** 用于 `sigaction` 函数，指定信号处理方式。  
**成员详解：**  
- `sa_handler`：简单信号处理函数。  
- `sa_sigaction`：当 `sa_flags` 包含 `SA_SIGINFO` 时使用，可获取更多信息（`siginfo_t`）。  
- `sa_mask`：在信号处理函数执行期间被阻塞的信号集。  
- `sa_flags`：标志位，常用：
  - `SA_RESTART`：被信号中断的系统调用自动重启。  
  - `SA_SIGINFO`：使用 `sa_sigaction` 而不是 `sa_handler`。  
  - `SA_NOCLDSTOP`：子进程停止时不产生 `SIGCHLD`。  
  - `SA_NOCLDWAIT`：子进程结束时不会成为僵尸。  
  - `SA_ONSTACK`：使用备用信号栈（`sigaltstack`）。  
  - `SA_RESETHAND`：执行后恢复默认处理（类似于 `signal` 的默认行为）。  
  - `SA_NODEFER`：处理函数执行时不阻塞当前信号。

---

### 2. siginfo_t 结构（信号信息）
**定义：**（精简）
```c
typedef struct {
    int      si_signo;   // 信号编号
    int      si_errno;   // 错误号（如果适用）
    int      si_code;    // 信号来源代码
    pid_t    si_pid;     // 发送进程 PID
    uid_t    si_uid;     // 发送进程 UID
    void    *si_addr;    // 引发错误的内存地址（如 SIGSEGV）
    int      si_status;  // 子进程退出状态（SIGCHLD）
    // ... 其他字段
} siginfo_t;
```
**作用：** 携带关于信号的详细信息，通过 `sa_sigaction` 获取。  
**成员详解：**  
- `si_code` 指示信号来源（如 `SI_USER` 来自 `kill`，`SI_QUEUE` 来自 `sigqueue`）。  
- 不同信号可能有不同的扩展字段（如 `SIGSEGV` 的 `si_addr`）。

---

### 3. sigset_t 类型（信号集）
`sigset_t` 是不透明类型，通过宏操作。

---

## 三、宏定义详解

### 1. 标准信号编号
| 宏名称 | 作用 |
|--------|------|
| `SIGABRT` | 异常终止（abort） |
| `SIGALRM` | 定时器超时（alarm） |
| `SIGBUS` | 总线错误（内存访问对齐） |
| `SIGCHLD` | 子进程状态改变 |
| `SIGCONT` | 继续执行被暂停的进程 |
| `SIGFPE` | 浮点异常 |
| `SIGHUP` | 挂断（终端关闭） |
| `SIGILL` | 非法指令 |
| `SIGINT` | 中断（Ctrl+C） |
| `SIGKILL` | 强制终止（不能被捕获或忽略） |
| `SIGPIPE` | 管道破裂（写无读者） |
| `SIGQUIT` | 退出（Ctrl+\） |
| `SIGSEGV` | 段错误（无效内存访问） |
| `SIGSTOP` | 暂停进程（不能被捕获或忽略） |
| `SIGTERM` | 终止信号（默认终止） |
| `SIGTSTP` | 终端停止（Ctrl+Z） |
| `SIGTTIN` | 后台进程读终端 |
| `SIGTTOU` | 后台进程写终端 |
| `SIGUSR1` | 用户自定义信号1 |
| `SIGUSR2` | 用户自定义信号2 |
| `SIGPROF` | 性能分析定时器 |
| `SIGSYS` | 无效系统调用 |
| `SIGTRAP` | 跟踪/断点陷阱 |
| `SIGURG` | 紧急数据（套接字带外数据） |
| `SIGVTALRM` | 虚拟定时器超时 |
| `SIGXCPU` | CPU 时间超限 |
| `SIGXFSZ` | 文件大小超限 |
| `SIGWINCH` | 窗口大小改变 |
| `SIGIO` / `SIGPOLL` | I/O 就绪事件 |

---

### 2. 信号处理方式常量
| 宏名称 | 作用 |
|--------|------|
| `SIG_DFL` | 默认处理 |
| `SIG_IGN` | 忽略信号 |
| `SIG_ERR` | 错误返回值 |

---

### 3. sigprocmask 的 how 参数
| 宏名称 | 作用 |
|--------|------|
| `SIG_BLOCK` | 添加信号到阻塞集 |
| `SIG_UNBLOCK` | 从阻塞集中移除信号 |
| `SIG_SETMASK` | 设置阻塞集 |

---

### 4. sa_flags 标志
| 宏名称 | 作用 |
|--------|------|
| `SA_RESTART` | 被信号中断的系统调用自动重启 |
| `SA_SIGINFO` | 使用 `sa_sigaction` |
| `SA_NOCLDSTOP` | 子进程停止时不产生 `SIGCHLD` |
| `SA_NOCLDWAIT` | 子进程结束时不会成为僵尸 |
| `SA_ONSTACK` | 使用备用信号栈 |
| `SA_RESETHAND` | 信号处理函数执行后重置为默认 |
| `SA_NODEFER` | 处理函数执行时不阻塞当前信号 |

---

### 5. siginfo_t 的 si_code 值（部分）
| 宏名称 | 作用 |
|--------|------|
| `SI_USER` | 来自 `kill`、`raise` 等 |
| `SI_KERNEL` | 来自内核 |
| `SI_QUEUE` | 来自 `sigqueue` |
| `SI_TIMER` | 来自定时器 |
| `SI_ASYNCIO` | 来自异步 I/O |
| `SI_MESGQ` | 来自消息队列 |

---

### 6. 其他宏
- `NSIG`：最大信号编号 + 1。
- `_NSIG`：信号数量（通常与 `NSIG` 相同）。
- `SIG_BLOCK`、`SIG_UNBLOCK`、`SIG_SETMASK` 已在上面列出。

---

## 四、类型定义
- `sigset_t`：信号集类型。
- `siginfo_t`：信号信息结构体。
- `sig_atomic_t`：原子整数类型（用于信号处理函数中共享变量，保证读写原子性）。

---

## 五、模板声明
`<signal.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变，但通常使用 `<csignal>`。

---
（注：它也是 C 标准库的一部分，属于 `C 标准库 / 信号处理`，但由于其在系统编程中的核心地位，这里按 POSIX 系统编程分类。）