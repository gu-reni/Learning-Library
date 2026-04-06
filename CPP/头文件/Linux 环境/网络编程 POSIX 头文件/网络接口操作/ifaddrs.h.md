## `<ifaddrs.h>` 头文件详解（Linux / POSIX）

`<ifaddrs.h>` 是 Linux 和 BSD 系统中用于获取网络接口地址的头文件。它提供了获取本地网络接口信息（如 IP 地址、子网掩码、广播地址等）的便捷接口，主要用于网络诊断、系统管理等需要枚举本地网络接口的场景。需要特别注意的是，`<ifaddrs.h>` 是 **Linux / BSD 特有接口**，Windows 环境不支持该函数，需使用其他方案（如 `GetAdaptersAddresses`）[reference:0][reference:1]。

---

## 一、函数详解

### 1. getifaddrs 函数

**函数原型：**
```c
int getifaddrs(struct ifaddrs **ifap);
```

**作用：** 创建并返回一个链表，链表中的每个节点描述本地系统的一个网络接口地址，并将链表的首地址存储在 `ifap` 指向的指针中。

**参数：**
- `ifap`：指向 `struct ifaddrs *` 的指针，用于接收链表首地址。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1，并设置 `errno` 以指示错误原因[reference:2]。

**示例用法：**
```c
#include <ifaddrs.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct ifaddrs *ifaddr, *ifa;
    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) continue;
        printf("接口: %s\n", ifa->ifa_name);
    }
    freeifaddrs(ifaddr);
    return 0;
}
```
**实现原理：**
`getifaddrs()` 通过 `netlink` 接口（在 Linux 上）与内核通信，获取网络接口配置信息。它创建 socket 并发送 `RTM_GETLINK` 和 `RTM_GETADDR` 等 netlink 消息，读取返回的信息并解析，动态分配内存构建 `ifaddrs` 链表[reference:3][reference:4]。

**线程安全提示：**
根据 Linux 手册页，`getifaddrs()` 和 `freeifaddrs()` 被标记为 **MT-Safe**（线程安全）[reference:5][reference:6]。

---

### 2. freeifaddrs 函数

**函数原型：**
```c
void freeifaddrs(struct ifaddrs *ifa);
```

**作用：** 释放 `getifaddrs()` 动态分配的 `ifaddrs` 链表内存。

**参数：**
- `ifa`：指向 `getifaddrs()` 返回的链表首地址的指针。

**返回值：** 无。

**示例用法：**
```c
freeifaddrs(ifaddr);  // 释放链表内存
```
**实现原理：**
遍历链表，释放每个节点及其关联的 `ifa_name`、`ifa_addr`、`ifa_netmask`、广播地址等动态分配的内存。

**线程安全提示：**
与 `getifaddrs()` 相同，`freeifaddrs()` 也是 MT-Safe[reference:7]。

---

## 二、结构体详解

### struct ifaddrs

**定义：**
```c
struct ifaddrs {
    struct ifaddrs *ifa_next;          // 指向链表中下一个结构的指针
    char            *ifa_name;          // 接口名称（以空字符结尾）
    unsigned int     ifa_flags;         // 接口标志（来自 SIOCGIFFLAGS ioctl）
    struct sockaddr *ifa_addr;          // 接口地址
    struct sockaddr *ifa_netmask;       // 接口的子网掩码
    union {
        struct sockaddr *ifu_broadaddr; // 广播地址（仅当 IFF_BROADCAST 标志设置时有效）
        struct sockaddr *ifu_dstaddr;   // 点对点目的地址（仅当 IFF_POINTOPOINT 标志设置时有效）
    } ifa_ifu;
#define ifa_broadaddr ifa_ifu.ifu_broadaddr
#define ifa_dstaddr   ifa_ifu.ifu_dstaddr
    void            *ifa_data;          // 地址族特定数据（通常为 NULL）
};
```
**作用：** 描述一个网络接口地址的信息。一个物理接口可能对应多个 `ifaddrs` 节点（例如分别对应 IPv4 和 IPv6 地址）。

**成员详解：**
- `ifa_next`：指向链表下一个节点，最后一个节点为 `NULL`[reference:8]。
- `ifa_name`：接口名称，如 `eth0`、`wlan0`、`lo`[reference:9]。
- `ifa_flags`：接口标志位，可通过检查 `IFF_UP`、`IFF_BROADCAST`、`IFF_POINTOPOINT`、`IFF_LOOPBACK` 等宏判断接口状态[reference:10]。
- `ifa_addr`：指向 `struct sockaddr` 的指针，包含接口的 IP 地址（IPv4 或 IPv6）。需检查 `sa_family` 字段确定地址族[reference:11]。
- `ifa_netmask`：指向包含子网掩码的 `struct sockaddr` 结构[reference:12]。
- `ifa_broadaddr` / `ifa_dstaddr`：通过联合体 `ifa_ifu` 实现，分别用于广播地址和点对点目的地址[reference:13]。
- `ifa_data`：地址族特定数据的缓冲区，通常为 `NULL`[reference:14]。

**使用注意事项：**
- 调用 `getifaddrs()` 后必须使用 `freeifaddrs()` 释放链表内存，防止内存泄漏[reference:15]。
- `ifa_addr`、`ifa_netmask` 等字段可能为 `NULL`，使用时需检查[reference:16]。
- `ifa_broadaddr` 仅在 `IFF_BROADCAST` 标志设置时有效，`ifa_dstaddr` 仅在 `IFF_POINTOPOINT` 标志设置时有效[reference:17]。

---

## 三、宏定义详解

### 1. 结构体字段宏

| 宏名称 | 作用 |
|--------|------|
| `ifa_broadaddr` | 访问联合体中的 `ifu_broadaddr`，获取广播地址 |
| `ifa_dstaddr` | 访问联合体中的 `ifu_dstaddr`，获取点对点目的地址 |

### 2. 接口标志宏（需包含 `<net/if.h>` 或 `<linux/if.h>`）

| 宏名称 | 作用 |
|--------|------|
| `IFF_UP` | 接口已启用 |
| `IFF_BROADCAST` | 支持广播 |
| `IFF_LOOPBACK` | 回环接口 |
| `IFF_POINTOPOINT` | 点对点接口（如 PPP） |
| `IFF_RUNNING` | 接口已运行 |
| `IFF_MULTICAST` | 支持多播 |
| `IFF_PROMISC` | 混杂模式 |
| `IFF_ALLMULTI` | 接收所有多播包 |
| `IFF_NOARP` | 不支持 ARP 协议 |

这些标志用于检查 `ifa_flags` 字段，判断接口属性和状态。

---

## 四、类型定义

- `struct ifaddrs`：描述网络接口信息的结构体。
- `struct sockaddr`：通用套接字地址结构（定义在 `<sys/socket.h>`），`ifa_addr`、`ifa_netmask`、`ifa_broadaddr`、`ifa_dstaddr` 均指向此类型。

---

## 五、示例用法

### 示例 1：获取所有接口的 IPv4 地址
```c
#include <ifaddrs.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct ifaddrs *ifaddr, *ifa;
    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) continue;
        if (ifa->ifa_addr->sa_family == AF_INET) {
            struct sockaddr_in *addr = (struct sockaddr_in *)ifa->ifa_addr;
            char ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &addr->sin_addr, ip, sizeof(ip));
            printf("%s: %s\n", ifa->ifa_name, ip);
        }
    }
    freeifaddrs(ifaddr);
    return 0;
}
```

### 示例 2：获取接口的广播地址
```c
#include <ifaddrs.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct ifaddrs *ifaddr, *ifa;
    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) continue;
        if ((ifa->ifa_flags & IFF_BROADCAST) && ifa->ifa_broadaddr) {
            struct sockaddr_in *addr = (struct sockaddr_in *)ifa->ifa_broadaddr;
            char ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &addr->sin_addr, ip, sizeof(ip));
            printf("%s broadcast: %s\n", ifa->ifa_name, ip);
        }
    }
    freeifaddrs(ifaddr);
    return 0;
}
```

### 示例 3：获取点对点接口的目的地址
```c
#include <ifaddrs.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct ifaddrs *ifaddr, *ifa;
    if (getifaddrs(&ifaddr) == -1) {
        perror("getifaddrs");
        exit(EXIT_FAILURE);
    }
    for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
        if (ifa->ifa_addr == NULL) continue;
        if ((ifa->ifa_flags & IFF_POINTOPOINT) && ifa->ifa_dstaddr) {
            struct sockaddr_in *addr = (struct sockaddr_in *)ifa->ifa_dstaddr;
            char ip[INET_ADDRSTRLEN];
            inet_ntop(AF_INET, &addr->sin_addr, ip, sizeof(ip));
            printf("%s destination: %s\n", ifa->ifa_name, ip);
        }
    }
    freeifaddrs(ifaddr);
    return 0;
}
```

---

## 六、实现原理

`getifaddrs()` 通过 `netlink` 套接字与内核通信获取接口信息，具体步骤：

1. 创建 `AF_NETLINK` 套接字，协议为 `NETLINK_ROUTE`。
2. 发送 `RTM_GETLINK` 消息获取接口索引和名称列表。
3. 发送 `RTM_GETADDR` 消息获取每个接口的地址信息。
4. 解析返回的 netlink 消息，为每个接口地址动态分配 `ifaddrs` 节点。
5. 填充各字段：`ifa_name`、`ifa_flags`、`ifa_addr`、`ifa_netmask`、`ifa_broadaddr`/`ifa_dstaddr` 等。
6. 将所有节点链接成链表，链表头地址存入 `ifap`。

---

## 七、线程安全

根据 Linux 手册页，`getifaddrs()` 和 `freeifaddrs()` 均被标记为 **MT-Safe**（线程安全）[reference:18][reference:19]。这意味着可以在多线程环境中安全调用，无需外部同步。

---

## 八、头文件分类

根据您提供的分类树，`<ifaddrs.h>` 的归类为：

```
Linux 环境 / Linux 特有接口 / 其他系统调用扩展
```

（注：`ifaddrs.h` 是 Linux/BSD 特有的获取网络接口地址的头文件，不属于 POSIX 标准，因此归类为 Linux 特有接口。）