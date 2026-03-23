
**核心套接字头文件**

- **`<sys/socket.h>`**  
  提供 `socket()`、`bind()`、`listen()`、`accept()`、`connect()`、`send()`、`recv()` 等核心套接字函数。
[[详解-1]]
- **`<netinet/in.h>`**  
  定义 `sockaddr_in` 结构体（IPv4 地址与端口）以及字节序转换函数 `htons()`、`htonl()` 等。

- **`<arpa/inet.h>`**  
  包含 IP 地址转换函数 `inet_pton()` 和 `inet_ntop()`，用于点分十进制与二进制 IP 地址互转。

- **`<unistd.h>`**  
  提供 `close()`、`read()`、`write()` 等系统调用，用于关闭套接字及基本 I/O 操作。

**辅助头文件**

- **`<sys/types.h>`**  
  定义基本系统数据类型，如 `size_t`、`ssize_t`，通常与其他头文件隐式包含。

- **`<netdb.h>`**  
  提供域名解析函数 `getaddrinfo()`、`gethostbyname()` 等。

- **`<fcntl.h>`**  
  用于通过 `fcntl()` 将套接字设置为非阻塞模式。

- **`<poll.h>`** 与 **`<sys/epoll.h>`**  
  分别提供 `poll()`（跨平台）和 Linux 特有的 `epoll` 系列函数，用于 I/O 多路复用。

- **`<pthread.h>`**  
  多线程编程头文件，用于实现多线程网络服务器。

- **`<signal.h>`**  
  信号处理，常用于忽略 `SIGPIPE` 防止进程异常退出。

---

**说明**  
以上头文件均属于 POSIX 标准，在 Linux 下可以直接用于 C++ 程序。它们并非 C++ 独有，而是 Linux 网络编程的基础接口。若希望使用现代 C++ 风格且跨平台的网络库，可考虑 **Boost.Asio** 或独立 **Asio**。