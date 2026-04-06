## `<arpa/inet.h>` 头文件详解（Linux / POSIX）

`<arpa/inet.h>` 是 POSIX 标准定义的网络编程头文件，提供了网络地址转换函数（如 `inet_pton`、`inet_ntop`）以及一些过时的地址转换函数（`inet_addr`、`inet_ntoa`、`inet_aton`）。它与 `<netinet/in.h>` 配合使用，后者定义了 IPv4/IPv6 地址结构。

---

## 一、函数详解

### 1. inet_addr

**函数原型：**
```c
in_addr_t inet_addr(const char *cp);
```

**作用：** 将点分十进制 IPv4 地址字符串转换为网络字节序的 32 位整数。

**参数：**
- `cp`：点分十进制字符串（如 `"192.168.1.1"`）。

**返回值：** 成功返回转换后的地址（网络字节序），失败返回 `INADDR_NONE`（通常为 `(in_addr_t)-1`）。

**示例用法：**
```c
in_addr_t addr = inet_addr("192.168.1.1");
if (addr == INADDR_NONE) { /* 错误 */ }
```

**实现原理：** 解析字符串，计算数值，并转换为网络字节序。

**线程安全提示：** 线程安全（不依赖静态缓冲区），但返回值 `INADDR_NONE` 与 `255.255.255.255` 的转换结果冲突（均为 -1），因此不推荐使用，改用 `inet_aton` 或 `inet_pton`。

---

### 2. inet_aton

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

### 3. inet_ntoa

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

### 4. inet_network

**函数原型：**
```c
in_addr_t inet_network(const char *cp);
```

**作用：** 将点分十进制 IPv4 地址转换为主机字节序的整数（用于网络地址）。

**参数：**
- `cp`：点分十进制字符串。

**返回值：** 成功返回转换后的值，失败返回 -1。

**示例用法：**
```c
in_addr_t net = inet_network("192.168.1.0");
```

**线程安全提示：** 线程安全。

---

### 5. inet_pton

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

### 6. inet_ntop

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

## 二、结构体详解

`<arpa/inet.h>` 本身不定义结构体。它使用 `<netinet/in.h>` 中定义的 `struct in_addr`、`struct in6_addr` 等。

---

## 三、宏定义详解

### 1. IPv4 地址常量

| 宏名称 | 作用 |
|--------|------|
| `INADDR_ANY` | 通配地址（0.0.0.0），表示绑定所有接口 |
| `INADDR_LOOPBACK` | 回环地址（127.0.0.1） |
| `INADDR_BROADCAST` | 广播地址（255.255.255.255） |
| `INADDR_NONE` | 用于 `inet_addr` 失败返回值（-1） |

这些常量通常在 `<netinet/in.h>` 中定义，但 `<arpa/inet.h>` 会间接包含。

---

### 2. IPv6 地址常量（静态初始化器）

| 宏名称 | 作用 |
|--------|------|
| `IN6ADDR_ANY_INIT` | 通配地址（`::`）的初始化器 |
| `IN6ADDR_LOOPBACK_INIT` | 回环地址（`::1`）的初始化器 |

---

### 3. 地址字符串长度宏

| 宏名称 | 作用 |
|--------|------|
| `INET_ADDRSTRLEN` | IPv4 点分十进制字符串的最大长度（16） |
| `INET6_ADDRSTRLEN` | IPv6 十六进制字符串的最大长度（46） |

这些宏同样在 `<netinet/in.h>` 中定义。

---

## 四、类型定义

- `in_addr_t`：IPv4 地址类型（通常为 `uint32_t`）。
- `in_port_t`：端口号类型（通常为 `uint16_t`）。
- `sa_family_t`：地址族类型。

这些类型在 `<sys/types.h>` 或 `<netinet/in.h>` 中定义。

---

## 五、模板声明

`<arpa/inet.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

