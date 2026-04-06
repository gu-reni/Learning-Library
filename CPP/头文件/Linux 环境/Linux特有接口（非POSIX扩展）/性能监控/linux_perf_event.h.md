## `<linux/perf_event.h>` 头文件详解（Linux 内核接口）

`<linux/perf_event.h>` 是 Linux 内核提供的性能事件监控（perf_event）子系统核心头文件，定义了 `perf_event_open()` 系统调用的参数结构体、事件类型、配置标志、采样格式等。通过 perf_event，用户态程序可以精确地测量 CPU 周期、指令退休、缓存未命中、分支预测错误、硬件断点、软件事件（如上下文切换）、以及跟踪点（tracepoint）等。该子系统是 Linux 性能分析工具（如 `perf` 命令）的基础，由 Linux 2.6.31 引入内核。

---

## 一、函数详解

### 1. perf_event_open 系统调用

**函数原型：**
```c
int perf_event_open(struct perf_event_attr *attr,
                    pid_t pid, int cpu, int group_fd,
                    unsigned long flags);
```

**作用：** 创建一个性能事件文件描述符，用于测量指定进程（或 CPU）上的硬件/软件事件。通过 `read()` 读取事件计数，通过 `mmap()` 共享内存缓冲区获取采样记录。

**参数：**
- `attr`：指向 `struct perf_event_attr` 的指针，描述事件类型、采样周期、标志等配置。
- `pid`：目标进程 PID。若为 -1 则测量所有进程；若为 0 则测量当前进程。
- `cpu`：目标 CPU 编号。若为 -1 则测量任意 CPU。
- `group_fd`：事件组的 leader 文件描述符。若为 -1 则创建新的事件组，否则将新事件添加到已有组。
- `flags`：控制行为，可取 `0` 或 `PERF_FLAG_FD_CLOEXEC`（设置 close-on-exec）、`PERF_FLAG_PID_CGROUP`（pid 解释为 cgroup 文件描述符）等。

**返回值：** 成功返回文件描述符，失败返回 -1 并设置 `errno`。

**示例用法（统计当前进程的 CPU 周期）：**
```c
#include <linux/perf_event.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    struct perf_event_attr attr = {
        .type = PERF_TYPE_HARDWARE,
        .config = PERF_COUNT_HW_CPU_CYCLES,
        .size = sizeof(struct perf_event_attr),
        .disabled = 1,   // 初始禁用
        .exclude_kernel = 1,  // 不测量内核
    };
    int fd = syscall(SYS_perf_event_open, &attr, 0, -1, -1, 0);
    if (fd == -1) {
        perror("perf_event_open");
        return 1;
    }
    // 开始计数
    ioctl(fd, PERF_EVENT_IOC_ENABLE, 0);
    // ... 执行要测量的代码 ...
    ioctl(fd, PERF_EVENT_IOC_DISABLE, 0);
    long long count;
    read(fd, &count, sizeof(count));
    printf("CPU cycles: %lld\n", count);
    close(fd);
    return 0;
}
```

**实现原理：**
`perf_event_open()` 是系统调用，内核分配 `struct perf_event` 对象，根据 `attr` 中的类型和配置选择对应的 PMU（Performance Monitoring Unit）或软件事件源。事件被挂载到目标进程或 CPU 的上下文中，当事件发生时，内核会递增计数器或根据采样周期将样本写入环形缓冲区。文件描述符用于控制（`ioctl`）、读取计数或映射缓冲区。

**线程安全提示：**
系统调用本身是线程安全的。但多个线程对同一个 perf event 文件描述符进行并发操作（如同时 `read` 或 `ioctl`）可能导致数据竞争，建议使用锁或确保单线程访问。

---

## 二、结构体详解

### 1. struct perf_event_attr

**定义（精简版，实际字段很多，仅列出常用）：**
```c
struct perf_event_attr {
    __u32 type;                 // 事件类型（PERF_TYPE_*）
    __u32 size;                 // 结构体大小（用于版本兼容）
    __u64 config;               // 具体事件配置（取决于 type）
    union {
        __u64 sample_period;    // 采样周期（事件发生多少次触发一次采样）
        __u64 sample_freq;      // 采样频率（每秒采样次数）
    };
    __u64 sample_type;          // 采样记录中包含的信息（PERF_SAMPLE_* 位掩码）
    __u64 read_format;          // read() 返回的数据格式（PERF_FORMAT_*）
    __u64 disabled       : 1;   // 初始禁用
    __u64 inherit        : 1;   // 子进程继承此事件
    __u64 pinned         : 1;   // 事件必须始终在 PMU 上
    __u64 exclusive      : 1;   // 独占 PMU
    __u64 exclude_user    : 1;   // 排除用户空间
    __u64 exclude_kernel  : 1;   // 排除内核空间
    __u64 exclude_hv      : 1;   // 排除虚拟机
    __u64 exclude_idle    : 1;   // 排除空闲任务
    __u64 mmap           : 1;   // 记录 mmap 事件
    __u64 comm           : 1;   // 记录进程名变更
    __u64 freq           : 1;   // 使用频率而非周期
    __u64 inherit_stat   : 1;   // 继承统计信息
    __u64 enable_on_exec : 1;   // exec 时启用事件
    __u64 task           : 1;   // 跟踪 fork/exit 事件
    __u64 watermark      : 1;   // 使用水位中断
    __u64 precise_ip     : 2;   // 精确度要求（0-3）
    __u64 mmap_data      : 1;   // 记录数据 mmap
    __u64 sample_id_all  : 1;   // 在所有样本中包含 ID
    __u64 exclude_host   : 1;   // 排除主机（用于 KVM）
    __u64 exclude_guest  : 1;   // 排除客户机
    __u64 exclude_callchain_kernel : 1;  // 排除内核调用链
    __u64 exclude_callchain_user   : 1;  // 排除用户调用链
    __u64 mmap2          : 1;   // 增强的 mmap 记录
    __u64 comm_exec      : 1;   // 记录 exec 后的 comm 变化
    __u64 use_clockid    : 1;   // 使用 clockid 作为时间基准
    __u64 context_switch : 1;   // 记录上下文切换
    __u64 write_backward : 1;   // 向后写入环形缓冲区
    __u64 namespaces     : 1;   // 记录命名空间信息
    __u64 ksymbol        : 1;   // 记录内核符号事件
    __u64 bpf_event      : 1;   // 记录 BPF 事件
    __u64 aux_output     : 1;   // 允许 AUX 输出
    __u64 cgroup         : 1;   // 支持 cgroup 事件
    __u64 text_poke      : 1;   // 记录文本修改事件
    __u64 build_id       : 1;   // 记录 build ID
    __u64 __reserved_1   : 1;   // 保留
    __u64 __reserved_2   : 30;  // 保留
    union {
        __u16 bp_len;           // 硬件断点长度
        __u64 config1;          // 扩展配置（取决于 type）
        __u64 config2;
    };
    __u64 branch_sample_type;   // 分支采样类型（PERF_SAMPLE_BRANCH_*）
    __u64 sample_regs_user;     // 用户寄存器掩码
    __u32 sample_stack_user;    // 用户栈大小
    __s32 clockid;              // 时钟 ID（当 use_clockid=1 时）
    __u64 sample_regs_intr;     // 中断时保存的寄存器掩码
    __u32 aux_watermark;        // AUX 区域水位
    __u16 sample_max_stack;     // 最大栈深度
    __u16 __reserved_2;         // 保留
    __u32 aux_sample_size;      // AUX 样本大小
    __u32 __reserved_3;         // 保留
};
```

**作用：** 配置性能事件的所有参数，传递给 `perf_event_open()`。

**常用成员说明：**
- `type`：事件大类，可选 `PERF_TYPE_HARDWARE`、`PERF_TYPE_SOFTWARE`、`PERF_TYPE_TRACEPOINT`、`PERF_TYPE_HW_CACHE`、`PERF_TYPE_RAW` 等。
- `config`：具体事件 ID（如 `PERF_COUNT_HW_CPU_CYCLES`）或原始事件码。
- `sample_period` / `sample_freq`：采样周期（事件次数）或采样频率（每秒次数）。若为 0，则只提供累加计数（通过 `read` 获取）。
- `sample_type`：决定每次采样记录中包含哪些信息，如时间戳、PID、IP、调用链等。
- `read_format`：决定 `read()` 返回的数据格式（如 `PERF_FORMAT_TOTAL_TIME_ENABLED`）。
- 标志位（`disabled`、`exclude_user` 等）控制事件行为和过滤。
- `precise_ip`：控制采样时指令指针的精确度（0=尽可能精确，3=必须精确）。

---

### 2. struct perf_event_mmap_page

**定义（部分）：**
```c
struct perf_event_mmap_page {
    __u32 version;          // 版本号
    __u32 compat_version;   // 兼容版本
    __u32 lock;             // 自旋锁（seqlock）
    __u32 index;            // 硬件计数器索引
    __s64 offset;           // 添加到计数器值的偏移量
    __u64 time_enabled;     // 事件启用时间（ns）
    __u64 time_running;     // 事件运行时间（ns）
    union {
        __u64   capabilities;   // 能力位掩码
        struct {
            __u64 cap_usr_time  : 1,
                  cap_usr_rdpmc : 1,
                  cap_user_time : 1,
                  cap_user_time_zero : 1,
                  cap_user_rdpmc   : 1,
                  cap_user_rdpmc_64: 1,
                  cap_user_time_short: 1,
                  cap_____reserved: 57;
        };
    };
    __u16 pmc_width;        // 计数器宽度（位）
    __u16 time_shift;       // 时间戳转换移位
    __u32 time_mult;        // 时间戳转换乘数
    __u64 time_zero;        // 时间零点
    __u32 size;             // 该结构体大小
    __u32 __reserved_1;     // 保留
    __u64 time_cycles;      // CPU 周期时间戳（如果支持）
    __u64 time_mask;        // 时间周期掩码
    __u32 __reserved_2[126];// 填充到 1024 字节
    __u64 data_head;        // 数据头部指针（用户态只读）
    __u64 data_tail;        // 数据尾部指针（用户态可写）
    // ... 后续为实际数据区域
};
```

**作用：** 通过 `mmap` 映射到用户空间的内存页，提供无锁的计数器读取和环形缓冲区访问。

---

### 3. struct perf_event_header

**定义：**
```c
struct perf_event_header {
    __u32 type;     // 事件记录类型（PERF_RECORD_*）
    __u16 misc;     // 额外标志（如 PERF_RECORD_MISC_*）
    __u16 size;     // 记录总大小（字节）
};
```

**作用：** 每个采样记录的开头部分，标识记录类型和长度。

---

### 4. 采样记录结构体（部分）

根据 `sample_type` 的不同，采样记录会包含不同的字段，常见的有：

- `struct perf_record_sample`：包含 IP、PID、时间戳、调用链等。
- `struct perf_record_mmap`：内存映射事件。
- `struct perf_record_comm`：进程名变更事件。
- `struct perf_record_fork`：fork/exit 事件。
- `struct perf_record_switch`：上下文切换事件。

这些结构体在 `<linux/perf_event.h>` 中也有定义，但通常通过 `perf_event_header` 和 `sample_type` 动态解析，不固定格式。

---

## 三、宏定义详解

### 1. 事件类型（用于 `perf_event_attr.type`）

| 宏名称 | 作用 |
|--------|------|
| `PERF_TYPE_HARDWARE` | 硬件事件（如 CPU 周期、指令退休） |
| `PERF_TYPE_SOFTWARE` | 软件事件（如上下文切换、缺页异常） |
| `PERF_TYPE_TRACEPOINT` | 内核 tracepoint 事件 |
| `PERF_TYPE_HW_CACHE` | 硬件缓存事件（L1/L2/LLC 访问/未命中） |
| `PERF_TYPE_RAW` | 原始 PMU 事件（直接指定 PMU 事件码） |
| `PERF_TYPE_BREAKPOINT` | 硬件断点事件（内存地址读写） |

---

### 2. 硬件事件 ID（用于 `perf_event_attr.config`，当 `type = PERF_TYPE_HARDWARE`）

| 宏名称 | 作用 |
|--------|------|
| `PERF_COUNT_HW_CPU_CYCLES` | CPU 周期数 |
| `PERF_COUNT_HW_INSTRUCTIONS` | 指令退休数 |
| `PERF_COUNT_HW_CACHE_REFERENCES` | 缓存访问次数 |
| `PERF_COUNT_HW_CACHE_MISSES` | 缓存未命中次数 |
| `PERF_COUNT_HW_BRANCH_INSTRUCTIONS` | 分支指令数 |
| `PERF_COUNT_HW_BRANCH_MISSES` | 分支预测错误次数 |
| `PERF_COUNT_HW_BUS_CYCLES` | 总线周期数 |
| `PERF_COUNT_HW_STALLED_CYCLES_FRONTEND` | 前端停顿周期 |
| `PERF_COUNT_HW_STALLED_CYCLES_BACKEND` | 后端停顿周期 |
| `PERF_COUNT_HW_REF_CPU_CYCLES` | 参考 CPU 周期（固定频率） |

---

### 3. 软件事件 ID（用于 `perf_event_attr.config`，当 `type = PERF_TYPE_SOFTWARE`）

| 宏名称 | 作用 |
|--------|------|
| `PERF_COUNT_SW_CPU_CLOCK` | CPU 时钟（纳秒） |
| `PERF_COUNT_SW_TASK_CLOCK` | 任务时钟（进程执行时间） |
| `PERF_COUNT_SW_PAGE_FAULTS` | 缺页异常总数 |
| `PERF_COUNT_SW_CONTEXT_SWITCHES` | 上下文切换次数 |
| `PERF_COUNT_SW_CPU_MIGRATIONS` | CPU 迁移次数 |
| `PERF_COUNT_SW_PAGE_FAULTS_MIN` | 次要缺页异常 |
| `PERF_COUNT_SW_PAGE_FAULTS_MAJ` | 主要缺页异常 |
| `PERF_COUNT_SW_ALIGNMENT_FAULTS` | 对齐错误 |
| `PERF_COUNT_SW_EMULATION_FAULTS` | 仿真错误 |
| `PERF_COUNT_SW_DUMMY` | 占位符事件 |
| `PERF_COUNT_SW_BPF_OUTPUT` | BPF 输出事件 |

---

### 4. 采样类型（`sample_type` 位掩码）

| 宏名称 | 作用 |
|--------|------|
| `PERF_SAMPLE_IP` | 指令指针 |
| `PERF_SAMPLE_TID` | 进程/线程 ID |
| `PERF_SAMPLE_TIME` | 时间戳 |
| `PERF_SAMPLE_ADDR` | 内存地址 |
| `PERF_SAMPLE_ID` | 事件 ID |
| `PERF_SAMPLE_STREAM_ID` | 流 ID |
| `PERF_SAMPLE_CPU` | CPU 编号 |
| `PERF_SAMPLE_PERIOD` | 当前采样周期 |
| `PERF_SAMPLE_CALLCHAIN` | 调用链 |
| `PERF_SAMPLE_RAW` | 原始数据（如 tracepoint 字段） |
| `PERF_SAMPLE_BRANCH_STACK` | 分支栈 |
| `PERF_SAMPLE_REGS_USER` | 用户态寄存器 |
| `PERF_SAMPLE_STACK_USER` | 用户态栈 |
| `PERF_SAMPLE_WEIGHT` | 内存访问权重 |
| `PERF_SAMPLE_DATA_SRC` | 数据源信息 |
| `PERF_SAMPLE_IDENTIFIER` | 标识符（用于排序） |
| `PERF_SAMPLE_TRANSACTION` | 事务内存状态 |
| `PERF_SAMPLE_REGS_INTR` | 中断时的寄存器 |
| `PERF_SAMPLE_PHYS_ADDR` | 物理地址 |
| `PERF_SAMPLE_CGROUP` | cgroup 信息 |
| `PERF_SAMPLE_DATA_PAGE_SIZE` | 数据页大小 |
| `PERF_SAMPLE_CODE_PAGE_SIZE` | 代码页大小 |
| `PERF_SAMPLE_WEIGHT_STRUCT` | 扩展权重结构 |

---

### 5. 读取格式（`read_format` 位掩码）

| 宏名称 | 作用 |
|--------|------|
| `PERF_FORMAT_TOTAL_TIME_ENABLED` | 包含 `time_enabled` |
| `PERF_FORMAT_TOTAL_TIME_RUNNING` | 包含 `time_running` |
| `PERF_FORMAT_ID` | 包含事件 ID |
| `PERF_FORMAT_GROUP` | 分组读取（一次读多个计数器） |
| `PERF_FORMAT_LOST` | 包含丢失计数 |
| `PERF_FORMAT_MAX` | 最大值（用于边界检查） |

---

### 6. ioctl 命令（用于 `ioctl(fd, cmd, ...)`）

| 宏名称 | 作用 |
|--------|------|
| `PERF_EVENT_IOC_ENABLE` | 启用事件 |
| `PERF_EVENT_IOC_DISABLE` | 禁用事件 |
| `PERF_EVENT_IOC_REFRESH` | 设置事件的 refresh 计数（用于周期性禁用/启用） |
| `PERF_EVENT_IOC_RESET` | 重置计数器 |
| `PERF_EVENT_IOC_PERIOD` | 动态调整采样周期 |
| `PERF_EVENT_IOC_SET_OUTPUT` | 将事件输出重定向到另一个 fd |
| `PERF_EVENT_IOC_SET_FILTER` | 设置 tracepoint 过滤器（BPF 或字符串） |
| `PERF_EVENT_IOC_ID` | 获取事件 ID |
| `PERF_EVENT_IOC_SET_BPF` | 附加 BPF 程序 |
| `PERF_EVENT_IOC_PAUSE_OUTPUT` | 暂停/恢复 AUX 输出 |
| `PERF_EVENT_IOC_QUERY_BPF` | 查询 BPF 程序信息 |

---

### 7. 记录类型（`perf_event_header.type`）

| 宏名称 | 作用 |
|--------|------|
| `PERF_RECORD_MMAP` | 内存映射事件 |
| `PERF_RECORD_LOST` | 样本丢失 |
| `PERF_RECORD_COMM` | 进程名变更 |
| `PERF_RECORD_EXIT` | 进程退出 |
| `PERF_RECORD_THROTTLE` / `PERF_RECORD_UNTHROTTLE` | 节流状态变化 |
| `PERF_RECORD_FORK` | 进程 fork |
| `PERF_RECORD_READ` | 读取事件组 |
| `PERF_RECORD_SAMPLE` | 普通采样记录 |
| `PERF_RECORD_MMAP2` | 增强的 mmap 记录（包含设备号、inode 等） |
| `PERF_RECORD_AUX` | AUX 跟踪数据 |
| `PERF_RECORD_ITRACE_START` | 指令跟踪开始 |
| `PERF_RECORD_LOST_SAMPLES` | 丢失采样记录 |
| `PERF_RECORD_SWITCH` | 上下文切换（CPU 切换） |
| `PERF_RECORD_SWITCH_CPU_WIDE` | 全 CPU 上下文切换 |
| `PERF_RECORD_NAMESPACES` | 命名空间变更 |
| `PERF_RECORD_KSYMBOL` | 内核符号注册 |
| `PERF_RECORD_BPF_EVENT` | BPF 事件 |
| `PERF_RECORD_CGROUP` | cgroup 事件 |
| `PERF_RECORD_TEXT_POKE` | 内核文本修改 |
| `PERF_RECORD_AUX_OUTPUT_HW_ID` | AUX 输出硬件 ID |
| `PERF_RECORD_MAX` | 最大值（用于边界检查） |

---

### 8. 其他重要标志

| 宏名称 | 作用 |
|--------|------|
| `PERF_FLAG_FD_CLOEXEC` | `perf_event_open()` 的 `flags` 标志，设置 close-on-exec |
| `PERF_FLAG_PID_CGROUP` | 将 `pid` 解释为 cgroup 文件描述符 |
| `PERF_FLAG_FD_OUTPUT` | 将事件输出到 `group_fd` 的缓冲区 |
| `PERF_SAMPLE_BRANCH_*` | 分支采样类型（用户、内核、预测、不预测等） |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `struct perf_event_attr` | 事件属性配置 |
| `struct perf_event_mmap_page` | mmap 共享页 |
| `struct perf_event_header` | 采样记录头 |
| `perf_event_ioctl` | 用于 `ioctl` 命令的枚举（实际为宏） |

---

## 五、模板声明

`<linux/perf_event.h>` 是 Linux 内核专用的 C 头文件，不包含 C++ 模板，且该头文件仅适用于内核空间编程和用户空间通过系统调用使用，不能在用户空间直接包含（除了系统调用参数结构体定义，用户空间程序可以包含此头文件来使用 `perf_event_open` 和相关结构体）。

---
