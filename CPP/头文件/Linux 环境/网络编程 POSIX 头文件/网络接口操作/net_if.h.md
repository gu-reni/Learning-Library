## `<net/if.h>` 头文件详解（Linux / POSIX）

`<net/if.h>` 是 POSIX 标准和 Linux 系统提供的头文件，用于获取和操作网络接口信息。它定义了网络接口名称、索引（index）转换函数、接口标志宏以及相关的结构体。该头文件常用于需要枚举网络接口或根据接口名称查找索引的场景。

---

## 一、函数详解

### 1. if_nametoindex 函数

**函数原型：**
```c
unsigned int if_nametoindex(const char *ifname);
```

**作用：** 将网络接口名称转换为接口索引（index）。

**参数：**
- `ifname`：接口名称字符串（如 `"eth0"`、`"lo"`）。

**返回值：**
- 成功：返回接口索引（非零整数）。
- 失败：返回 0（表示没有对应名称的接口）。

**示例用法：**
```c
#include <net/if.h>
#include <stdio.h>

int main() {
    unsigned int idx = if_nametoindex("eth0");
    if (idx == 0) {
        printf("Interface not found\n");
    } else {
        printf("eth0 index: %u\n", idx);
    }
    return 0;
}
```

**实现原理：**
系统调用（或基于 `/proc/net/dev` 或 netlink）查找接口名称对应的索引。在 Linux 上通常通过 netlink 或直接读取 `/proc/net/if_inet6` 等文件实现。

**线程安全提示：**
线程安全（POSIX 要求）。

---

### 2. if_indextoname 函数

**函数原型：**
```c
char *if_indextoname(unsigned int ifindex, char *ifname);
```

**作用：** 将接口索引转换为接口名称。

**参数：**
- `ifindex`：接口索引（非零整数）。
- `ifname`：指向缓冲区的指针，用于存储接口名称。缓冲区大小至少为 `IFNAMSIZ`（通常为 16）。

**返回值：**
- 成功：返回 `ifname`（指向接口名称字符串）。
- 失败：返回 `NULL`，并设置 `errno`（如 `ENXIO` 表示没有对应索引的接口）。

**示例用法：**
```c
#include <net/if.h>
#include <stdio.h>
#include <string.h>

int main() {
    char ifname[IFNAMSIZ];
    if (if_indextoname(2, ifname) != NULL) {
        printf("Index 2 -> %s\n", ifname);
    } else {
        perror("if_indextoname");
    }
    return 0;
}
```

**实现原理：**
类似 `if_nametoindex`，通过内核接口映射表查找。

**线程安全提示：**
线程安全（写入用户提供的缓冲区）。

---

### 3. if_nameindex 函数

**函数原型：**
```c
struct if_nameindex *if_nameindex(void);
```

**作用：** 获取系统中所有网络接口的名称和索引列表，返回一个以 `NULL` 结尾的结构体数组。

**返回值：**
- 成功：返回指向 `struct if_nameindex` 数组首元素的指针。数组最后一个元素的 `if_index` 为 0，`if_name` 为 `NULL`。
- 失败：返回 `NULL`，并设置 `errno`。

**示例用法：**
```c
#include <net/if.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct if_nameindex *iflist, *ifp;
    iflist = if_nameindex();
    if (iflist == NULL) {
        perror("if_nameindex");
        exit(EXIT_FAILURE);
    }
    for (ifp = iflist; ifp->if_index != 0 && ifp->if_name != NULL; ifp++) {
        printf("%s: %u\n", ifp->if_name, ifp->if_index);
    }
    if_freenameindex(iflist);
    return 0;
}
```

**实现原理：**
内部通过 netlink 或系统调用获取所有接口信息，动态分配内存存储数组。

**线程安全提示：**
线程安全，但返回的数组需用 `if_freenameindex` 释放。

---

### 4. if_freenameindex 函数

**函数原型：**
```c
void if_freenameindex(struct if_nameindex *ptr);
```

**作用：** 释放 `if_nameindex()` 动态分配的内存。

**参数：**
- `ptr`：`if_nameindex()` 返回的指针。

**返回值：** 无。

**示例用法：**
```c
if_freenameindex(iflist);
```

**实现原理：**
遍历数组，释放每个节点中的 `if_name` 字符串和数组本身。

**线程安全提示：**
线程安全。

---

## 二、结构体详解

### struct if_nameindex

**定义：**
```c
struct if_nameindex {
    unsigned int if_index;  // 接口索引
    char        *if_name;   // 接口名称（以空字符结尾）
};
```

**作用：** 存储一个接口的名称和索引，用于 `if_nameindex()` 返回的数组。

**成员详解：**
- `if_index`：接口索引，非零正整数。
- `if_name`：指向接口名称字符串的指针。

---

### 其他相关结构体（可能在其他头文件中，但常与 `<net/if.h>` 一起使用）

- `struct ifreq`：通常在 `<net/if.h>` 中定义，用于 ioctl 操作（如 `SIOCGIFADDR` 获取接口地址）。`ifreq` 结构体包含一个接口名称和一个联合体（用于地址、标志等）。该结构体在 `<net/if.h>` 中定义，但本文件主要展示 POSIX 标准部分，`ifreq` 在 Linux 中也有定义。

**简化的 `ifreq` 定义（Linux）：**
```c
struct ifreq {
    char ifr_name[IFNAMSIZ];   // 接口名称
    union {
        struct sockaddr ifr_addr;
        struct sockaddr ifr_dstaddr;
        struct sockaddr ifr_broadaddr;
        struct sockaddr ifr_netmask;
        struct sockaddr ifr_hwaddr;
        short           ifr_flags;
        int             ifr_ifindex;
        // ... 其他字段
    };
};
```

---

## 三、宏定义详解

### 1. 常量宏

| 宏名称 | 作用 |
|--------|------|
| `IFNAMSIZ` | 接口名称的最大长度（包含终止符）。在 Linux 中通常为 16。 |

### 2. 接口标志宏（用于 `ifr_flags`，需通过 ioctl 获取）

这些标志与 `<net/if.h>` 一起定义，用于检查接口状态。

| 宏名称 | 作用 |
|--------|------|
| `IFF_UP` | 接口已启用 |
| `IFF_BROADCAST` | 支持广播 |
| `IFF_LOOPBACK` | 回环接口 |
| `IFF_POINTOPOINT` | 点对点接口 |
| `IFF_RUNNING` | 接口已运行 |
| `IFF_MULTICAST` | 支持多播 |
| `IFF_PROMISC` | 混杂模式 |
| `IFF_ALLMULTI` | 接收所有多播包 |
| `IFF_NOARP` | 不支持 ARP |
| `IFF_MASTER` | 负载均衡主设备 |
| `IFF_SLAVE` | 负载均衡从设备 |

### 3. ioctl 命令（通常在其他头文件，但常与 `<net/if.h>` 配合）

| 宏名称 | 作用 |
|--------|------|
| `SIOCGIFADDR` | 获取接口地址 |
| `SIOCGIFNETMASK` | 获取子网掩码 |
| `SIOCGIFBRDADDR` | 获取广播地址 |
| `SIOCGIFDSTADDR` | 获取点对点目的地址 |
| `SIOCGIFFLAGS` | 获取接口标志 |
| `SIOCSIFFLAGS` | 设置接口标志 |
| `SIOCGIFINDEX` | 获取接口索引 |

这些命令用于 `ioctl(fd, command, &ifreq)`。

---

## 四、类型定义

- `struct if_nameindex`：接口名称索引结构。
- `struct ifreq`：接口请求结构（通常用于 ioctl）。
- `if_nametoindex`、`if_indextoname`、`if_nameindex`、`if_freenameindex` 的函数原型。

---

## 五、示例用法

### 示例 1：使用 if_nameindex 打印所有接口
```c
#include <net/if.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    struct if_nameindex *iflist, *ifp;
    iflist = if_nameindex();
    if (iflist == NULL) {
        perror("if_nameindex");
        exit(EXIT_FAILURE);
    }
    for (ifp = iflist; ifp->if_index != 0 && ifp->if_name != NULL; ifp++) {
        printf("Name: %s, Index: %u\n", ifp->if_name, ifp->if_index);
    }
    if_freenameindex(iflist);
    return 0;
}
```

### 示例 2：使用 if_nametoindex 和 if_indextoname
```c
#include <net/if.h>
#include <stdio.h>
#include <string.h>

int main() {
    unsigned int idx = if_nametoindex("lo");
    if (idx == 0) {
        printf("lo not found\n");
    } else {
        printf("lo index: %u\n", idx);
        char name[IFNAMSIZ];
        if (if_indextoname(idx, name) != NULL) {
            printf("Index %u -> %s\n", idx, name);
        }
    }
    return 0;
}
```

---

## 六、实现原理

`if_nametoindex`、`if_indextoname`、`if_nameindex` 在 Linux 上通常通过 netlink 套接字（`NETLINK_ROUTE`）与内核通信，获取接口信息。`if_nameindex` 会动态分配内存存储所有接口信息。这些函数是 POSIX 标准的一部分（POSIX.1-2008），保证跨平台一致性（但 Windows 不支持）。

---

## 七、线程安全

- `if_nametoindex`：线程安全。
- `if_indextoname`：线程安全（写入用户缓冲区）。
- `if_nameindex`：线程安全（返回动态分配的内存，需配对 `if_freenameindex` 释放）。
- `if_freenameindex`：线程安全。

---
