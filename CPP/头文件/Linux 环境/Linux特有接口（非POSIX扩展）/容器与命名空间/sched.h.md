## `<sched.h>` 头文件详解（Linux / POSIX）

`<sched.h>` 是 POSIX 标准定义的头文件，提供了进程和线程的**调度控制接口**。它允许应用程序设置和获取进程的调度策略、优先级、CPU 亲和性（affinity），并让线程主动让出 CPU。该头文件是实现实时系统和性能敏感型程序的关键基础。

---

## 一、函数详解

### 1. `sched_setaffinity` / `sched_getaffinity`

**函数原型：**
```c
int sched_setaffinity(pid_t pid, size_t cpusetsize, const cpu_set_t *mask);
int sched_getaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);
```

**作用：** 设置或获取线程的 CPU 亲和性掩码（affinity mask），即线程可以运行在哪些 CPU 上。该功能可提升缓存亲和性，避免线程在不同 CPU 间迁移时的缓存失效开销[reference:0]。

**参数：**
- `pid`：目标线程 ID。若为 `0`，则作用于当前线程。
- `cpusetsize`：`mask` 指向的 CPU 集合的大小（字节数），通常为 `sizeof(cpu_set_t)`。
- `mask`：指向 CPU 集合的指针。对于 `sched_setaffinity`，该参数指定线程允许运行的 CPU 集合。

**返回值：** 成功返回 `0`，失败返回 `-1` 并设置 `errno`。

**示例用法：**
```c
#define _GNU_SOURCE
#include <sched.h>
#include <stdio.h>

int main() {
    cpu_set_t mask;
    CPU_ZERO(&mask);
    CPU_SET(0, &mask);          // 绑定到 CPU 0
    sched_setaffinity(0, sizeof(mask), &mask);
    // ... 当前线程现在只能在 CPU 0 上运行
    return 0;
}
```

**实现原理：** 系统调用进入内核，内核修改线程的 `cpus_allowed` 位掩码，调度器随后会基于该掩码决定线程的放置位置。如果新掩码不包含当前运行的 CPU，线程会被立即迁移到允许的 CPU。

**线程安全提示：** 对同一线程的并发调用可能导致掩码被覆盖。设置亲和性不影响其他线程。

---

### 2. `sched_setscheduler` / `sched_getscheduler`

**函数原型：**
```c
int sched_setscheduler(pid_t pid, int policy, const struct sched_param *param);
int sched_getscheduler(pid_t pid);
```

**作用：** 设置或获取进程的调度策略和优先级参数。

**参数：**
- `pid`：目标进程 ID。若为 `0`，则作用于当前进程。
- `policy`：调度策略，可取 `SCHED_OTHER`（普通分时）、`SCHED_FIFO`（实时先进先出）、`SCHED_RR`（实时时间片轮转）、`SCHED_BATCH`（批处理）、`SCHED_IDLE`（空闲）或 `SCHED_DEADLINE`（截止时间调度）。
- `param`：指向 `struct sched_param` 的指针，包含调度参数（主要是 `sched_priority` 和扩展的实时参数）。

**返回值：** `sched_setscheduler` 成功返回 `0`，失败返回 `-1`。`sched_getscheduler` 成功返回调度策略值，失败返回 `-1`。

**示例用法：**
```c
struct sched_param sp = { .sched_priority = 50 };
if (sched_setscheduler(0, SCHED_FIFO, &sp) == -1)
    perror("sched_setscheduler");
```

**实现原理：** 内核修改进程的调度策略和优先级。实时策略（`SCHED_FIFO`/`SCHED_RR`）需要 `CAP_SYS_NICE` 能力，且优先级范围为 1-99（数值越大优先级越高）。调度器在任务切换时会根据策略和优先级选择下一个运行的线程[reference:1]。

**线程安全提示：** 进程级调度策略影响所有线程，在多线程环境中设置策略时需要谨慎考虑。

---

### 3. `sched_setparam` / `sched_getparam`

**函数原型：**
```c
int sched_setparam(pid_t pid, const struct sched_param *param);
int sched_getparam(pid_t pid, struct sched_param *param);
```

**作用：** 仅修改进程的调度优先级（不改变调度策略）。

**参数：**
- `pid`：目标进程 ID。若为 `0`，则作用于当前进程。
- `param`：指向 `struct sched_param` 的指针，包含优先级值。

**返回值：** 成功返回 `0`，失败返回 `-1`。

**示例用法：**
```c
struct sched_param sp;
sched_getparam(0, &sp);
sp.sched_priority += 10;
sched_setparam(0, &sp);
```

**实现原理：** 系统调用直接修改进程的静态优先级。对于 `SCHED_FIFO`/`SCHED_RR` 策略，优先级必须在 1-99 之间；对于其他策略，优先级通常为 0[reference:2]。

**线程安全提示：** 同 `sched_setscheduler`。

---

### 4. `sched_get_priority_max` / `sched_get_priority_min`

**函数原型：**
```c
int sched_get_priority_max(int policy);
int sched_get_priority_min(int policy);
```

**作用：** 返回指定调度策略下允许的最高和最低优先级。

**参数：**
- `policy`：调度策略（`SCHED_FIFO`、`SCHED_RR` 等）。

**返回值：** 成功返回优先级值，失败返回 `-1`。

**示例用法：**
```c
int max = sched_get_priority_max(SCHED_FIFO);  // 通常返回 99
int min = sched_get_priority_min(SCHED_FIFO);  // 通常返回 1
```

**实现原理：** 从内核的调度策略定义中读取优先级范围。对于实时策略，Linux 固定为 1-99；非实时策略通常返回 0。

**线程安全提示：** 纯只读操作，安全。

---

### 5. `sched_rr_get_interval`

**函数原型：**
```c
int sched_rr_get_interval(pid_t pid, struct timespec *interval);
```

**作用：** 获取指定进程的时间片长度（仅对 `SCHED_RR` 策略有效）。

**参数：**
- `pid`：目标进程 ID。若为 `0`，则作用于当前进程。
- `interval`：输出参数，返回时间片（秒和纳秒）。

**返回值：** 成功返回 `0`，失败返回 `-1`。

**示例用法：**
```c
struct timespec ts;
if (sched_rr_get_interval(0, &ts) == 0)
    printf("Time quantum: %ld.%09ld\n", ts.tv_sec, ts.tv_nsec);
```

**实现原理：** 内核返回 `SCHED_RR` 策略的时间片长度。Linux 默认时间片为 100 毫秒，但可通过 `/proc/sys/kernel/sched_rr_timeslice_ms` 调整。

**线程安全提示：** 安全。

---

### 6. `sched_yield`

**函数原型：**
```c
int sched_yield(void);
```

**作用：** 让当前线程主动放弃 CPU，使同优先级或更高优先级的其他线程有机会运行。

**返回值：** 成功返回 `0`，失败返回 `-1`。

**示例用法：**
```c
while (!work_done) {
    if (try_do_work() == 0)
        sched_yield();   // 没有工作可做，让出 CPU
}
```

**实现原理：** 系统调用将当前线程移动到其优先级队列的末尾，调度器选择下一个可运行线程。对于实时策略（`SCHED_FIFO`/`SCHED_RR`），`sched_yield()` 会明确让出 CPU；对于 `SCHED_OTHER`，行为由调度器决定，通常也会让出 CPU[reference:3]。

**线程安全提示：** 安全。只影响调用线程。

---

### 7. `sched_getcpu`

**函数原型：**
```c
int sched_getcpu(void);
```

**作用：** 返回当前线程正在运行的 CPU 编号。

**返回值：** 成功返回 CPU 编号（非负整数），失败返回 `-1`。

**示例用法：**
```c
int cpu = sched_getcpu();
printf("Running on CPU %d\n", cpu);
```

**实现原理：** `sched_getcpu()` 等价于 `getcpu(2)` 系统调用的封装，从内核获取当前 CPU 信息[reference:4]。

**线程安全提示：** 安全。但 CPU 编号仅在调用时刻有效，调用后线程可能被迁移（除非绑定了 CPU 亲和性）。

---

## 二、结构体详解

### 1. `struct sched_param`

**定义：**
```c
struct sched_param {
    int sched_priority;                 // 调度优先级
    int sched_ss_low_priority;          // 低优先级（零星服务器）
    struct timespec sched_ss_repl_period;  // 补充周期
    struct timespec sched_ss_init_budget;  // 初始预算
    int sched_ss_max_repl;               // 最大补充数
};
```

**作用：** 存储调度策略所需的参数。对于 `SCHED_FIFO` 和 `SCHED_RR`，唯一必需的成员是 `sched_priority`；对于零星服务器策略（`SCHED_SPORADIC`），则使用全部字段。

**成员详解：**
- `sched_priority`：调度优先级，实时策略有效范围为 1-99（值越大优先级越高）。`SCHED_OTHER` 等非实时策略通常忽略此字段。
- `sched_ss_low_priority`：零星服务器的低优先级。
- `sched_ss_repl_period`：预算补充周期。
- `sched_ss_init_budget`：初始预算（时间长度）。
- `sched_ss_max_repl`：最大挂起的补充次数。

---

### 2. `cpu_set_t`

**定义：** 不透明类型，通常实现为一个位掩码。不应直接访问其成员，而应使用 `CPU_*` 系列宏进行操作。

**作用：** 表示 CPU 集合，用于 CPU 亲和性操作。

**限制：** `CPU_SETSIZE` 宏定义了 `cpu_set_t` 能表示的最大 CPU 编号，默认值为 1024。

---

### 3. `struct timespec`（从 `<time.h>` 引入）

**定义：**
```c
struct timespec {
    time_t tv_sec;   // 秒
    long   tv_nsec;  // 纳秒
};
```

**作用：** 表示高精度时间值，用于 `sched_rr_get_interval` 等函数。

---

## 三、宏定义详解

### 1. 调度策略常量

| 宏名称 | 作用 |
|--------|------|
| `SCHED_OTHER` | 标准分时调度策略（默认），适合大多数普通应用。 |
| `SCHED_FIFO` | 实时先进先出策略，线程按优先级排队运行，直到主动让出或被更高优先级任务抢占。 |
| `SCHED_RR` | 实时时间片轮转策略，类似于 `SCHED_FIFO`，但同优先级线程会按时间片轮转。 |
| `SCHED_BATCH` | Linux 特有，适用于非交互式批处理进程。 |
| `SCHED_IDLE` | Linux 特有，用于极低优先级的后台任务。 |
| `SCHED_DEADLINE` | Linux 特有，基于截止时间（Earliest Deadline First）的调度策略。 |
| `SCHED_SPORADIC` | POSIX 零星服务器调度策略（需要系统支持）。 |

---

### 2. CPU 集合操作宏

| 宏名称 | 作用 |
|--------|------|
| `CPU_ZERO(set)` | 清空 CPU 集合，使其不包含任何 CPU。 |
| `CPU_SET(cpu, set)` | 将指定 CPU 添加到集合。 |
| `CPU_CLR(cpu, set)` | 从集合中移除指定 CPU。 |
| `CPU_ISSET(cpu, set)` | 测试指定 CPU 是否在集合中。 |
| `CPU_COUNT(set)` | 返回集合中 CPU 的数量。 |
| `CPU_EQUAL(set1, set2)` | 比较两个集合是否相等。 |
| `CPU_AND(dest, src1, src2)` | 对两个集合执行按位与，结果存入 `dest`。 |
| `CPU_OR(dest, src1, src2)` | 对两个集合执行按位或。 |
| `CPU_XOR(dest, src1, src2)` | 对两个集合执行按位异或。 |
| `CPU_ALLOC(num_cpus)` | 动态分配一个 CPU 集合（适用于大型集合）。 |
| `CPU_ALLOC_SIZE(num_cpus)` | 返回动态分配所需的大小（字节数）。 |
| `CPU_FREE(set)` | 释放动态分配的 CPU 集合。 |

**使用注意事项：** 这些宏可能多次计算其参数，因此传入的 `cpu` 值不应有副作用[reference:5]。动态分配版本（`CPU_ALLOC` 等）用于支持超过 `CPU_SETSIZE` 的场景。

---

### 3. 其他常量

| 宏名称 | 作用 |
|--------|------|
| `CPU_SETSIZE` | `cpu_set_t` 能表示的最大 CPU 编号加 1（通常为 1024）。 |
| `_GNU_SOURCE` | 功能测试宏（feature test macro），需要在包含 `<sched.h>` 之前定义，以启用 GNU 扩展（如 `CPU_*` 宏和亲和性函数）。 |

---

## 四、类型定义

- `cpu_set_t`：CPU 集合类型，用于亲和性操作。
- `struct sched_param`：调度参数结构体。
- `struct timespec`：高精度时间结构体。
- `pid_t`：进程 ID 类型（从 `<sys/types.h>` 引入）。
- `time_t`：时间类型（从 `<sys/types.h>` 引入）。

---

## 五、模板声明

`<sched.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
