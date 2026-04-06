## `<linux/futex.h>` 头文件详解（Linux 特有接口）

`<linux/futex.h>` 是 Linux 特有的头文件，定义了 **futex**（Fast Userspace muTEX）系统调用使用的操作码、标志位和相关数据结构。futex 是 Linux 内核提供的一种高效同步原语，允许用户空间程序在无竞争时完全在用户态完成锁操作，仅在需要阻塞或唤醒时才陷入内核。它被广泛用于实现 pthread 互斥锁、条件变量、读写锁等高级同步机制。该机制由 Linux 2.5.7 引入内核[reference:0]。

---

## 一、函数详解

### 1. futex 系统调用

**函数原型：**
```c
long syscall(SYS_futex, uint32_t *uaddr, int futex_op, uint32_t val,
             const struct timespec *timeout, uint32_t *uaddr2, uint32_t val3);
```

**作用：** 提供快速用户空间锁定的核心机制。futex 是一个 32 位的值（futex word），其地址作为系统调用的参数。系统调用根据不同的 `futex_op` 执行等待、唤醒、重入队列等操作。futex 在所有平台上（包括 64 位系统）都是 32 位大小[reference:1]。

**参数：**
- `uaddr`：指向 futex 变量的指针（32 位无符号整数）。
- `futex_op`：操作码（见宏定义部分），决定要执行的操作类型。
- `val`：操作参数，具体含义取决于 `futex_op`。对于 `FUTEX_WAIT`，它是期望值；对于 `FUTEX_WAKE`，它是最大唤醒线程数。
- `timeout`：指向 `struct timespec` 的指针，用于指定等待的超时时间（绝对时间）。
- `uaddr2`：第二个 futex 地址，用于 `FUTEX_REQUEUE` 等操作。
- `val3`：第三个操作参数，用于 `FUTEX_WAKE_OP` 等操作。

**返回值：**
- `FUTEX_WAIT`：成功被唤醒时返回 0；被信号中断返回 `-EINTR`；超时返回 `-ETIMEDOUT`。
- `FUTEX_WAKE`：返回成功唤醒的线程数（可能为 0）。
- `FUTEX_REQUEUE`：返回成功唤醒的线程数。
- `FUTEX_CMP_REQUEUE`：返回成功唤醒的线程数。
- 其他操作返回值见具体 man 手册。

**示例用法：**
```c
#include <linux/futex.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdint.h>
#include <stdio.h>

int main() {
    uint32_t futex_var = 0;
    int ret;

    // 期望 futex_var == 0，若成立则阻塞等待唤醒
    ret = syscall(SYS_futex, &futex_var, FUTEX_WAIT, 0, NULL, NULL, 0);
    if (ret == -1) perror("futex_wait");

    // 唤醒等待在 futex_var 上的一个线程
    ret = syscall(SYS_futex, &futex_var, FUTEX_WAKE, 1, NULL, NULL, 0);
    if (ret == -1) perror("futex_wake");

    return 0;
}
```

**实现原理：**
当用户空间执行一个锁操作时，通常使用原子操作（如 `cmpxchg`）在用户态尝试获取锁。如果锁未被争用，整个过程在用户态完成，无需陷入内核。只有当锁被争用（即需要阻塞）时，才会调用 `futex()` 系统调用进入内核。内核将当前线程加入等待队列，并在锁释放时唤醒等待队列中的线程。这种"用户态优先，内核态辅助"的设计避免了大量无谓的系统调用，从而实现了高性能同步[reference:2]。

**线程安全提示：**
futex 系统调用本身是线程安全的。但 futex 变量的用户态操作（如原子增减）需要正确使用原子操作（如 GCC 的 `__sync_*` 或 C11 的 `atomic_*`），以避免数据竞争。

---

### 2. futex2 系列（Linux 5.16+）

`<linux/futex.h>` 也定义了 futex2 的相关操作，如 `futex_waitv`。

#### futex_waitv 系统调用

**函数原型：**
```c
long syscall(SYS_futex_waitv, struct futex_waitv *waiters, unsigned int nr_futexes,
             unsigned int flags, struct timespec *timeout, clockid_t clockid);
```

**作用：** 等待一个 futex 数组中的任意一个满足条件，是 `futex()` 的扩展，支持同时等待多个 futex。

**参数：**
- `waiters`：指向 `struct futex_waitv` 数组的指针。
- `nr_futexes`：数组大小，必须在 1 到 128 之间。
- `flags`：必须为 0（预留未来扩展）。
- `timeout`：指向绝对超时时间的指针。
- `clockid`：时钟类型，支持 `CLOCK_MONOTONIC` 和 `CLOCK_REALTIME`。

**返回值：**
- 成功：返回被唤醒的 futex 在数组中的索引。
- 失败：返回 -1 并设置 `errno`（如 `-EAGAIN`、`-ETIMEOUT`、`-EFAULT` 等）。

**示例用法：**
```c
struct futex_waitv waiters[2] = {
    { .val = 0, .uaddr = (__u64)&futex1, .flags = FUTEX_32 | FUTEX_PRIVATE_FLAG },
    { .val = 1, .uaddr = (__u64)&futex2, .flags = FUTEX_32 | FUTEX_PRIVATE_FLAG }
};
int ret = syscall(SYS_futex_waitv, waiters, 2, 0, NULL, CLOCK_MONOTONIC);
```

**实现原理：**
该接口是初代 `futex` 系统调用的后续版本，旨在克服原有接口的限制。内核依次检查每个 `waiters` 项：如果 `uaddr` 当前值与 `val` 不一致，立即返回 `-EAGAIN`；如果所有项验证通过，则当前线程进入睡眠，直到某个 futex 被唤醒或超时。每次 `read` 将计数器值拷贝到用户空间并清零计数器。

**线程安全提示：**
线程安全。对同一 `fd` 的并发 `read` 需要外部同步，建议单线程读取。

---

## 二、结构体详解

### 1. struct robust_list

**定义：**
```c
struct robust_list {
    struct robust_list __user *next;
};
```

**作用：** robust futex 列表中的节点。robust futex 是一种特殊的 futex，用于在线程意外终止时自动清理未解锁的锁[reference:3]。

**成员详解：**
- `next`：指向列表中下一个 `robust_list` 节点的指针。

**使用注意事项：** 该结构体是系统调用 ABI 的一部分，用户空间的数据结构必须与此兼容，且不可随意更改[reference:4]。

---

### 2. struct robust_list_head

**定义：**
```c
struct robust_list_head {
    struct robust_list list;              // 列表头，空时指向自身
    long futex_offset;                    // futex 字段在用户锁结构中的偏移量
    struct robust_list __user *list_op_pending;  // 待处理锁的地址
};
```

**作用：** per-thread robust futex 列表的头部，用于管理线程持有的所有 robust futex 锁。内核在线程退出时会遍历此列表，自动清理未解锁的 futex[reference:5]。

**成员详解：**
- `list`：链表头，空时指向自身。内核通过此链表找到所有线程持有的锁。
- `futex_offset`：用户锁结构体中 futex 字段相对于锁结构起始位置的偏移量，由用户空间设置，内核据此定位 futex 字段。
- `list_op_pending`：指向正在尝试获取的锁的地址，用于处理竞态条件。

**使用注意事项：** 该结构体是系统调用 ABI 的一部分，任何不兼容的更改都必须先与 glibc 开发者沟通[reference:6]。

---

### 3. struct futex_waitv

**定义：**
```c
struct futex_waitv {
    __u64 val;           // 期望值
    __u64 uaddr;         // futex 地址
    __u32 flags;         // 标志位（类型和大小）
    __u32 __reserved;    // 预留字段，必须为 0
};
```

**作用：** 用于 `futex_waitv()` 系统调用，描述一个要等待的 futex[reference:7]。

**成员详解：**
- `val`：期望的 futex 值。只有当 `uaddr` 指向的 futex 当前值等于 `val` 时，才会被纳入等待。
- `uaddr`：futex 变量的用户空间地址。
- `flags`：标志位，用于指定 futex 类型（如 `FUTEX_PRIVATE_FLAG`）和大小（如 `FUTEX_32`）[reference:8]。
- `__reserved`：预留字段，必须设为 0，用于未来扩展[reference:9]。

**使用注意事项：**
- 最多支持 128 个 `futex_waitv` 项[reference:10]。
- 如果用户空间拥有 32 位指针，需要显式转换以保证高位清零，`uintptr_t` 可正确处理 32/64 位指针[reference:11]。
- 每个 `futex_waitv` 项的 `uaddr` 地址必须有效，否则返回 `-EFAULT`[reference:12]。

---

## 三、宏定义详解

### 1. futex 操作码

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `FUTEX_WAIT` | 0 | 如果 `*uaddr == val`，则阻塞当前线程等待唤醒[reference:13] |
| `FUTEX_WAKE` | 1 | 唤醒最多 `val` 个等待在此 futex 上的线程[reference:14] |
| `FUTEX_FD` | 2 | 已废弃（曾用于关联文件描述符）[reference:15] |
| `FUTEX_REQUEUE` | 3 | 将等待在 `uaddr` 上的线程移到等待 `uaddr2` 的队列中[reference:16] |
| `FUTEX_CMP_REQUEUE` | 4 | 带比较的 `FUTEX_REQUEUE`，仅当 `*uaddr == val3` 时执行重入队操作[reference:17] |
| `FUTEX_WAKE_OP` | 5 | 原子地执行操作和比较，根据结果决定是否唤醒等待者[reference:18] |
| `FUTEX_LOCK_PI` | 6 | 优先级继承（Priority Inheritance）锁操作[reference:19] |
| `FUTEX_UNLOCK_PI` | 7 | 优先级继承解锁操作[reference:20] |
| `FUTEX_TRYLOCK_PI` | 8 | 尝试获取优先级继承锁（非阻塞）[reference:21] |
| `FUTEX_WAIT_BITSET` | 9 | 带位掩码的 `FUTEX_WAIT` |
| `FUTEX_WAKE_BITSET` | 10 | 带位掩码的 `FUTEX_WAKE` |
| `FUTEX_WAIT_REQUEUE_PI` | 11 | 等待并重入队到 PI 锁队列 |
| `FUTEX_CMP_REQUEUE_PI` | 12 | 带比较的重入队到 PI 锁队列 |
| `FUTEX_LOCK_PI2` | 13 | PI 锁操作（带超时） |

---

### 2. FUTEX_WAKE_OP 相关宏

#### 2.1 操作宏（op）

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `FUTEX_OP_SET` | 0 | `*(int *)UADDR2 = OPARG;`[reference:22] |
| `FUTEX_OP_ADD` | 1 | `*(int *)UADDR2 += OPARG;`[reference:23] |
| `FUTEX_OP_OR` | 2 | `*(int *)UADDR2 \|= OPARG;`[reference:24] |
| `FUTEX_OP_ANDN` | 3 | `*(int *)UADDR2 &= ~OPARG;`[reference:25] |
| `FUTEX_OP_XOR` | 4 | `*(int *)UADDR2 ^= OPARG;`[reference:26] |
| `FUTEX_OP_OPARG_SHIFT` | 8 | 使用 `(1 << OPARG)` 代替 `OPARG`[reference:27] |

#### 2.2 比较宏（cmp）

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `FUTEX_OP_CMP_EQ` | 0 | `if (oldval == CMPARG) wake`[reference:28] |
| `FUTEX_OP_CMP_NE` | 1 | `if (oldval != CMPARG) wake`[reference:29] |
| `FUTEX_OP_CMP_LT` | 2 | `if (oldval < CMPARG) wake`[reference:30] |
| `FUTEX_OP_CMP_LE` | 3 | `if (oldval <= CMPARG) wake`[reference:31] |
| `FUTEX_OP_CMP_GT` | 4 | `if (oldval > CMPARG) wake`[reference:32] |
| `FUTEX_OP_CMP_GE` | 5 | `if (oldval >= CMPARG) wake`[reference:33] |

#### 2.3 辅助宏

| 宏名称 | 作用 |
|--------|------|
| `FUTEX_OP(op, oparg, cmp, cmparg)` | 将操作和比较编码为 32 位值，用于 `FUTEX_WAKE_OP` 的 `val3` 参数[reference:34] |

---

### 3. 标志位宏

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `FUTEX_PRIVATE_FLAG` | 128 | 标记为私有 futex（进程内共享），允许内核进行优化[reference:35] |
| `FUTEX_CLOCK_REALTIME` | 256 | 使用 `CLOCK_REALTIME` 作为超时时钟（默认使用 `CLOCK_MONOTONIC`） |
| `FUTEX_32` | 2 | 标记 futex 为 32 位（用于 `futex_waitv`）[reference:36] |

---

### 4. robust futex 相关宏

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `FUTEX_WAITERS` | 0x80000000 | 表示有等待者在此 futex 上[reference:37] |
| `FUTEX_OWNER_DIED` | 0x40000000 | 表示持有 futex 的线程已退出且未解锁，内核会唤醒等待者[reference:38] |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `struct robust_list` | robust futex 链表节点 |
| `struct robust_list_head` | robust futex 链表头 |
| `struct futex_waitv` | `futex_waitv()` 系统调用的等待项 |

---

## 五、模板声明

`<linux/futex.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口和常量定义保持不变。

---
