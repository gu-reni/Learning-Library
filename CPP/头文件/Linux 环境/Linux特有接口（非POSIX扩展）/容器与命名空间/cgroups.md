`cgroups` 是 Linux 内核的一个功能，它本身没有提供传统的 C 头文件，也不对应一个固定的系统调用。它的接口是一个名为 `cgroupfs` 的伪文件系统（pseudo-filesystem）[reference:0]，通常挂载在 `/sys/fs/cgroup/` 目录下[reference:1][reference:2]。用户和程序通过读写这个目录下的文件来创建、删除和配置 cgroup，这与 sysfs 或 procfs 类似。

---

## 一、核心概念

要理解 cgroups 的运作，需要先掌握几个核心概念，它们之间的关系共同构成了资源控制的逻辑。

### 1. 控制组 (Control Group)

一个 cgroup 就是一组按照某种规则划分的进程的集合[reference:3]。我们可以把这个集合想象成一个“容器”或“文件夹”，所有放入其中的进程都会受到为该组设定的资源限制[reference:4]。一个进程可以属于多个 cgroup，前提是这些 cgroup 位于不同的层级中[reference:5]。

### 2. 子系统/控制器 (Subsystem/Controller)

一个子系统（也称为资源控制器）是一个内核组件，它代表一种可以被限制、审计或隔离的资源类型[reference:6][reference:7]。例如，`cpu` 子系统用于控制进程的 CPU 使用，`memory` 子系统用于限制内存使用量。一个子系统只能被附加到一个层级上[reference:8]。

### 3. 层级树 (Hierarchy)

cgroup 以树状结构进行组织[reference:9][reference:10]。一个层级树就是一棵由 cgroup 组成的家族树。所有进程在创建时，都会自动成为其父进程所在 cgroup 的成员[reference:11]。在树中，子 cgroup 会**继承**父 cgroup 的属性，也就是说，在父节点上设置的资源限制，其所有子孙节点都会受到同样的限制，无法突破[reference:12][reference:13]。

### 4. cgroupfs 文件系统

`cgroupfs` 是一个虚拟文件系统，它将 cgroup 的内核对象以文件和目录的形式暴露给用户空间[reference:14]。通过在这个文件系统中创建目录来新建 cgroup，删除目录来移除 cgroup，并通过读写目录中的特定文件来配置该 cgroup 的资源限制。

---

## 二、cgroups 主要功能

cgroups 为 Linux 提供了多种资源管理能力，主要体现在四个方面[reference:15][reference:16][reference:17]。

| 功能 | 描述 | 示例 |
| :--- | :--- | :--- |
| **资源限制** | 为一个进程组设定其可使用的资源总量上限，防止单个进程耗尽系统资源[reference:18]。 | 使用 `memory` 子系统限制某个 cgroup 的内存使用量最多为 1GB[reference:19]。 |
| **优先级控制** | 为不同的进程组分配不同的资源使用优先级，确保关键任务获得更多资源[reference:20]。 | 使用 `cpu` 子系统的 `shares` 文件，为不同 cgroup 分配不同的 CPU 时间权重[reference:21]。 |
| **资源审计** | 记录和统计一个进程组实际使用了多少资源[reference:22]。 | 使用 `cpuacct` 子系统查看某个 cgroup 中所有进程累积使用的 CPU 时间[reference:23]。 |
| **进程控制** | 对进程组执行特定的控制操作，例如挂起或恢复进程[reference:24]。 | 使用 `freezer` 子系统挂起一个 cgroup 中的所有进程，以便进行维护或快照[reference:25]。 |

---

## 三、常用控制器

Linux 内核提供了多种控制器来实现上述功能[reference:26]。下面是一些最常用的。

| 控制器 | 主要功能 | 版本支持 |
| :--- | :--- | :--- |
| **cpu** | 限制 cgroup 的 CPU 使用。例如，通过 `cpu.shares` 设置相对权重，或通过 `cpu.cfs_period_us` 和 `cpu.cfs_quota_us` 设置绝对上限[reference:27]。 | v1, v2 |
| **cpuacct** | 自动生成 cgroup 中任务所使用的 CPU 资源的统计报告，如 `cpuacct.usage` 记录 CPU 使用时间[reference:28]。 | v1 |
| **cpuset** | 将 cgroup 中的任务绑定到指定的 CPU 核心和内存节点上运行[reference:29]。 | v1, v2 |
| **memory** | 限制 cgroup 的内存使用量，并可生成内存使用报告。可以设置物理内存+交换分区的硬限制和软限制[reference:30]。 | v1, v2 |
| **blkio / io** | 限制 cgroup 对块设备（如磁盘）的 I/O 访问。在 v1 中为 `blkio`，v2 中统一为 `io`，可设置 I/O 的带宽上限和 IOPS 上限[reference:31]。 | v1, v2 |
| **pids** | 限制一个 cgroup 中能够创建的进程总数，可有效防范“fork 炸弹”攻击[reference:32]。 | v1, v2 |
| **devices** | 控制 cgroup 中的任务可以访问哪些设备文件[reference:33]。 | v1 |
| **freezer** | 挂起或恢复 cgroup 中的所有任务[reference:34]。 | v1, v2 |
| **net_cls** | 使用等级识别符 (classid) 标记 cgroup 中进程发出的网络数据包，以便 `tc` (Traffic Control) 或 `iptables` 根据这些标记对流量进行控制[reference:35]。 | v1 |
| **net_prio** | 动态设置 cgroup 中产生的网络流量的优先级[reference:36]。 | v1 |
| **perf_event** | 将 cgroup 中的任务分组，以便通过 `perf` 性能监控工具对它们进行统一监控和分析[reference:37]。 | v1, v2 |
| **hugetlb** | 限制 cgroup 对大页内存（HugeTLB）的使用[reference:38]。 | v1, v2 |
| **rdma** | 限制 cgroup 对 RDMA（远程直接内存访问）和 InfiniBand 等特定资源的使用[reference:39]。 | v1, v2 |

---

## 四、cgroups v1 vs. v2

cgroups 主要有两个版本：v1 和 v2，它们在设计和实现上存在显著差异。

| 对比项 | cgroups v1 | cgroups v2 |
| :--- | :--- | :--- |
| **层级结构** | **多层级结构**。每个控制器可以有自己的独立层级树[reference:40]。 | **单一统一层级**。所有控制器共享一个统一的层级树[reference:41][reference:42]。 |
| **接口一致性** | 接口和行为在不同控制器间不一致，因为它们是逐步添加的[reference:43][reference:44]。 | 接口和行为在不同控制器间保持一致[reference:45][reference:46]。 |
| **控制器管理** | 通过将控制器挂载到不同目录来组织，管理方式复杂[reference:47]。 | 通过 `cgroup.controllers` 和 `cgroup.subtree_control` 文件动态管理子节点可用的控制器[reference:48]。 |
| **设计哲学** | **资源导向**。目录结构主要围绕资源类型（如 `cpu/`, `memory/`）组织[reference:49]。 | **进程导向**。目录结构围绕进程组组织，其内部文件代表不同的资源控制器[reference:50]。 |

---

## 五、管理 cgroups

用户可以通过多种方式来创建和管理 cgroups。

### 1. 直接操作 cgroupfs

这是最基础的方法，通过 Shell 命令直接操作文件系统。

**查看系统当前挂载的 cgroups 信息：**
```bash
mount -t cgroup
cat /proc/cgroups
```
执行 `mount -t cgroup` 命令可以列出当前系统挂载的 cgroup 文件系统类型及相关信息[reference:51]。`/proc/cgroups` 文件则提供了已注册的控制器的摘要信息[reference:52]。

**创建一个 cgroup 并设置限制 (v1)：**
```bash
# 在 memory 子系统下创建一个名为 mygroup 的 cgroup
sudo mkdir /sys/fs/cgroup/memory/mygroup

# 设置内存使用上限为 100MB
echo "100000000" | sudo tee /sys/fs/cgroup/memory/mygroup/memory.limit_in_bytes

# 将进程 1234 加入到该 cgroup 中
echo 1234 | sudo tee /sys/fs/cgroup/memory/mygroup/cgroup.procs
```

**创建一个 cgroup 并设置限制 (v2)：**
```bash
# 在默认挂载点下创建一个名为 mygroup 的 cgroup
sudo mkdir /sys/fs/cgroup/mygroup

# 为这个新的 cgroup 启用 memory 和 cpu 控制器
echo "+memory +cpu" | sudo tee /sys/fs/cgroup/mygroup/cgroup.subtree_control

# 设置内存限制
echo "100000000" | sudo tee /sys/fs/cgroup/mygroup/memory.max

# 将进程 1234 加入到该 cgroup 中
echo 1234 | sudo tee /sys/fs/cgroup/mygroup/cgroup.procs
```

### 2. 使用 `cgroup-tools` 软件包

`cgroup-tools` 提供了一组命令行工具，封装了直接操作 `cgroupfs` 的细节，简化了管理过程[reference:53]。

| 命令 | 作用 | 示例 |
| :--- | :--- | :--- |
| `cgcreate` | 在指定子系统中创建新的 cgroup[reference:54]。 | `sudo cgcreate -g cpu,memory:/mygroup` |
| `cgset` | 设置 cgroup 的参数[reference:55]。 | `sudo cgset -r memory.limit_in_bytes=100M mygroup` |
| `cgclassify` | 将正在运行的进程移动到一个 cgroup 中[reference:56]。 | `sudo cgclassify -g cpu,memory:mygroup 1234` |
| `cgexec` | 在一个指定的 cgroup 中启动一个新程序[reference:57]。 | `sudo cgexec -g cpu,memory:mygroup stress --cpu 1` |
| `cgget` | 查看 cgroup 的配置参数[reference:58]。 | `sudo cgget -g memory:mygroup` |
| `cgdelete` | 删除一个已存在的 cgroup[reference:59]。 | `sudo cgdelete -g cpu,memory:mygroup` |
| `lscgroup` | 列出系统中所有的 cgroup[reference:60]。 | `lscgroup` |

### 3. 通过 Systemd

在大多数现代 Linux 发行版中，`systemd` 是用户空间启动的第一个进程，它利用 cgroups 来组织和管理所有服务与用户会话[reference:61]。

- **管理服务**：通过 `systemctl` 命令设置的 CPUQuota, MemoryMax 等选项，`systemd` 会自动在 `system.slice` 层级下创建对应的 cgroup 并应用限制。
- **查看系统资源使用**：`systemd-cgtop` 命令可以动态查看各个 cgroup 的资源使用情况。

---

## 六、系统调用

cgroups 的核心接口是 `cgroupfs` 文件系统，其操作直接映射为对文件系统的读写。虽然 Linux 内核没有为 cgroup 提供一个专用的系统调用，但其背后的实现依赖于其他通用系统调用。

| 系统调用 | 作用 |
| :--- | :--- |
| **`mount()`** | 将 `cgroupfs` 文件系统挂载到 `/sys/fs/cgroup` 目录，以启动 cgroup 功能[reference:62]。 |
| **`mkdir()`** | 在挂载了 `cgroupfs` 的目录下创建子目录，这实际上是在创建新的 cgroup。 |
| **`rmdir()`** | 删除目录来移除一个 cgroup。 |
| **`open()` / `read()` / `write()`** | 打开并读写 cgroup 目录下的控制文件（如 `cgroup.procs`, `memory.limit_in_bytes`），以管理 cgroup 或设置资源限制。 |

---

## 七、相关文件

| 文件路径 | 描述 |
| :--- | :--- |
| `/proc/cgroups` | 提供系统当前支持的 cgroup 子系统列表及其相关信息（如层级、启用状态）[reference:63]。 |
| `/proc/[pid]/cgroup` | 显示指定进程（PID）所属的 cgroup[reference:64]。 |

---
