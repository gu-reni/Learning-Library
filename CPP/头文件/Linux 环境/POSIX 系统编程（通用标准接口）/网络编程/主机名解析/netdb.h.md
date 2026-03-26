# `<netdb.h>` 核心函数、结构体与宏详解（Linux / POSIX）

## 说明
`<netdb.h>` 是 POSIX 标准定义的头文件，提供了网络数据库（主机名、服务名、协议名等）的查询接口。它包含了主机名解析（`gethostbyname`、`getaddrinfo`）、服务名解析（`getservbyname`、`getservbyport`）等函数，以及相关的结构体和错误码。本文档按照统一格式整理其中的函数、结构体、宏等信息，并说明内部原理与线程安全性。

---

## 一、函数详解

### 1. gethostbyname 函数
**函数原型：**
```c
struct hostent *gethostbyname(const char *name);
```
**作用：** 根据主机名（域名或 IP 地址字符串）获取主机信息（IP 地址列表等）。  
**参数：**  
- `name`：主机名（如 `"www.example.com"`）或点分十进制 IPv4 地址（如 `"192.168.1.1"`）。  
**返回值：** 成功返回指向 `struct hostent` 的指针（静态缓冲区），失败返回 `NULL`，并设置 `h_errno`。  
**示例用法：**
```c
struct hostent *he = gethostbyname("www.google.com");
if (he == NULL) {
    herror("gethostbyname");
    exit(1);
}
printf("Official name: %s\n", he->h_name);
```
**实现原理：**  
1. 检查输入字符串是否为点分十进制 IPv4 地址，若是则直接构造 `hostent`。  
2. 否则，查询本地 `/etc/hosts` 文件或 DNS 服务器（通过 `resolv.conf`）。  
3. 结果存储在静态缓冲区中返回。  
**线程安全提示：** **非线程安全**，使用静态缓冲区。多线程环境应使用 `getaddrinfo` 或可重入版本 `gethostbyname_r`。

---

### 2. gethostbyaddr 函数
**函数原型：**
```c
struct hostent *gethostbyaddr(const void *addr, socklen_t len, int type);
```
**作用：** 根据二进制 IP 地址获取主机信息（反向解析）。  
**参数：**  
- `addr`：指向二进制地址的指针（网络字节序）。  
- `len`：地址长度（IPv4 为 4，IPv6 为 16）。  
- `type`：地址族（`AF_INET` 或 `AF_INET6`）。  
**返回值：** 成功返回指向 `struct hostent` 的指针，失败返回 `NULL`。  
**线程安全提示：** **非线程安全**，使用静态缓冲区。

---

### 3. getservbyname 函数
**函数原型：**
```c
struct servent *getservbyname(const char *name, const char *proto);
```
**作用：** 根据服务名和协议（如 `"http"`、`"tcp"`）获取服务信息（端口号等）。  
**参数：**  
- `name`：服务名（如 `"http"`、`"ftp"`）。  
- `proto`：协议名（`"tcp"` 或 `"udp"`），若为 `NULL` 则返回任一协议的服务。  
**返回值：** 成功返回指向 `struct servent` 的指针（静态缓冲区），失败返回 `NULL`。  
**示例用法：**
```c
struct servent *se = getservbyname("http", "tcp");
if (se) {
    printf("Port: %d\n", ntohs(se->s_port));
}
```
**实现原理：** 查询 `/etc/services` 文件。  
**线程安全提示：** **非线程安全**，使用静态缓冲区。

---

### 4. getservbyport 函数
**函数原型：**
```c
struct servent *getservbyport(int port, const char *proto);
```
**作用：** 根据端口号和协议获取服务信息。  
**参数：**  
- `port`：端口号（网络字节序）。  
- `proto`：协议名（`"tcp"` 或 `"udp"`），若为 `NULL` 则返回任一协议的服务。  
**返回值：** 成功返回指向 `struct servent` 的指针，失败返回 `NULL`。  
**线程安全提示：** **非线程安全**，使用静态缓冲区。

---

### 5. getaddrinfo 函数
**函数原型：**
```c
int getaddrinfo(const char *node, const char *service,
                const struct addrinfo *hints, struct addrinfo **res);
```
**作用：** 将主机名和服务名转换为套接字地址结构列表（现代推荐函数，支持 IPv4/IPv6，线程安全）。  
**参数：**  
- `node`：主机名（如 `"www.example.com"`）或 IP 地址字符串（如 `"192.168.1.1"`），可为 `NULL`（与 `AI_PASSIVE` 配合）。  
- `service`：服务名（如 `"http"`）或端口号字符串（如 `"8080"`），可为 `NULL`。  
- `hints`：指向 `struct addrinfo` 的指针，用于指定筛选条件（如地址族、套接字类型等），可为 `NULL`。  
- `res`：输出参数，指向链表头指针，需用 `freeaddrinfo` 释放。  
**返回值：** 成功返回 0，失败返回非零错误码（`EAI_*`）。  
**示例用法：**
```c
struct addrinfo hints = {0};
hints.ai_family = AF_UNSPEC;    // IPv4 或 IPv6
hints.ai_socktype = SOCK_STREAM;
struct addrinfo *result;
int err = getaddrinfo("www.google.com", "80", &hints, &result);
if (err != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(err));
    exit(1);
}
// 使用 result 链表
freeaddrinfo(result);
```
**实现原理：**  
1. 根据 `hints` 确定查询类型（IPv4/IPv6、TCP/UDP 等）。  
2. 若 `node` 为 IP 地址字符串，则直接转换；否则进行 DNS 查询。  
3. 若 `service` 为服务名，则查询 `/etc/services` 获取端口号。  
4. 构建 `addrinfo` 链表，每个节点包含一个 `sockaddr` 结构。  
**线程安全提示：** **线程安全**（结果分配在动态内存中）。

---

### 6. freeaddrinfo 函数
**函数原型：**
```c
void freeaddrinfo(struct addrinfo *res);
```
**作用：** 释放 `getaddrinfo` 返回的链表及其关联的所有内存。  
**参数：**  
- `res`：`getaddrinfo` 返回的链表头指针。  
**线程安全提示：** 线程安全。

---

### 7. getnameinfo 函数
**函数原型：**
```c
int getnameinfo(const struct sockaddr *sa, socklen_t salen,
                char *host, socklen_t hostlen,
                char *serv, socklen_t servlen, int flags);
```
**作用：** 将套接字地址结构转换为主机名和服务名（反向解析，线程安全）。  
**参数：**  
- `sa`：指向 `sockaddr` 结构的指针。  
- `salen`：地址结构长度。  
- `host`：输出缓冲区，存放主机名。  
- `hostlen`：主机缓冲区大小（至少 `NI_MAXHOST`）。  
- `serv`：输出缓冲区，存放服务名。  
- `servlen`：服务缓冲区大小（至少 `NI_MAXSERV`）。  
- `flags`：控制标志（`NI_NUMERICHOST`、`NI_NUMERICSERV` 等）。  
**返回值：** 成功返回 0，失败返回非零错误码。  
**示例用法：**
```c
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = htonl(0xC0A80101);  // 192.168.1.1
addr.sin_port = htons(80);
char host[NI_MAXHOST];
char serv[NI_MAXSERV];
int err = getnameinfo((struct sockaddr*)&addr, sizeof(addr),
                       host, sizeof(host), serv, sizeof(serv),
                       NI_NUMERICHOST | NI_NUMERICSERV);
if (err == 0) {
    printf("Host: %s, Service: %s\n", host, serv);
}
```
**实现原理：**  
1. 若 `flags` 包含 `NI_NUMERICHOST`，则直接将 IP 地址格式化为字符串。  
2. 否则，进行反向 DNS 查询。  
3. 若 `flags` 包含 `NI_NUMERICSERV`，则直接输出端口号；否则查询 `/etc/services` 获取服务名。  
**线程安全提示：** **线程安全**（写入用户提供的缓冲区）。

---

### 8. 主机数据库控制函数（较少用）
- `void sethostent(int stayopen);`：打开主机数据库文件（如 `/etc/hosts`）并可选保持打开状态。  
- `void endhostent(void);`：关闭主机数据库文件。  
- `void setservent(int stayopen);`：打开服务数据库文件（`/etc/services`）。  
- `void endservent(void);`：关闭服务数据库文件。  

这些函数通常用于多次查询时避免重复打开文件，但现代应用更推荐使用 `getaddrinfo`。

**线程安全提示：** 这些函数通常不是线程安全的，因为可能修改全局状态。

---

## 二、结构体详解

### 1. struct hostent
**定义：**
```c
struct hostent {
    char  *h_name;            // 官方主机名
    char **h_aliases;         // 别名列表（以 NULL 结尾）
    int    h_addrtype;        // 地址族（AF_INET 或 AF_INET6）
    int    h_length;          // 地址长度（4 或 16）
    char **h_addr_list;       // 地址列表（网络字节序，以 NULL 结尾）
};
```
**作用：** 存储主机信息，由 `gethostbyname`、`gethostbyaddr` 等返回。  
**成员详解：**  
- `h_name`：官方主机名。  
- `h_aliases`：别名数组，每个元素是字符串指针，最后为 `NULL`。  
- `h_addrtype`：地址类型（`AF_INET` 或 `AF_INET6`）。  
- `h_length`：地址长度（IPv4 为 4，IPv6 为 16）。  
- `h_addr_list`：IP 地址列表，每个元素是二进制地址指针，网络字节序，最后为 `NULL`。  
**使用注意事项：** 该结构体使用静态缓冲区，不可跨线程使用，且后续调用可能覆盖之前结果。

---

### 2. struct servent
**定义：**
```c
struct servent {
    char  *s_name;      // 官方服务名
    char **s_aliases;   // 别名列表（以 NULL 结尾）
    int    s_port;      // 端口号（网络字节序）
    char  *s_proto;     // 协议名（"tcp" 或 "udp"）
};
```
**作用：** 存储服务信息，由 `getservbyname`、`getservbyport` 返回。  
**成员详解：**  
- `s_name`：服务名（如 `"http"`）。  
- `s_aliases`：别名数组。  
- `s_port`：端口号（网络字节序）。  
- `s_proto`：协议名（`"tcp"` 或 `"udp"`）。  
**使用注意事项：** 使用静态缓冲区，线程不安全。

---

### 3. struct addrinfo
**定义：**
```c
struct addrinfo {
    int              ai_flags;      // 选项标志（AI_PASSIVE 等）
    int              ai_family;     // 地址族（AF_INET、AF_INET6）
    int              ai_socktype;   // 套接字类型（SOCK_STREAM、SOCK_DGRAM）
    int              ai_protocol;   // 协议（0 表示自动）
    socklen_t        ai_addrlen;    // ai_addr 的长度
    struct sockaddr *ai_addr;       // 套接字地址结构
    char            *ai_canonname;  // 规范主机名（仅在 AI_CANONNAME 时设置）
    struct addrinfo *ai_next;       // 下一个节点（链表）
};
```
**作用：** 用于 `getaddrinfo` 的输入提示和输出结果。  
**成员详解：**  
- `ai_flags`：选项标志（`AI_PASSIVE`、`AI_CANONNAME`、`AI_NUMERICHOST` 等）。  
- `ai_family`、`ai_socktype`、`ai_protocol`：与 `socket` 参数类似，用于筛选结果。  
- `ai_addrlen`：`ai_addr` 指向的地址结构长度。  
- `ai_addr`：指向 `sockaddr_in` 或 `sockaddr_in6` 的指针。  
- `ai_canonname`：规范主机名（仅在指定 `AI_CANONNAME` 时有效）。  
- `ai_next`：下一个节点，形成链表。  
**使用注意事项：** `getaddrinfo` 动态分配内存，必须用 `freeaddrinfo` 释放。

---

## 三、宏定义详解

### 1. 错误码（用于 h_errno）
这些宏通常在 `<netdb.h>` 中定义，用于 `gethostbyname` 等函数的错误处理。

| 宏名称 | 作用 |
|--------|------|
| `HOST_NOT_FOUND` | 主机不存在 |
| `TRY_AGAIN` | 临时错误（如 DNS 服务器不可用） |
| `NO_RECOVERY` | 不可恢复的错误 |
| `NO_DATA` | 请求的类型没有数据（如 IPv4 地址不存在） |
| `NO_ADDRESS` | 同 `NO_DATA` |

**示例：**
```c
extern int h_errno;
if (he == NULL) {
    switch (h_errno) {
        case HOST_NOT_FOUND: ...
    }
}
```

---

### 2. getaddrinfo / getnameinfo 相关标志

#### 2.1 ai_flags 标志（用于 `addrinfo` 结构）
| 宏名称 | 作用 |
|--------|------|
| `AI_PASSIVE` | 返回适合 `bind` 的地址（`INADDR_ANY` 或 `in6addr_any`），通常用于服务器 |
| `AI_CANONNAME` | 返回规范主机名（在 `ai_canonname` 中） |
| `AI_NUMERICHOST` | 强制将 `node` 解释为数字地址字符串，不进行 DNS 查询 |
| `AI_NUMERICSERV` | 强制将 `service` 解释为端口号字符串 |
| `AI_V4MAPPED` | 若未找到 IPv6 地址，返回 IPv4 映射的 IPv6 地址 |
| `AI_ALL` | 与 `AI_V4MAPPED` 配合，返回所有 IPv6 和映射的 IPv4 地址 |
| `AI_ADDRCONFIG` | 仅返回本机已配置的地址族对应的地址 |

---

#### 2.2 getnameinfo flags
| 宏名称 | 作用 |
|--------|------|
| `NI_NUMERICHOST` | 返回 IP 地址字符串（不进行反向 DNS 查询） |
| `NI_NUMERICSERV` | 返回端口号字符串（不查询服务名） |
| `NI_NAMEREQD` | 若无法获取主机名则返回错误 |
| `NI_DGRAM` | 指定服务为 UDP（某些服务 TCP/UDP 端口不同） |
| `NI_NOFQDN` | 返回主机名时只返回第一个点之前的部分（本地） |
| `NI_MAXHOST` | 主机名字符串的最大长度（1025） |
| `NI_MAXSERV` | 服务名字符串的最大长度（32） |

---

### 3. getaddrinfo 错误码（EAI_*）
这些宏用于 `getaddrinfo` 的返回值，可通过 `gai_strerror` 转换为字符串。

| 宏名称 | 作用 |
|--------|------|
| `EAI_ADDRFAMILY` | 指定地址族无地址 |
| `EAI_AGAIN` | 临时 DNS 错误 |
| `EAI_BADFLAGS` | `hints.ai_flags` 无效 |
| `EAI_FAIL` | 永久 DNS 错误 |
| `EAI_FAMILY` | 不支持的地址族 |
| `EAI_MEMORY` | 内存不足 |
| `EAI_NODATA` | 无数据（已废弃） |
| `EAI_NONAME` | 未知主机或服务 |
| `EAI_SERVICE` | 未知服务 |
| `EAI_SOCKTYPE` | 不支持的套接字类型 |
| `EAI_SYSTEM` | 系统错误（检查 errno） |
| `EAI_OVERFLOW` | 缓冲区溢出 |

---

### 4. 其他常量
- `IPPORT_RESERVED`：保留端口上限（1024）。
- `IPPORT_USERRESERVED`：用户保留端口上限（5000，较老定义）。

---

## 四、类型定义
- `h_errno`：外部整数，用于主机查询错误。
- `socklen_t`：套接字地址长度类型（在 `<sys/socket.h>` 定义）。

---

## 五、模板声明
`<netdb.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

## 六、头文件分类
根据您提供的分类树，`<netdb.h>` 的归类为：
