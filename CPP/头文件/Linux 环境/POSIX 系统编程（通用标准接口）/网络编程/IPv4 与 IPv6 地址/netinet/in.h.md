# `<netinet/in.h>` 核心函数、结构体与宏详解（Linux / POSIX）

## 说明
`<netinet/in.h>` 是 POSIX 标准定义的头文件，提供了 IPv4 和 IPv6 地址结构、字节序转换函数、地址转换函数以及相关常量。它是网络编程（特别是 TCP/IP 协议族）的核心头文件之一，通常与 `<sys/socket.h>` 配合使用。本文档按照统一格式整理其中的函数、结构体、宏等信息，并说明内部原理与线程安全性。

---

## 一、函数详解

### 1. htonl / htons / ntohl / ntohs 函数
**函数原型：**
```c
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```
**作用：** 在主机字节序与网络字节序（大端）之间转换 16 位和 32 位整数。  
**参数：**  
- `hostlong` / `hostshort`：主机字节序的 32/16 位整数。  
- `netlong` / `netshort`：网络字节序的 32/16 位整数。  
**返回值：** 转换后的整数值。  
**示例用法：**
```c
uint16_t port = htons(8080);   // 主机端口转网络字节序
uint32_t addr = htonl(0x0100007F); // 主机 IP 转网络字节序（注意此处示例仅为演示，实际用 inet_pton）
```
**实现原理：**  
这些函数通常被实现为内联函数或宏，在小端机器上执行字节交换，在大端机器上直接返回原值。内核和编译器会高效处理。  
**线程安全提示：** 这些是纯计算函数，完全线程安全。

---

### 2. inet_addr 函数
**函数原型：**
```c
in_addr_t inet_addr(const char *cp);
```
**作用：** 将点分十进制 IPv4 地址字符串转换为网络字节序的 32 位整数（`in_addr_t`）。  
**参数：**  
- `cp`：点分十进制字符串（如 `"192.168.1.1"`）。  
**返回值：** 成功返回转换后的地址，失败返回 `INADDR_NONE`（通常为 `(in_addr_t)-1`）。  
**示例用法：**
```c
in_addr_t addr = inet_addr("192.168.1.1");
if (addr == INADDR_NONE) { /* 错误 */ }
```
**实现原理：** 解析字符串，计算数值，并转换为网络字节序。  
**线程安全提示：** 线程安全（不依赖静态缓冲区），但返回值 `INADDR_NONE` 与 `255.255.255.255` 的转换结果冲突（均为 -1），因此不建议使用，改用 `inet_aton` 或 `inet_pton`。

---

### 3. inet_aton 函数
**函数原型：**
```c
int inet_aton(const char *cp, struct in_addr *inp);
```
**作用：** 将点分十进制 IPv4 地址字符串转换为网络字节序的 32 位整数，并存入 `inp`。  
**参数：**  
- `cp`：点分十进制字符串。  
- `inp`：指向 `struct in_addr` 的指针，用于存储结果。  
**返回值：** 成功返回非零（通常 1），失败返回 0。  
**示例用法：**
```c
struct in_addr addr;
if (inet_aton("192.168.1.1", &addr) == 0) { /* 错误 */ }
```
**实现原理：** 解析字符串，计算结果，设置 `inp->s_addr`。  
**线程安全提示：** 线程安全。

---

### 4. inet_ntoa 函数
**函数原型：**
```c
char *inet_ntoa(struct in_addr in);
```
**作用：** 将网络字节序的 IPv4 地址转换为点分十进制字符串。  
**参数：**  
- `in`：`struct in_addr` 结构（包含 `s_addr` 成员）。  
**返回值：** 指向静态缓冲区（线程不安全）的指针。  
**示例用法：**
```c
struct in_addr addr;
addr.s_addr = htonl(0xC0A80101);  // 192.168.1.1
printf("%s\n", inet_ntoa(addr));
```
**实现原理：** 将整数分解为四个字节，格式化为字符串，存放在静态缓冲区中返回。  
**线程安全提示：** **非线程安全**，使用静态缓冲区。多线程环境应使用 `inet_ntop`。

---

### 5. inet_pton 函数
**函数原型：**
```c
int inet_pton(int af, const char *src, void *dst);
```
**作用：** 将 IPv4 或 IPv6 地址字符串转换为二进制形式（网络字节序）。  
**参数：**  
- `af`：地址族（`AF_INET` 或 `AF_INET6`）。  
- `src`：地址字符串（IPv4 点分十进制，IPv6 十六进制冒号分隔）。  
- `dst`：指向存储结果的缓冲区（对于 `AF_INET`，应为 `struct in_addr *`；对于 `AF_INET6`，应为 `struct in6_addr *`）。  
**返回值：** 成功返回 1；字符串无效返回 0；地址族不支持返回 -1。  
**示例用法：**
```c
struct in_addr addr4;
struct in6_addr addr6;
if (inet_pton(AF_INET, "192.168.1.1", &addr4) != 1) { /* 错误 */ }
if (inet_pton(AF_INET6, "::1", &addr6) != 1) { /* 错误 */ }
```
**实现原理：** 内核解析地址字符串并填充二进制结构。  
**线程安全提示：** 线程安全（结果写入用户提供的缓冲区）。

---

### 6. inet_ntop 函数
**函数原型：**
```c
const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);
```
**作用：** 将二进制 IPv4 或 IPv6 地址转换为字符串形式。  
**参数：**  
- `af`：地址族（`AF_INET` 或 `AF_INET6`）。  
- `src`：指向二进制地址的指针（`struct in_addr *` 或 `struct in6_addr *`）。  
- `dst`：指向输出缓冲区的指针。  
- `size`：输出缓冲区的大小（至少 `INET_ADDRSTRLEN` 或 `INET6_ADDRSTRLEN`）。  
**返回值：** 成功返回 `dst`（指向结果字符串），失败返回 `NULL`。  
**示例用法：**
```c
struct in_addr addr4;
addr4.s_addr = htonl(0xC0A80101);
char str[INET_ADDRSTRLEN];
if (inet_ntop(AF_INET, &addr4, str, sizeof(str)) != NULL) {
    printf("%s\n", str);  // 192.168.1.1
}
```
**实现原理：** 将二进制地址格式化为字符串。  
**线程安全提示：** 线程安全（写入用户提供的缓冲区）。

---

### 7. inet_network 函数（较老，较少用）
**函数原型：**
```c
in_addr_t inet_network(const char *cp);
```
**作用：** 将点分十进制 IPv4 地址转换为主机字节序的整数（用于网络地址）。  
**返回值：** 成功返回转换后的值，失败返回 -1。  
**线程安全提示：** 线程安全。

---

## 二、结构体详解

### 1. struct in_addr
**定义：**
```c
struct in_addr {
    in_addr_t s_addr;   // IPv4 地址（网络字节序）
};
```
**作用：** 存储 IPv4 地址。  
**成员详解：** `s_addr` 为 32 位网络字节序整数。

---

### 2. struct sockaddr_in（IPv4 套接字地址）
**定义：**
```c
struct sockaddr_in {
    sa_family_t    sin_family;   // AF_INET
    in_port_t      sin_port;     // 端口号（网络字节序）
    struct in_addr sin_addr;     // IPv4 地址
    unsigned char  sin_zero[8];  // 填充，使结构体与 sockaddr 大小相同
};
```
**作用：** IPv4 专用地址结构，用于 `bind`、`connect`、`accept` 等函数。  
**成员详解：**  
- `sin_family`：必须为 `AF_INET`。  
- `sin_port`：端口号（网络字节序）。  
- `sin_addr`：IPv4 地址结构。  
- `sin_zero`：填充字段，通常使用 `memset` 清零。  

---

### 3. struct in6_addr
**定义：**
```c
struct in6_addr {
    unsigned char s6_addr[16];   // IPv6 地址（网络字节序）
};
```
**作用：** 存储 IPv6 地址。  
**成员详解：** `s6_addr` 为 128 位地址数组，网络字节序。

---

### 4. struct sockaddr_in6（IPv6 套接字地址）
**定义：**
```c
struct sockaddr_in6 {
    sa_family_t     sin6_family;   // AF_INET6
    in_port_t       sin6_port;     // 端口号（网络字节序）
    uint32_t        sin6_flowinfo; // 流信息（通常 0）
    struct in6_addr sin6_addr;     // IPv6 地址
    uint32_t        sin6_scope_id; // 作用域 ID（如链路本地地址的接口索引）
};
```
**作用：** IPv6 专用地址结构。  
**成员详解：**  
- `sin6_family`：必须为 `AF_INET6`。  
- `sin6_port`：端口号（网络字节序）。  
- `sin6_flowinfo`：流标签和优先级，通常设为 0。  
- `sin6_addr`：IPv6 地址。  
- `sin6_scope_id`：用于链路本地地址的接口索引。

---

## 三、宏定义详解

### 1. 地址族常量
| 宏名称 | 作用 |
|--------|------|
| `AF_INET` | IPv4 地址族 |
| `AF_INET6` | IPv6 地址族 |

（注：这些常量在 `<sys/socket.h>` 中也有定义，但 `<netinet/in.h>` 同样提供。）

---

### 2. 协议族常量（IPPROTO_ 前缀）
| 宏名称 | 作用 |
|--------|------|
| `IPPROTO_IP` | IPv4 伪协议 |
| `IPPROTO_TCP` | TCP 协议 |
| `IPPROTO_UDP` | UDP 协议 |
| `IPPROTO_ICMP` | ICMP 协议 |
| `IPPROTO_IPV6` | IPv6 协议 |
| `IPPROTO_RAW` | 原始 IP 包 |
| `IPPROTO_SCTP` | SCTP 协议 |

---

### 3. IPv4 地址常量
| 宏名称 | 作用 |
|--------|------|
| `INADDR_ANY` | 通配地址（0.0.0.0），表示绑定所有接口 |
| `INADDR_LOOPBACK` | 回环地址（127.0.0.1） |
| `INADDR_BROADCAST` | 广播地址（255.255.255.255） |
| `INADDR_NONE` | 用于 `inet_addr` 失败返回值（-1） |

---

### 4. IPv6 地址常量（静态初始化器）
| 宏名称 | 作用 |
|--------|------|
| `IN6ADDR_ANY_INIT` | 通配地址（`::`）的初始化器 |
| `IN6ADDR_LOOPBACK_INIT` | 回环地址（`::1`）的初始化器 |

**示例：**
```c
struct in6_addr in6_any = IN6ADDR_ANY_INIT;
struct in6_addr in6_loop = IN6ADDR_LOOPBACK_INIT;
```

---

### 5. 地址字符串长度宏
| 宏名称 | 作用 |
|--------|------|
| `INET_ADDRSTRLEN` | IPv4 点分十进制字符串的最大长度（16） |
| `INET6_ADDRSTRLEN` | IPv6 十六进制字符串的最大长度（46） |

---

### 6. 套接字选项（IPPROTO_IP 级别）
| 宏名称 | 作用 |
|--------|------|
| `IP_OPTIONS` | IP 选项 |
| `IP_TTL` | 生存时间 |
| `IP_TOS` | 服务类型 |
| `IP_MULTICAST_IF` | 多播接口 |
| `IP_MULTICAST_TTL` | 多播 TTL |
| `IP_MULTICAST_LOOP` | 多播回环 |
| `IP_ADD_MEMBERSHIP` | 加入多播组 |
| `IP_DROP_MEMBERSHIP` | 离开多播组 |

### 7. 套接字选项（IPPROTO_IPV6 级别）
| 宏名称 | 作用 |
|--------|------|
| `IPV6_MULTICAST_IF` | IPv6 多播接口 |
| `IPV6_MULTICAST_HOPS` | IPv6 多播跳数 |
| `IPV6_MULTICAST_LOOP` | IPv6 多播回环 |
| `IPV6_JOIN_GROUP` | 加入 IPv6 多播组 |
| `IPV6_LEAVE_GROUP` | 离开 IPv6 多播组 |
| `IPV6_V6ONLY` | 仅 IPv6（禁止 IPv4 映射） |

---

## 四、类型定义
| 类型 | 作用 |
|------|------|
| `in_addr_t` | IPv4 地址类型（通常为 `uint32_t`） |
| `in_port_t` | 端口号类型（通常为 `uint16_t`） |
| `sa_family_t` | 地址族类型（通常为 `unsigned short`） |

这些类型在 `<sys/types.h>` 中定义，但 `<netinet/in.h>` 会引入它们。

---

## 五、模板声明
`<netinet/in.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

## 六、头文件分类
根据您提供的分类树，`<netinet/in.h>` 的归类为：

```
