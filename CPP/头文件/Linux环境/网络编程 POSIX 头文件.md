
|头文件|主要功能|备注|
|---|---|---|
|`<sys/socket.h>`|核心套接字 API：`socket()`, `bind()`, `listen()`, `accept()`, `connect()`, `send()`, `recv()` 等|POSIX 标准，Linux 网络编程基础|
|`<netinet/in.h>`|Internet 地址族结构体（如 `sockaddr_in`）和字节序转换函数（`htons`, `htonl` 等）|与 `<sys/socket.h>` 配合使用|
|`<arpa/inet.h>`|IP 地址转换函数：`inet_pton()`, `inet_ntop()`|推荐使用，替代老旧的 `inet_addr`|
|`<unistd.h>`|系统调用：`close()`, `read()`, `write()` 等|用于关闭套接字和基本 I/O|
|`<sys/types.h>`|基本系统数据类型（`size_t`, `ssize_t` 等）|常与其他头文件隐含包含|
|`<netdb.h>`|域名解析：`gethostbyname()`, `getaddrinfo()` 等|`getaddrinfo` 是推荐使用的现代 API|
|`<fcntl.h>`|文件控制：`fcntl()` 函数|用于设置套接字非阻塞模式|
|`<poll.h>`|I/O 多路复用：`poll()`|跨平台较好的多路复用方案|
|`<sys/epoll.h>`|Linux 特有高性能 I/O 多路复用：`epoll` 系列函数|Linux 独有，高并发服务器常用|
|`<pthread.h>`|多线程编程|用于多线程网络服务器|
|`<signal.h>`|信号处理|常用于忽略 `SIGPIPE` 等信号|