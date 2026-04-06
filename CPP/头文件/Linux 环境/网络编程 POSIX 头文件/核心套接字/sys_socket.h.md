## `<sys/socket.h>` 头文件详解（Linux / POSIX）

`<sys/socket.h>` 是 POSIX 套接字编程的核心头文件，提供了创建通信端点（套接字）的函数、通用地址结构以及各种常量和宏。它是网络编程的基础，支持 TCP、UDP、Unix 域套接字等通信协议。

---

## 一、函数详解

### 1. socket 函数

**函数原型：**
```c
int socket(int domain, int type, int protocol);
```

**作用：** 创建一个通信端点（套接字），返回一个文件描述符，后续操作（绑定、监听、收发数据）都通过该描述符进行。

**参数：**
- `domain`：协议族（地址族）。常用值：`AF_INET`（IPv4）、`AF_INET6`（IPv6）、`AF_UNIX`（Unix 域）、`AF_PACKET`（链路层）。
- `type`：套接字类型。常用值：`SOCK_STREAM`（流式，TCP）、`SOCK_DGRAM`（数据报，UDP）、`SOCK_RAW`（原始套接字）。
- `protocol`：具体协议。通常设为 0，让系统根据前两个参数自动选择（如 `SOCK_STREAM` + `AF_INET` 自动选 TCP）。

**返回值：**
- 成功：返回非负文件描述符。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd == -1) { perror("socket"); exit(1); }
```

**实现原理：**
系统调用。内核根据 `domain`、`type`、`protocol` 查找已注册的协议族，分配 `struct socket` 内核对象，初始化协议栈操作表，分配文件描述符并关联。

**线程安全提示：**
线程安全。多个线程同时调用 `socket` 不会相互干扰。

---

### 2. bind 函数

**函数原型：**
```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

**作用：** 将套接字与本地协议地址（IP + 端口）绑定，通常用于服务器端。

**参数：**
- `sockfd`：`socket()` 返回的描述符。
- `addr`：指向协议地址结构的指针（如 `struct sockaddr_in`）。
- `addrlen`：地址结构的长度。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1，常见 `errno`：`EADDRINUSE`（地址已被占用）。

**示例用法：**
```c
struct sockaddr_in addr;
memset(&addr, 0, sizeof(addr));
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = INADDR_ANY;
addr.sin_port = htons(8080);
if (bind(sockfd, (struct sockaddr*)&addr, sizeof(addr)) == -1) { perror("bind"); exit(1); }
```

**实现原理：**
系统调用。内核将地址信息保存到套接字对象，检查端口是否被占用（除非设置了 `SO_REUSEADDR`）。

**线程安全提示：**
线程安全。多个线程对同一未绑定套接字并发调用可能导致竞争，但函数本身不会破坏数据结构。

---

### 3. listen 函数

**函数原型：**
```c
int listen(int sockfd, int backlog);
```

**作用：** 将套接字转为被动监听状态，使其能够接收客户端连接请求。

**参数：**
- `sockfd`：已绑定的套接字。
- `backlog`：已完成连接队列的最大长度（全连接队列大小）。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
if (listen(sockfd, SOMAXCONN) == -1) { perror("listen"); exit(1); }
```

**实现原理：**
内核将套接字状态转为 `LISTEN`，创建半连接队列（SYN 队列）和全连接队列（Accept 队列），开始接收 SYN 报文。

**线程安全提示：**
线程安全。多线程对同一套接字调用 `listen` 只有第一个成功。

---

### 4. accept 函数

**函数原型：**
```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

**作用：** 从全连接队列中取出第一个已完成连接的客户端，返回一个新的套接字用于通信。

**参数：**
- `sockfd`：监听套接字。
- `addr`：输出参数，返回客户端地址（可为 NULL）。
- `addrlen`：输入输出参数，传入地址结构长度，返回实际长度。

**返回值：**
- 成功：返回新的文件描述符（已连接套接字）。
- 失败：返回 -1，常见 `errno`：`EAGAIN`（非阻塞模式下无连接）、`EINTR`（被信号中断）。

**示例用法：**
```c
struct sockaddr_in client_addr;
socklen_t len = sizeof(client_addr);
int client_fd = accept(sockfd, (struct sockaddr*)&client_addr, &len);
if (client_fd == -1) { perror("accept"); exit(1); }
```

**实现原理：**
系统调用。若全连接队列为空则阻塞（默认），取出连接后创建新套接字对象，分配新文件描述符。

**线程安全提示：**
线程安全。多线程可并发 `accept` 同一监听套接字，内核互斥保护队列。

---

### 5. connect 函数

**函数原型：**
```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```

**作用：** 客户端主动与服务器建立连接（TCP）或设定对端地址（UDP）。

**参数：**
- `sockfd`：未连接的套接字。
- `addr`：服务器的协议地址。
- `addrlen`：地址长度。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1，常见 `errno`：`ECONNREFUSED`（连接被拒绝）、`ETIMEDOUT`（超时）、`EINPROGRESS`（非阻塞）。

**示例用法：**
```c
struct sockaddr_in server;
server.sin_family = AF_INET;
server.sin_port = htons(8080);
inet_pton(AF_INET, "127.0.0.1", &server.sin_addr);
if (connect(sockfd, (struct sockaddr*)&server, sizeof(server)) == -1) { perror("connect"); exit(1); }
```

**实现原理：**
TCP：发送 SYN 报文，完成三次握手；UDP：仅记录对端地址，不发送报文。

**线程安全提示：**
线程安全，但同一套接字不应被多线程同时连接。

---

### 6. send 函数

**函数原型：**
```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```

**作用：** 在已连接的套接字上发送数据（通常用于 TCP）。

**参数：**
- `flags`：控制选项，常用 `0`、`MSG_DONTWAIT`（非阻塞）、`MSG_NOSIGNAL`（禁止 `SIGPIPE`）。

**返回值：**
- 成功：返回实际发送的字节数（可能小于 `len`）。
- 失败：返回 -1。

**示例用法：**
```c
char *msg = "Hello";
ssize_t sent = send(client_fd, msg, strlen(msg), 0);
if (sent == -1) perror("send");
```

**实现原理：**
数据拷贝到内核发送缓冲区，TCP 协议栈根据拥塞控制发送。

**线程安全提示：**
线程安全。多线程并发发送时，内核保证每次 `send` 的原子性，但顺序可能交叉。

---

### 7. recv 函数

**函数原型：**
```c
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```

**作用：** 在已连接的套接字上接收数据（通常用于 TCP）。

**参数：**
- `flags`：常用 `0`、`MSG_DONTWAIT`、`MSG_WAITALL`。

**返回值：**
- 成功：返回实际接收的字节数。
- 对端关闭连接：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
char buffer[1024];
ssize_t n = recv(client_fd, buffer, sizeof(buffer), 0);
if (n > 0) { buffer[n] = '\0'; printf("%s\n", buffer); }
```

**实现原理：**
从内核接收缓冲区读取数据，若为空则阻塞。

**线程安全提示：**
线程安全，但多线程读取同一套接字可能导致数据混乱，需同步。

---

### 8. sendto 函数

**函数原型：**
```c
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
```

**作用：** 用于无连接套接字（UDP）发送数据，可指定目标地址。

**示例用法：**
```c
struct sockaddr_in dest;
dest.sin_family = AF_INET;
dest.sin_port = htons(8080);
inet_pton(AF_INET, "192.168.1.100", &dest.sin_addr);
sendto(sockfd, msg, strlen(msg), 0, (struct sockaddr*)&dest, sizeof(dest));
```

**线程安全提示：**
线程安全，每次发送一个完整数据报，互不干扰。

---

### 9. recvfrom 函数

**函数原型：**
```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
```

**作用：** 用于无连接套接字（UDP）接收数据，同时返回源地址。

**线程安全提示：**
线程安全，每个数据报只被一个线程接收。

---

### 10. setsockopt / getsockopt 函数

**函数原型：**
```c
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
```

**作用：** 设置/获取套接字选项（如 `SO_REUSEADDR`、`SO_RCVTIMEO`、`TCP_NODELAY` 等）。

**线程安全提示：**
线程安全。多线程可并发设置不同选项，但修改同一选项可能竞争。

---

### 11. shutdown 函数

**函数原型：**
```c
int shutdown(int sockfd, int how);
```

**作用：** 独立关闭套接字的读或写方向。

**参数：** `how` 为 `SHUT_RD`（关闭读）、`SHUT_WR`（关闭写）、`SHUT_RDWR`（同时关闭）。

**线程安全提示：**
线程安全，但应与读写操作同步。

---

### 12. close 函数（定义在 `<unistd.h>`，但常与套接字一同使用）

**函数原型：**
```c
int close(int fd);
```

**作用：** 关闭文件描述符，当引用计数为零时释放套接字资源并终止连接。

**线程安全提示：**
线程安全，但并发关闭可能导致其他线程使用无效 fd。

---

## 二、结构体详解

### 1. struct sockaddr

**定义：**
```c
struct sockaddr {
    sa_family_t sa_family;    // 地址族
    char        sa_data[14];  // 地址数据
};
```

**作用：** 通用套接字地址结构，用于参数传递。所有具体地址结构都可转换为 `struct sockaddr *`。

---

### 2. struct sockaddr_storage

**定义：**
```c
struct sockaddr_storage {
    sa_family_t ss_family;      // 地址族
    char        __ss_padding[_SS_PADSIZE];  // 填充
};
```

**作用：** 足够大且对齐合适的通用地址结构，可容纳任意协议族的地址。

---

### 3. struct msghdr

**定义：**
```c
struct msghdr {
    void         *msg_name;       // 协议地址
    socklen_t     msg_namelen;    // 地址长度
    struct iovec *msg_iov;        // 数据缓冲区数组
    size_t        msg_iovlen;     // iov 数组长度
    void         *msg_control;    // 辅助数据缓冲区
    size_t        msg_controllen; // 辅助数据长度
    int           msg_flags;      // 接收标志
};
```

**作用：** 用于 `sendmsg`/`recvmsg`，支持分散/聚集 I/O 和辅助数据传递。

---

### 4. struct cmsghdr

**定义：**
```c
struct cmsghdr {
    size_t cmsg_len;    // 总长度
    int    cmsg_level;  // 协议层
    int    cmsg_type;   // 消息类型
    /* 后面跟数据 */
};
```

**作用：** 辅助数据头部，用于传递文件描述符等控制信息。

---

### 5. struct iovec

**定义：**
```c
struct iovec {
    void  *iov_base;   // 缓冲区地址
    size_t iov_len;    // 缓冲区长度
};
```

**作用：** 描述一个数据缓冲区，用于 scatter/gather I/O。

---

### 6. struct linger

**定义：**
```c
struct linger {
    int l_onoff;   // 是否启用
    int l_linger;  // 延迟时间（秒）
};
```

**作用：** 与 `SO_LINGER` 选项配合，控制 `close` 行为。

---

### 7. 派生地址结构（定义在其他头文件，但常与 `<sys/socket.h>` 一起使用）

- **`struct sockaddr_in`**（`<netinet/in.h>`）：IPv4 地址结构，包含 `sin_family`、`sin_port`、`sin_addr`。
- **`struct sockaddr_in6`**（`<netinet/in.h>`）：IPv6 地址结构。
- **`struct sockaddr_un`**（`<sys/un.h>`）：Unix 域套接字地址结构。
- **`struct sockaddr_ll`**（`<linux/if_packet.h>`）：链路层地址结构（Linux 特有）。
- **`struct sockaddr_nl`**（`<linux/netlink.h>`）：Netlink 套接字地址结构（Linux 特有）。

---

## 三、宏定义详解

### 1. 地址族常量（domain）

| 宏名称 | 作用 |
|--------|------|
| `AF_UNIX` / `AF_LOCAL` | Unix 域套接字（本地通信） |
| `AF_INET` | IPv4 网络协议 |
| `AF_INET6` | IPv6 网络协议 |
| `AF_PACKET` | 链路层套接字（Linux 特有） |
| `AF_NETLINK` | Netlink 套接字（Linux 特有） |
| `AF_UNSPEC` | 未指定地址族 |

---

### 2. 套接字类型常量（type）

| 宏名称 | 作用 |
|--------|------|
| `SOCK_STREAM` | 流式套接字（TCP） |
| `SOCK_DGRAM` | 数据报套接字（UDP） |
| `SOCK_RAW` | 原始套接字（直接访问 IP 层） |
| `SOCK_SEQPACKET` | 顺序数据包套接字（SCTP） |
| `SOCK_NONBLOCK` | 非阻塞标志（Linux 扩展，可与其他类型按位或） |
| `SOCK_CLOEXEC` | `FD_CLOEXEC` 标志（Linux 扩展） |

---

### 3. 协议常量（protocol，部分）

| 宏名称 | 作用 |
|--------|------|
| `IPPROTO_TCP` | TCP 协议 |
| `IPPROTO_UDP` | UDP 协议 |
| `IPPROTO_ICMP` | ICMP 协议 |
| `IPPROTO_IP` | IPv4 协议 |
| `IPPROTO_IPV6` | IPv6 协议 |
| `IPPROTO_RAW` | 原始 IP 包 |
| `IPPROTO_SCTP` | SCTP 协议 |

---

### 4. 套接字选项级别和名称（部分）

**级别（level）：**
| 宏名称 | 作用 |
|--------|------|
| `SOL_SOCKET` | 通用套接字选项 |
| `IPPROTO_TCP` | TCP 协议选项 |
| `IPPROTO_IP` | IPv4 协议选项 |
| `IPPROTO_IPV6` | IPv6 协议选项 |

**通用选项（optname，SOL_SOCKET 级别）：**
| 宏名称 | 作用 |
|--------|------|
| `SO_REUSEADDR` | 允许重用本地地址 |
| `SO_REUSEPORT` | 允许多个套接字绑定同一端口 |
| `SO_KEEPALIVE` | 启用 TCP 保活机制 |
| `SO_RCVBUF` / `SO_SNDBUF` | 获取/设置接收/发送缓冲区大小 |
| `SO_RCVTIMEO` / `SO_SNDTIMEO` | 设置接收/发送超时（`struct timeval`） |
| `SO_LINGER` | 控制关闭行为（`struct linger`） |
| `SO_BROADCAST` | 允许发送广播数据报 |
| `SO_ERROR` | 获取套接字错误状态 |
| `SO_TYPE` | 获取套接字类型 |

**TCP 选项（IPPROTO_TCP 级别）：**
| 宏名称 | 作用 |
|--------|------|
| `TCP_NODELAY` | 禁用 Nagle 算法 |
| `TCP_KEEPIDLE` | 保活空闲时间 |
| `TCP_KEEPINTVL` | 保活探测间隔 |
| `TCP_KEEPCNT` | 保活探测次数 |

---

### 5. 消息标志（flags，用于 send/recv 等）

| 宏名称 | 作用 |
|--------|------|
| `MSG_DONTWAIT` | 非阻塞操作 |
| `MSG_NOSIGNAL` | 禁止产生 `SIGPIPE` 信号 |
| `MSG_WAITALL` | 等待接收全部数据（仅 `recv`） |
| `MSG_PEEK` | 窥视数据而不从缓冲区移除 |
| `MSG_OOB` | 带外数据 |
| `MSG_TRUNC` | 返回数据报被截断的标志（仅 `recv`） |
| `MSG_CTRUNC` | 返回辅助数据被截断的标志（仅 `recv`） |

---

### 6. shutdown 常量

| 宏名称 | 作用 |
|--------|------|
| `SHUT_RD` | 关闭读方向 |
| `SHUT_WR` | 关闭写方向 |
| `SHUT_RDWR` | 同时关闭读写 |

---

### 7. 其他常用宏

- `SOMAXCONN`：系统允许的最大 `backlog` 值（通常为 128）。
- `INADDR_ANY`：IPv4 通配地址（0.0.0.0）。
- `INADDR_LOOPBACK`：IPv4 回环地址（127.0.0.1）。
- `IN6ADDR_ANY_INIT`：IPv6 通配地址（`::`）的初始化器。
- `IN6ADDR_LOOPBACK_INIT`：IPv6 回环地址（`::1`）的初始化器。

---

## 四、类型定义

- `socklen_t`：无符号整数，用于地址结构长度。
- `sa_family_t`：地址族类型。
- `ssize_t`：有符号整数，用于字节数或错误。

---

## 五、模板声明

`<sys/socket.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

