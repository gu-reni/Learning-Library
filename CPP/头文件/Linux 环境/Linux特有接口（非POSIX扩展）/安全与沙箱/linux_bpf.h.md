## `<linux/bpf.h>` 头文件详解（Linux 内核接口）

`<linux/bpf.h>` 是 Linux 内核提供的 BPF（Berkeley Packet Filter）子系统核心头文件，定义了 BPF 指令集、程序类型、映射类型、辅助函数编号、`bpf()` 系统调用的参数结构体以及各种常量。BPF 允许用户将字节码加载到内核中安全执行，广泛应用于包过滤、性能监控、网络流量控制、安全策略、追踪等领域。

---

## 一、函数详解

BPF 操作通过 `bpf()` 系统调用完成，该头文件定义了命令常量（`BPF_*`）和参数联合体 `union bpf_attr`，并未直接提供 C 函数封装。通常通过 `syscall(SYS_bpf, cmd, attr, size)` 调用。

**常用 `bpf()` 命令：**

| 命令 | 作用 |
|------|------|
| `BPF_MAP_CREATE` | 创建 BPF 映射（map） |
| `BPF_MAP_LOOKUP_ELEM` | 根据键查找映射中的元素 |
| `BPF_MAP_UPDATE_ELEM` | 创建或更新映射中的元素 |
| `BPF_MAP_DELETE_ELEM` | 删除映射中的元素 |
| `BPF_MAP_GET_NEXT_KEY` | 获取下一个键 |
| `BPF_PROG_LOAD` | 加载 BPF 程序字节码 |
| `BPF_OBJ_PIN` | 固定 BPF 对象到 BPF 文件系统 |
| `BPF_OBJ_GET` | 从 BPF 文件系统中获取对象 |
| `BPF_PROG_ATTACH` | 附加 BPF 程序到挂载点 |
| `BPF_PROG_DETACH` | 分离 BPF 程序 |
| `BPF_LINK_CREATE` | 创建 BPF 链接 |
| `BPF_LINK_UPDATE` | 更新 BPF 链接 |

**示例用法（创建 BPF 映射）：**
```c
#include <linux/bpf.h>
#include <sys/syscall.h>
#include <unistd.h>

union bpf_attr attr = {
    .map_type = BPF_MAP_TYPE_HASH,
    .key_size = sizeof(int),
    .value_size = sizeof(int),
    .max_entries = 1024,
};
int fd = syscall(SYS_bpf, BPF_MAP_CREATE, &attr, sizeof(attr));
```

**实现原理：** `bpf()` 是系统调用，内核根据命令执行相应操作，如创建映射、加载程序等。映射和程序在内核中由文件描述符引用，通过 BPF 文件系统可持久化。

**线程安全提示：** `bpf()` 系统调用本身是线程安全的。但多个线程对同一文件描述符的并发操作需用户态同步。

---

## 二、结构体详解

### 1. struct bpf_insn

**定义：**
```c
struct bpf_insn {
    __u8 code;      // 操作码
    __u8 dst_reg:4; // 目标寄存器
    __u8 src_reg:4; // 源寄存器
    __s16 off;      // 偏移量
    __s32 imm;      // 立即数
};
```

**作用：** 表示一条 BPF 指令，BPF 程序由这样的指令数组构成。

### 2. union bpf_attr

**定义：** 一个巨大的联合体，用于向 `bpf()` 系统调用传递参数。不同命令使用不同的成员，例如：
- 创建映射：`map_type`, `key_size`, `value_size`, `max_entries`, `map_flags`, `map_name` 等。
- 加载程序：`prog_type`, `insns`, `insn_cnt`, `license`, `log_buf`, `log_size`, `log_level`, `prog_name` 等。
- 元素操作：`map_fd`, `key`, `value`, `flags`。

由于该联合体非常大（数百字节），此处不完整列出，实际使用时参考内核文档。

### 3. struct bpf_map_info / bpf_prog_info

这些结构体用于通过 `BPF_OBJ_GET_INFO_BY_FD` 命令获取映射或程序的详细信息，包含 ID、类型、名称、大小等字段。

---

## 三、宏定义详解

### 1. BPF 映射类型（部分）

| 宏名称 | 作用 |
|--------|------|
| `BPF_MAP_TYPE_UNSPEC` | 未指定 |
| `BPF_MAP_TYPE_HASH` | 哈希表 |
| `BPF_MAP_TYPE_ARRAY` | 数组 |
| `BPF_MAP_TYPE_PROG_ARRAY` | 程序数组 |
| `BPF_MAP_TYPE_PERF_EVENT_ARRAY` | 性能事件数组 |
| `BPF_MAP_TYPE_PERCPU_HASH` | 每 CPU 哈希表 |
| `BPF_MAP_TYPE_PERCPU_ARRAY` | 每 CPU 数组 |
| `BPF_MAP_TYPE_STACK_TRACE` | 栈跟踪 |
| `BPF_MAP_TYPE_CGROUP_ARRAY` | cgroup 数组 |
| `BPF_MAP_TYPE_LRU_HASH` | LRU 哈希表 |
| `BPF_MAP_TYPE_LRU_PERCPU_HASH` | LRU 每 CPU 哈希表 |

### 2. BPF 程序类型（部分）

| 宏名称 | 作用 |
|--------|------|
| `BPF_PROG_TYPE_UNSPEC` | 未指定 |
| `BPF_PROG_TYPE_SOCKET_FILTER` | 套接字过滤器 |
| `BPF_PROG_TYPE_KPROBE` | kprobe 跟踪 |
| `BPF_PROG_TYPE_SCHED_CLS` | 流量分类器 |
| `BPF_PROG_TYPE_SCHED_ACT` | 流量动作 |
| `BPF_PROG_TYPE_TRACEPOINT` | 跟踪点 |
| `BPF_PROG_TYPE_XDP` | 快速数据路径 |
| `BPF_PROG_TYPE_PERF_EVENT` | 性能事件 |
| `BPF_PROG_TYPE_CGROUP_SKB` | cgroup 套接字过滤 |
| `BPF_PROG_TYPE_CGROUP_SOCK` | cgroup 套接字 |
| `BPF_PROG_TYPE_LWT_IN` | 轻量隧道入站 |
| `BPF_PROG_TYPE_LWT_OUT` | 轻量隧道出站 |
| `BPF_PROG_TYPE_LWT_XMIT` | 轻量隧道转发 |
| `BPF_PROG_TYPE_SOCK_OPS` | 套接字操作 |
| `BPF_PROG_TYPE_SK_SKB` | 套接字 SKB |
| `BPF_PROG_TYPE_CGROUP_DEVICE` | cgroup 设备 |
| `BPF_PROG_TYPE_SK_MSG` | 套接字消息 |
| `BPF_PROG_TYPE_RAW_TRACEPOINT` | 原始跟踪点 |
| `BPF_PROG_TYPE_CGROUP_SOCK_ADDR` | cgroup 套接字地址 |
| `BPF_PROG_TYPE_LWT_SEG6LOCAL` | 本地段路由 |
| `BPF_PROG_TYPE_LIRC_MODE2` | LIRC 模式2 |
| `BPF_PROG_TYPE_SK_REUSEPORT` | 复用端口选择 |
| `BPF_PROG_TYPE_FLOW_DISSECTOR` | 流解析器 |
| `BPF_PROG_TYPE_CGROUP_SYSCTL` | cgroup sysctl |
| `BPF_PROG_TYPE_RAW_TRACEPOINT_WRITABLE` | 可写原始跟踪点 |
| `BPF_PROG_TYPE_CGROUP_SOCKOPT` | cgroup 套接字选项 |
| `BPF_PROG_TYPE_TRACING` | 跟踪（fentry/fexit） |
| `BPF_PROG_TYPE_STRUCT_OPS` | 结构体操作 |
| `BPF_PROG_TYPE_EXT` | 扩展程序 |
| `BPF_PROG_TYPE_LSM` | Linux 安全模块 |
| `BPF_PROG_TYPE_SK_LOOKUP` | 套接字查找 |

### 3. 辅助函数编号（部分）

| 宏名称 | 作用 |
|--------|------|
| `BPF_FUNC_map_lookup_elem` | 查找映射元素 |
| `BPF_FUNC_map_update_elem` | 更新映射元素 |
| `BPF_FUNC_map_delete_elem` | 删除映射元素 |
| `BPF_FUNC_probe_read` | 安全读取内核内存 |
| `BPF_FUNC_ktime_get_ns` | 获取纳秒时间戳 |
| `BPF_FUNC_trace_printk` | 打印调试信息 |
| `BPF_FUNC_get_smp_processor_id` | 获取当前 CPU ID |
| `BPF_FUNC_skb_store_bytes` | 存储 SKB 字节 |
| `BPF_FUNC_skb_load_bytes` | 加载 SKB 字节 |
| `BPF_FUNC_get_current_pid_tgid` | 获取进程 PID/TGID |
| `BPF_FUNC_get_current_uid_gid` | 获取 UID/GID |
| `BPF_FUNC_perf_event_output` | 输出到 perf 事件 |

### 4. 其他常量

- `BPF_OBJ_NAME_LEN`：对象名称最大长度（16 字节）
- `BPF_TAG_SIZE`：程序标签大小（8 字节）
- `BPF_F_RDONLY` / `BPF_F_WRONLY`：映射权限标志
- `BPF_F_MMAPABLE`：映射支持 mmap

---

## 四、类型定义

- `bpf_insn`：BPF 指令结构体
- `bpf_attr`：系统调用参数联合体
- `bpf_cmd`：命令枚举（通常使用宏定义）

---

## 五、模板声明

`<linux/bpf.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
