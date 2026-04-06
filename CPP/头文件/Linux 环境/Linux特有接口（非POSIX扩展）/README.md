# Linux 特有接口总介绍

以下为 Linux 操作系统特有的、不属于 POSIX 标准的系统调用和接口，它们提供了高性能 I/O 多路复用、增强型文件描述符事件通知、容器资源隔离等高级功能，是构建高并发服务器、容器运行时及系统级工具的基础。

---

## epoll 多路复用

**<sys/epoll.h>**  
提供 epoll 事件驱动接口：`epoll_create()`、`epoll_ctl()`、`epoll_wait()`。支持水平触发（LT）和边缘触发（ET），可高效管理数十万级文件描述符，是 Linux 下替代 `select`/`poll` 的现代方案。  
典型场景：高并发网络服务器（如 Nginx、Redis）、事件驱动框架。  
现代 C++ 对应：Boost.Asio 或独立 Asio 库在 Linux 平台内部使用 epoll 作为后端，用户无需直接操作 epoll；C++20 协程配合 `io_context` 可间接使用。  
头文件依赖：通常独立，需链接 `-lpthread`（若使用多线程）。

---

## signalfd / eventfd / timerfd

**<sys/signalfd.h>**  
提供 `signalfd()` 函数，将信号转换为文件描述符事件，允许在 epoll 中统一处理信号，避免传统信号处理函数的异步中断问题。  
典型场景：需要可靠处理信号（如 `SIGINT`、`SIGTERM`）的事件驱动程序。  
现代 C++ 对应：C++ 标准库中的 `std::signal` 仍存在，但 signalfd 与事件循环集成更自然。  
头文件依赖：通常独立。

**<sys/eventfd.h>**  
提供 `eventfd()` 函数，创建用于事件通知的文件描述符，支持线程/进程间轻量级计数信号。  
典型场景：事件循环中唤醒 `epoll_wait`、实现简单的线程间通知。  
现代 C++ 对应：`std::promise`/`std::future` 或条件变量可部分替代，但 eventfd 更轻量且可与 epoll 集成。  
头文件依赖：通常独立。

**<sys/timerfd.h>**  
提供 `timerfd_create()`、`timerfd_settime()` 等函数，将定时器事件通过文件描述符暴露，可与 epoll 统一管理。  
典型场景：实现高精度定时任务、超时控制，避免信号机制。  
现代 C++ 对应：`<chrono>` 与 `<thread>` 或 `std::async` 可组合定时任务，但 timerfd 与事件循环的集成更自然。  
头文件依赖：通常独立。

---

## cgroups 与命名空间

cgroups（控制组）和命名空间（Namespaces）是 Linux 内核实现容器隔离和资源限制的核心技术，主要通过操作 `/sys/fs/cgroup` 下的文件系统接口以及特定系统调用实现。

**<sched.h>**（需定义 `_GNU_SOURCE`）  
提供 `clone()` 系统调用的扩展标志，如 `CLONE_NEWNS`、`CLONE_NEWUTS`、`CLONE_NEWNET` 等，用于创建新的命名空间。  
典型场景：容器运行时（如 Docker、Podman）实现进程、网络、文件系统等隔离。  
现代 C++ 对应：无直接标准库替代，通常使用 `libc` 提供的封装或直接系统调用。  
头文件依赖：需定义 `_GNU_SOURCE` 以获取扩展标志。

**<sys/mount.h>**  
提供 `mount()` 和 `umount()` 系统调用，用于挂载和卸载文件系统，结合命名空间可实现容器根文件系统隔离。  
典型场景：创建独立文件系统视图、实现 `chroot` 类似功能。  
头文件依赖：通常独立。

**cgroups 接口**  
无特定头文件，通过文件系统 API（`open`、`read`、`write`）操作 `/sys/fs/cgroup` 下的文件来配置资源限制（CPU、内存、I/O 等）。  
典型场景：限制容器或进程组的资源使用。  
现代 C++ 对应：可借助 `<filesystem>` 和标准文件流操作，但无高级封装，通常仍需直接操作文件。

---

## 其他系统调用扩展

**<sys/inotify.h>**  
提供 inotify 机制：`inotify_init()`、`inotify_add_watch()`、`read()` 等，用于监控文件系统事件（创建、删除、修改等）。  
典型场景：文件同步工具、自动构建系统、安全监控。  
现代 C++ 对应：无标准库替代，可使用 `std::filesystem` 轮询，但 inotify 更高效。

**<linux/perf_event.h>**  
提供性能事件接口，用于硬件性能计数器、软件事件（如 CPU 周期、指令数）的采样与统计。  
典型场景：性能剖析工具、调优分析。  
头文件依赖：需定义 `_GNU_SOURCE`，通常需链接 `-lperf`。

**<linux/fs.h>**  
包含文件系统相关的高级接口和常量，如 `ioctl` 操作、文件系统挂载标志等。  
典型场景：实现低级文件系统操作（如 `ioctl` 调用）。  
头文件依赖：通常与 `<sys/ioctl.h>` 配合使用。

---

## io_uring 异步 I/O

**<linux/io_uring.h>**（需安装 `liburing` 开发库）  
提供 `io_uring_setup()`、`io_uring_enter()`、`io_uring_register()` 系统调用，实现高性能异步 I/O。支持批量提交、轮询完成，显著减少系统调用次数，是 epoll 的高效替代方案。  
典型场景：高吞吐磁盘 I/O、网络服务器、零拷贝数据传输。  
现代 C++ 对应：尚无标准库封装，可使用 `liburing` 的 C++ 绑定或自研封装。  
头文件依赖：需定义 `_GNU_SOURCE`，链接 `-luring`。

---

## futex 快速用户态互斥锁

**<linux/futex.h>**  
提供 `futex()` 系统调用，实现用户态下的快速锁同步。`pthread_mutex` 等同步原语底层依赖 futex，当无竞争时完全在用户态自旋，避免陷入内核。  
典型场景：实现低开销的用户态锁、条件变量、信号量。  
现代 C++ 对应：`std::mutex`、`std::atomic` 等底层可能使用 futex，但无需直接操作。  
头文件依赖：需定义 `_GNU_SOURCE`。

---

## seccomp 安全计算模式

**<linux/seccomp.h>**  
提供 `seccomp()` 系统调用，限制进程可使用的系统调用集合，用于构建安全沙箱（如 Docker、Chrome 沙箱）。  
典型场景：容器安全隔离、减少内核攻击面。  
现代 C++ 对应：无标准库封装，通常使用 `libseccomp` 库。  
头文件依赖：需定义 `_GNU_SOURCE`，链接 `-lseccomp`。

---

## BPF / eBPF 内核虚拟机

**<linux/bpf.h>**  
提供 `bpf()` 系统调用，允许用户态程序加载字节码到内核中执行，用于网络包过滤（如 `tcpdump`）、性能监控、安全策略等。  
典型场景：高性能网络处理（XDP）、系统追踪（BCC）、容器安全。  
现代 C++ 对应：可使用 `libbpf` 或 `bpftrace` 等工具，无标准库封装。  
头文件依赖：需定义 `_GNU_SOURCE`，链接 `-lbpf`。

---

## userfaultfd 用户态缺页处理

**<linux/userfaultfd.h>**  
提供 `userfaultfd()` 系统调用，创建文件描述符用于接收缺页异常，允许用户态程序按需填充内存（如 QEMU 的 post-copy 迁移）。  
典型场景：用户态内存管理、虚拟机热迁移、大页内存管理。  
现代 C++ 对应：无标准库封装，需直接使用系统调用。  
头文件依赖：需定义 `_GNU_SOURCE`。

---

## memfd_create 匿名内存文件

**<sys/memfd.h>**  
提供 `memfd_create()` 系统调用，创建匿名内存文件（不关联磁盘），用于共享内存（如 Wayland 的共享缓冲区）、临时文件。  
典型场景：进程间共享内存、密封内存（`memfd_secret`）。  
现代 C++ 对应：C++17 无直接对应，可借助 `std::shared_memory`（C++26 可能引入）。  
头文件依赖：通常独立，需定义 `_GNU_SOURCE`。

---

## pidfd 进程文件描述符

**<sys/pidfd.h>**  
提供 `pidfd_open()`、`pidfd_send_signal()` 等函数，将进程 ID 包装为文件描述符，避免 PID 重用导致的竞争（如等待特定进程退出）。  
典型场景：可靠地等待子进程、向进程发送信号而不受 PID 重用影响。  
现代 C++ 对应：C++ 标准库无对应，可借助 `std::process`（C++26 可能引入）。  
头文件依赖：需定义 `_GNU_SOURCE`。

---

## openat2 更安全的路径打开

**<linux/openat2.h>**  
提供 `openat2()` 系统调用，扩展 `openat` 功能，支持额外的 `resolve` 标志（如禁止路径解析跟随符号链接、禁止越过挂载点）。  
典型场景：安全文件打开、沙箱路径解析。  
现代 C++ 对应：C++17 的 `std::filesystem` 无法实现同等安全级别，需直接调用。  
头文件依赖：需定义 `_GNU_SOURCE`。

---

## fanotify 文件系统事件监控

**<sys/fanotify.h>**  
提供 `fanotify_init()`、`fanotify_mark()` 函数，监控文件系统事件（比 inotify 更强大，支持整个挂载点、文件访问控制）。  
典型场景：防病毒软件、文件完整性监控、用户行为审计。  
现代 C++ 对应：无标准库封装，需直接调用系统接口。  
头文件依赖：通常独立。

---

## 总括说明

- **定位**：Linux 特有接口充分利用了 Linux 内核的先进特性，提供比 POSIX 标准更高性能、更灵活的系统编程能力。它们是构建现代高性能服务器、容器技术及系统工具的基石。
- **现代 C++ 替代方案**：  
  - 对于网络编程，**Boost.Asio / 独立 Asio** 内部会根据平台自动选择最优后端（如 epoll），用户无需直接操作 Linux 特有接口。  
  - 对于文件系统事件监控，可使用 `std::filesystem` 轮询（效率低），或封装 inotify/fanotify 的第三方库。  
  - 对于异步 I/O，**liburing** 提供了 C 风格的封装，未来可能有 C++ 标准库支持。  
  - 容器隔离技术通常由底层运行时（如 `runc`）处理，上层应用一般通过 SDK（如 Docker API）间接使用。
- **跨平台性**：这些接口均为 Linux 独有，不具备可移植性。在编写跨平台代码时，应使用预处理器宏 `#ifdef __linux__` 隔离，并准备备选方案（如使用 `select`/`poll` 或第三方库的抽象层）。
- **使用建议**：在开发高性能 Linux 服务时，直接使用这些接口可达到极致性能；但在一般业务开发中，推荐使用更高层次的封装（如 Asio、liburing）以降低复杂度。对于容器相关功能，优先使用成熟的开源库（如 `libcontainer`）或标准工具。