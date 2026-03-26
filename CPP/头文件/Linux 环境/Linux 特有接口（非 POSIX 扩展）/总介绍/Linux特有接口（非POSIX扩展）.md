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

## 总括说明

- **定位**：Linux 特有接口充分利用了 Linux 内核的先进特性，提供比 POSIX 标准更高性能、更灵活的系统编程能力。它们是构建现代高性能服务器、容器技术及系统工具的基石。
- **现代 C++ 替代方案**：  
  - 对于网络编程，**Boost.Asio / 独立 Asio** 内部会根据平台自动选择最优后端（如 epoll），用户无需直接操作 Linux 特有接口。  
  - 对于文件系统事件监控，可使用 `std::filesystem` 轮询（效率低），或封装 inotify 的第三方库（如 `cpp-filesystem`）。  
  - 容器隔离技术通常由底层运行时（如 `runc`）处理，上层应用一般通过 SDK（如 Docker API）间接使用。
- **跨平台性**：这些接口均为 Linux 独有，不具备可移植性。在编写跨平台代码时，应使用预处理器宏 `#ifdef __linux__` 隔离，并准备备选方案（如使用 `select`/`poll` 或第三方库的抽象层）。
- **使用建议**：在开发高性能 Linux 服务时，直接使用这些接口可达到极致性能；但在一般业务开发中，推荐使用更高层次的封装（如 Asio、libevent）以降低复杂度。对于容器相关功能，优先使用成熟的开源库（如 `libcontainer`）或标准工具。