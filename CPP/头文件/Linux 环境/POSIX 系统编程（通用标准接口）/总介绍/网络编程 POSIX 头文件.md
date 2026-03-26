# POSIX 网络编程常用头文件

以下为 POSIX 标准定义的网络编程相关头文件（不含 Linux 特有扩展如 `<sys/epoll.h>`），在类 Unix 系统上通用。

---
## 核心套接字
**<sys/socket.h>**  
提供 socket()、bind()、listen()、accept()、connect()、send()、recv() 等基础套接字操作，以及 struct sockaddr、struct msghdr 等核心结构。  
典型场景：创建和管理网络通信端点，执行基本的 TCP/UDP 操作。  
现代 C++ 对应：Boost.Asio 或独立 Asio 库中的 `ip::tcp::socket`、`ip::udp::socket` 等封装。  
头文件依赖：通常独立，但建议在显式使用类型如 `pid_t` 时包含 <sys/types.h>。
**<unistd.h>**  
提供 close()、read()、write()、pipe() 等 POSIX 基础系统调用，常用于对套接字描述符进行读写和关闭。  
典型场景：直接对 socket 文件描述符进行 I/O 操作或关闭连接。  
现代 C++ 对应：Asio 中通过 `async_read`、`async_write` 等高级接口管理 I/O，无需直接调用 read/write。  
头文件依赖：通常独立。

**<fcntl.h>**  
提供 fcntl() 函数，用于设置文件描述符属性，如将套接字设置为非阻塞模式（O_NONBLOCK）或获取/设置文件状态标志。  
典型场景：控制套接字的非阻塞 I/O 或获取描述符标志。  
现代 C++ 对应：Asio 中通过 `socket::non_blocking()` 或 `io_context` 配合异步操作自动管理非阻塞行为。  
头文件依赖：通常独立。

---

## 地址与字节序

**<netinet/in.h>**  
定义 IPv4 地址结构 struct sockaddr_in、IPv6 地址结构 struct sockaddr_in6，以及协议族常量（AF_INET、AF_INET6）和字节序转换函数（htons、ntohl 等）。  
典型场景：填充或解析 IP 地址和端口号，转换网络与主机字节序。  
现代 C++ 对应：Asio 中的 `ip::address_v4`、`ip::address_v6` 和 `ip::tcp::endpoint` 等。  
头文件依赖：通常独立。

---

## 地址转换与主机名解析

**<arpa/inet.h>**  
提供 inet_pton()、inet_ntop() 等函数，用于点分十进制字符串与二进制地址的互转。  
典型场景：将用户输入的 IP 地址字符串转换为网络字节序的二进制格式，或反向输出。  
现代 C++ 对应：Asio 中 `ip::address::from_string()`、`ip::address::to_string()`。  
头文件依赖：通常需要 <netinet/in.h> 作为隐式依赖。

**<netdb.h>**  
提供 getaddrinfo()、getnameinfo()、gethostbyname()（已过时）等函数及 struct addrinfo 结构，实现主机名与服务名的解析。  
典型场景：域名解析、服务名到端口号的转换，获取地址信息。  
现代 C++ 对应：Asio 中的 `ip::tcp::resolver` 提供同步/异步解析接口。  
头文件依赖：通常依赖 <sys/socket.h> 和 <netinet/in.h> 中的类型。

---

## I/O 多路复用

**<sys/select.h>**  
提供 select() 函数以及 fd_set 操作宏（FD_ZERO、FD_SET、FD_CLR、FD_ISSET）。  
典型场景：在少量文件描述符上实现同步 I/O 多路复用，监控可读、可写或异常事件。  
现代 C++ 对应：Asio 内部使用更高效的机制（如 epoll、kqueue），但高层接口统一为 `io_context` + 回调/协程。  
头文件依赖：通常独立，但某些系统可能需要 <sys/types.h>。

**<poll.h>**  
提供 poll() 函数及 struct pollfd。  
典型场景：相比 select() 支持更多描述符且无 FD_SETSIZE 限制，适用于中等规模的 I/O 多路复用。  
现代 C++ 对应：同样被 Asio 作为可选的底层后端之一，用户无需直接使用。  
头文件依赖：通常独立。

---

## 其他辅助

**<sys/types.h>**  
定义多种 POSIX 数据类型，如 pid_t、size_t、ssize_t、socklen_t 等。  
典型场景：在使用某些系统调用（如 read、write）或套接字 API 需要特定类型时提供类型定义。  
现代 C++ 对应：通常由其他头文件间接包含，现代 C++ 编程中可直接使用 `std::size_t` 等，但在 POSIX 编程中仍常见。  
头文件依赖：通常被其他网络头文件隐式包含。

**<signal.h>**  
提供信号处理函数（signal、sigaction）及信号常量（SIGPIPE、SIGIO 等）。  
典型场景：处理网络编程中的信号，如忽略 SIGPIPE 防止进程因对端关闭而退出，或使用 SIGIO 实现异步 I/O 通知。  
现代 C++ 对应：现代网络库通常不依赖信号机制，而是通过异步操作避免信号干扰；若需处理信号，可用 C++ 标准库 `<csignal>` 或 Boost.Signals2。  
头文件依赖：通常独立。

---

## 总括说明

- **定位**：以上头文件构成 POSIX 网络编程的底层 API 基础，直接基于系统调用，提供最细粒度的控制。它们在所有符合 POSIX 标准的系统（Linux、macOS、BSD、Solaris 等）上可用，保证了代码的可移植性。
- **现代 C++ 替代方案**：  
  - **Boost.Asio / 独立 Asio** 提供了跨平台、异步、类型安全的网络编程模型，内部自动根据平台选择最优多路复用机制（如 Linux 的 epoll、macOS 的 kqueue），并支持 C++20 协程，大幅减少样板代码和错误风险。  
  - **C++23 标准库网络库**（基于 Asio 的提案，尚未最终纳入）未来将提供标准化的网络组件。
- **与 Linux 特有的区别**：Linux 下高性能服务器常使用 `<sys/epoll.h>`，它并非 POSIX 标准，但 Asio 等库会在可用时自动启用，作为对 select/poll 的替代，用户无需直接操作 epoll 即可获得高并发性能。

使用这些 POSIX 头文件可以编写可移植的网络程序，但需注意手动管理错误、非阻塞模式、资源生命周期等细节。现代 C++ 开发建议优先考虑 Asio 等库，以获得更高层次的抽象和更安全的编程体验。