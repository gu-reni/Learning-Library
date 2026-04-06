## `<sys/timerfd.h>` 头文件详解（Linux 特有接口）

`<sys/timerfd.h>` 是 Linux 特有的头文件，提供了 **timerfd** 机制，允许将定时器事件转换为文件描述符上的可读事件。通过 timerfd，应用程序可以使用 `read()`、`select()`、`poll()` 或 `epoll()` 来同步地处理定时器到期，避免了传统定时器（如 `setitimer`）信号处理的异步安全问题。该机制由 Linux 2.6.25 引入内核。

---

## 一、函数详解

### 1. timerfd_create

**函数原型：**
```c
int timerfd_create(int clockid, int flags);
```

**作用：** 创建一个 timerfd 对象，返回一个文件描述符，该描述符用于后续的定时器操作和事件读取。

**参数：**
- `clockid`：指定定时器使用的时钟类型，可取以下值：
  - `CLOCK_REALTIME`：系统范围的实时时钟（可被系统时间调整影响）。
  - `CLOCK_MONOTONIC`：单调递增时钟（不受系统时间调整影响）。
  - `CLOCK_BOOTTIME`：包含系统休眠时间的单调时钟（Linux 3.15+）。
  - `CLOCK_REALTIME_ALARM`（需 `CAP_WAKE_ALARM`）：实时时钟，可唤醒系统。
  - `CLOCK_BOOTTIME_ALARM`（需 `CAP_WAKE_ALARM`）：可唤醒系统的启动时钟。
- `flags`：可取 `0` 或以下值的按位或：
  - `TFD_CLOEXEC`：设置 close-on-exec 标志。
  - `TFD_NONBLOCK`：设置非阻塞模式。

**返回值：** 成功返回非负文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/timerfd.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int tfd = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK);
    if (tfd == -1) {
        perror("timerfd_create");
        return 1;
    }
    close(tfd);
    return 0;
}
```

**实现原理：** 系统调用进入内核，创建 `struct timerfd_ctx` 结构体，分配一个匿名文件描述符，并关联到进程的文件描述符表。该结构体包含一个高精度定时器（`hrtimer`）和一个等待队列。

**线程安全提示：** 线程安全。多个线程可以同时创建不同的 timerfd 实例。

---

### 2. timerfd_settime

**函数原型：**
```c
int timerfd_settime(int fd, int flags,
                    const struct itimerspec *new_value,
                    struct itimerspec *old_value);
```

**作用：** 启动或停止 timerfd 上的定时器，设置首次到期时间和后续间隔。

**参数：**
- `fd`：timerfd 文件描述符。
- `flags`：可取 `0` 或 `TFD_TIMER_ABSTIME`（使用绝对时间）。
- `new_value`：指向 `struct itimerspec` 的指针，指定新的定时器设置。
- `old_value`：输出参数，返回之前的定时器设置（可为 `NULL`）。

**返回值：** 成功返回 0，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/timerfd.h>
#include <time.h>
#include <stdio.h>

int main() {
    int tfd = timerfd_create(CLOCK_MONOTONIC, 0);
    struct itimerspec its = {
        .it_value = { .tv_sec = 1, .tv_nsec = 0 },      // 1 秒后首次到期
        .it_interval = { .tv_sec = 2, .tv_nsec = 0 }     // 之后每隔 2 秒到期
    };
    if (timerfd_settime(tfd, 0, &its, NULL) == -1) {
        perror("timerfd_settime");
        close(tfd);
        return 1;
    }
    // 读取定时器事件...
    close(tfd);
    return 0;
}
```

**实现原理：** 内核将 `new_value` 转换为内核定时器参数，调用 `hrtimer_start()` 启动定时器。若使用绝对时间（`TFD_TIMER_ABSTIME`），则直接使用指定的到期时间；否则计算当前时间加上相对时间。`old_value` 返回之前的定时器设置。

**线程安全提示：** 线程安全。对同一 `fd` 的并发调用由内核同步保护。

---

### 3. timerfd_gettime

**函数原型：**
```c
int timerfd_gettime(int fd, struct itimerspec *curr_value);
```

**作用：** 获取 timerfd 当前定时器的设置和剩余时间。

**参数：**
- `fd`：timerfd 文件描述符。
- `curr_value`：输出参数，返回当前定时器设置。其中 `it_value` 表示距离下次到期的剩余时间（若定时器未启动则为 0），`it_interval` 表示间隔。

**返回值：** 成功返回 0，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
struct itimerspec curr;
if (timerfd_gettime(tfd, &curr) == -1) {
    perror("timerfd_gettime");
} else {
    printf("Remaining: %ld.%09ld seconds\n", curr.it_value.tv_sec, curr.it_value.tv_nsec);
}
```

**实现原理：** 内核读取 `timerfd_ctx` 中的定时器状态，计算剩余时间并填充 `curr_value`。

**线程安全提示：** 线程安全。

---

### 4. 通过 `read()` 读取到期事件

timerfd 文件描述符支持 `read()` 操作，每次定时器到期时，内核会将一个 8 字节的 `uint64_t` 值写入队列，表示到期的次数。

**作用：** 读取并消费定时器到期事件。若未设置 `TFD_NONBLOCK` 且队列为空，则阻塞直到下次到期。

**返回值：** 成功读取 8 字节，失败返回 -1。

**示例用法：**
```c
uint64_t expirations;
ssize_t s = read(tfd, &expirations, sizeof(expirations));
if (s == sizeof(expirations)) {
    printf("Timer expired %llu times\n", (unsigned long long)expirations);
}
```

**实现原理：** 每次定时器到期，内核递增 `timerfd_ctx` 中的到期计数器，并唤醒等待队列中的进程。`read()` 将计数器值拷贝到用户空间并清零计数器。

**线程安全提示：** 对同一 `fd` 的并发 `read` 需要外部同步，建议单线程读取。

---

## 二、结构体详解

### struct itimerspec

**定义：**
```c
struct itimerspec {
    struct timespec it_interval;  // 定时器间隔（周期）
    struct timespec it_value;     // 首次到期时间（或剩余时间）
};
```

**作用：** 描述定时器的设置，用于 `timerfd_settime` 和 `timerfd_gettime`。

**成员详解：**
- `it_interval`：定时器到期后的间隔时间。若 `it_interval.tv_sec == 0` 且 `tv_nsec == 0`，则为单次定时器，到期后不再重复。
- `it_value`：首次到期的绝对时间（若 `flags` 含 `TFD_TIMER_ABSTIME`）或相对时间。若 `it_value` 为 0，则定时器停止。

---

### struct timespec

**定义：**
```c
struct timespec {
    time_t tv_sec;   // 秒
    long   tv_nsec;  // 纳秒（0 到 999999999）
};
```

**作用：** 表示时间值，用于 `itimerspec` 的成员。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `TFD_CLOEXEC` | `timerfd_create` 标志，设置 close-on-exec。 |
| `TFD_NONBLOCK` | `timerfd_create` 标志，设置非阻塞模式。 |
| `TFD_TIMER_ABSTIME` | `timerfd_settime` 标志，表示使用绝对时间。 |

---

## 四、类型定义

- `uint64_t`：用于 `read()` 读取的到期计数器类型（通常来自 `<stdint.h>`）。
- `struct itimerspec`、`struct timespec`。

---

## 五、模板声明

`<sys/timerfd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
