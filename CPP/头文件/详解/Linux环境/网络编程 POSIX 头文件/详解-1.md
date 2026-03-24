# `<sys/socket.h>` 核心函数、结构体与宏详解（Linux / POSIX）

## 说明
`<sys/socket.h>` 是 POSIX 标准定义的套接字编程核心头文件，提供了创建网络通信端点的函数、通用地址结构以及各种常量和宏。本文档按照统一格式整理其中的函数、结构体、宏等信息，并说明内部原理与线程安全性。

---

## 一、函数详解

### 1. socket 函数
**函数原型：**
```c
int socket(int domain, int type, int protocol);
```
**作用：** 创建一个通信端点（套接字），返回一个文件描述符，后续操作通过该描述符进行。  
**参数：**  
- `domain`：协议族（`AF_INET`、`AF_INET6`、`AF_UNIX`、`AF_PACKET` 等）。  
- `type`：套接字类型（`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW` 等）。  
- `protocol`：具体协议，通常设为 `0` 让系统自动选择。  
**返回值：** 成功返回非负文件描述符，失败返回 `-1`。  
**示例用法：**
```c
int sockfd = socket(AF_INET, SOCK_STREAM, 0);
if (sockfd == -1) { perror("socket"); exit(1); }
```
**实现原理：** 系统调用。内核分配 `struct socket` 对象，关联文件描述符，初始化协议栈操作表。  
**线程安全提示：** 线程安全。多个线程同时调用不会相互干扰。

---

### 2. bind 函数
**函数原型：**
```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
**作用：** 将套接字与本地协议地址（IP + 端口）绑定。  
**参数：**  
- `sockfd`：socket 返回的描述符。  
- `addr`：指向协议地址结构的指针（如 `sockaddr_in`）。  
- `addrlen`：地址结构的长度。  
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
struct sockaddr_in addr;
memset(&addr, 0, sizeof(addr));
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = INADDR_ANY;
addr.sin_port = htons(8080);
if (bind(sockfd, (struct sockaddr*)&addr, sizeof(addr)) == -1) { perror("bind"); exit(1); }
```
**实现原理：** 系统调用。内核将地址信息保存到套接字对象，检查端口是否被占用（除非设置 `SO_REUSEADDR`）。  
**线程安全提示：** 线程安全。多个线程对同一未绑定套接字并发调用可能导致竞争，但函数本身不会破坏数据结构。

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
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
if (listen(sockfd, SOMAXCONN) == -1) { perror("listen"); exit(1); }
```
**实现原理：** 内核将套接字状态转为 `LISTEN`，创建半连接队列和全连接队列，开始接收 SYN 报文。  
**线程安全提示：** 线程安全。多线程对同一套接字调用 `listen` 只有第一个成功。

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
**返回值：** 成功返回新的文件描述符，失败返回 -1。  
**示例用法：**
```c
struct sockaddr_in client_addr;
socklen_t len = sizeof(client_addr);
int client_fd = accept(sockfd, (struct sockaddr*)&client_addr, &len);
if (client_fd == -1) { perror("accept"); exit(1); }
```
**实现原理：** 系统调用。若全连接队列为空则阻塞，取出连接后创建新套接字对象，分配新文件描述符。  
**线程安全提示：** 线程安全。多线程可并发 `accept` 同一监听套接字，内核互斥保护队列。

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
**返回值：** 成功返回 0，失败返回 -1。  
**示例用法：**
```c
struct sockaddr_in server;
server.sin_family = AF_INET;
server.sin_port = htons(8080);
inet_pton(AF_INET, "127.0.0.1", &server.sin_addr);
if (connect(sockfd, (struct sockaddr*)&server, sizeof(server)) == -1) { perror("connect"); exit(1); }
```
**实现原理：** TCP 发送 SYN 报文，完成三次握手；UDP 仅记录对端地址。  
**线程安全提示：** 线程安全，但同一套接字不应被多线程同时连接。

---

### 6. send 函数
**函数原型：**
```c
ssize_t send(int sockfd, const void *buf, size_t len, int flags);
```
**作用：** 在已连接的套接字上发送数据（TCP）。  
**参数：**  
- `flags`：控制选项（`MSG_DONTWAIT`、`MSG_NOSIGNAL` 等）。  
**返回值：** 成功返回实际发送字节数，失败返回 -1。  
**示例用法：**
```c
ssize_t sent = send(client_fd, "Hello", 5, 0);
if (sent == -1) perror("send");
```
**实现原理：** 数据拷贝到内核发送缓冲区，TCP 协议栈根据拥塞控制发送。  
**线程安全提示：** 线程安全。多线程并发发送时，内核保证每次 `send` 的原子性，但顺序可能交叉。

---

### 7. recv 函数
**函数原型：**
```c
ssize_t recv(int sockfd, void *buf, size_t len, int flags);
```
**作用：** 在已连接的套接字上接收数据（TCP）。  
**返回值：** 成功返回实际接收字节数，对端关闭返回 0，失败返回 -1。  
**示例用法：**
```c
char buffer[1024];
ssize_t n = recv(client_fd, buffer, sizeof(buffer), 0);
if (n > 0) { buffer[n] = '\0'; printf("%s\n", buffer); }
```
**实现原理：** 从内核接收缓冲区读取数据，若为空则阻塞。  
**线程安全提示：** 线程安全，但多线程读取同一套接字可能导致数据混乱，需同步。

---

### 8. sendto 函数
**函数原型：**
```c
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
```
**作用：** 用于无连接套接字（UDP）发送数据，可指定目标地址。  
**示例用法：** 见前文。  
**线程安全提示：** 线程安全，每次发送一个完整数据报，互不干扰。

---

### 9. recvfrom 函数
**函数原型：**
```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
```
**作用：** 用于无连接套接字（UDP）接收数据，同时返回源地址。  
**线程安全提示：** 线程安全，每个数据报只被一个线程接收。

---

### 10. setsockopt / getsockopt 函数
**函数原型：**
```c
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t *optlen);
```
**作用：** 设置/获取套接字选项。  
**参数：** `level` 指定协议层（`SOL_SOCKET`、`IPPROTO_TCP` 等），`optname` 指定选项名。  
**线程安全提示：** 线程安全。多线程可并发设置不同选项，但修改同一选项可能竞争。

---

### 11. shutdown 函数
**函数原型：**
```c
int shutdown(int sockfd, int how);
```
**作用：** 独立关闭套接字的读或写方向。  
**参数：** `how` 为 `SHUT_RD`、`SHUT_WR` 或 `SHUT_RDWR`。  
**线程安全提示：** 线程安全，但应与读写操作同步。

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
**派生结构：** `sockaddr_in`、`sockaddr_in6`、`sockaddr_un` 等。

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

### 7. struct sockaddr_in（定义在 `<netinet/in.h>`）
**定义：**
```c
struct sockaddr_in {
    sa_family_t    sin_family;   // AF_INET
    in_port_t      sin_port;     // 端口（网络字节序）
    struct in_addr sin_addr;     // IPv4 地址
    unsigned char  sin_zero[8];  // 填充
};
struct in_addr {
    in_addr_t s_addr;   // IPv4 地址（网络字节序）
};
```
**作用：** IPv4 专用地址结构。

---

### 8. struct sockaddr_in6（定义在 `<netinet/in.h>`）
**定义：**
```c
struct sockaddr_in6 {
    sa_family_t     sin6_family;   // AF_INET6
    in_port_t       sin6_port;     // 端口（网络字节序）
    uint32_t        sin6_flowinfo; // 流信息
    struct in6_addr sin6_addr;     // IPv6 地址
    uint32_t        sin6_scope_id; // 作用域 ID
};
struct in6_addr {
    unsigned char s6_addr[16];     // IPv6 地址
};
```
**作用：** IPv6 专用地址结构。

---

### 9. struct sockaddr_un（定义在 `<sys/un.h>`）
**定义：**
```c
struct sockaddr_un {
    sa_family_t sun_family;   // AF_UNIX
    char        sun_path[108]; // 套接字文件路径
};
```
**作用：** Unix 域套接字地址结构。

---

### 10. struct sockaddr_ll（定义在 `<linux/if_packet.h>` 或 `<netpacket/packet.h>`）
**定义：**
```c
struct sockaddr_ll {
    unsigned short sll_family;   // AF_PACKET
    unsigned short sll_protocol; // 协议类型
    int            sll_ifindex;  // 接口索引
    unsigned short sll_hatype;   // 硬件类型
    unsigned char  sll_pkttype;  // 包类型
    unsigned char  sll_halen;    // 硬件地址长度
    unsigned char  sll_addr[8];  // 硬件地址
};
```
**作用：** 链路层地址结构，用于原始套接字（`AF_PACKET`）。

---

### 11. struct sockaddr_nl（定义在 `<linux/netlink.h>`）
**定义：**
```c
struct sockaddr_nl {
    sa_family_t nl_family;  // AF_NETLINK
    unsigned short nl_pad;  // 填充
    __u32        nl_pid;    // 端口 ID
    __u32        nl_groups; // 多播组掩码
};
```
**作用：** Netlink 套接字地址，用于内核与用户空间通信。

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
| `AF_BLUETOOTH` | 蓝牙协议（Linux 特有） |
| `AF_ALG` | 加密算法套接字（Linux 特有） |
| `AF_UNSPEC` | 未指定地址族 |

---

### 2. 套接字类型常量（type）
| 宏名称 | 作用 |
|--------|------|
| `SOCK_STREAM` | 流式套接字（TCP） |
| `SOCK_DGRAM` | 数据报套接字（UDP） |
| `SOCK_RAW` | 原始套接字（直接访问 IP 层） |
| `SOCK_SEQPACKET` | 顺序数据包套接字（SCTP） |
| `SOCK_RDM` | 可靠数据报套接字（较少使用） |
| `SOCK_PACKET` | 过时的链路层套接字（Linux，已弃用） |
| 可组合标志 | `SOCK_NONBLOCK`、`SOCK_CLOEXEC`（Linux 扩展） |

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
- `AF_MAX`：最大地址族值（通常用于数组大小）。

---

## 四、模板声明
`<sys/socket.h>` 是 C 标准头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

## 五、环境判定
`<sys/socket.h>` 是 POSIX 标准头文件，在 Linux、macOS、BSD 等 Unix-like 系统上普遍存在。  
在 Windows 环境中，网络编程需包含 `<winsock2.h>`，并需调用 `WSAStartup` 初始化 Winsock 库，函数名和行为有细微差异（如 `close` 改为 `closesocket`，错误码使用 `WSAGetLastError`）。因此，该头文件明确指示目标环境为 **Linux / Unix / POSIX 兼容系统**。