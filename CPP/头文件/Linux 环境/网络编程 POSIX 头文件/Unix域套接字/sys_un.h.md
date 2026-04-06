## `<sys/un.h>` 头文件详解（Linux / POSIX）

`<sys/un.h>` 是 POSIX 标准定义的头文件，用于支持 **Unix 域套接字**（Unix domain socket）。Unix 域套接字允许同一台主机上的进程之间进行高效通信，它使用文件系统路径名作为地址，而不是 IP 地址和端口号。该头文件主要定义了 `struct sockaddr_un` 结构体及相关常量，配合 `<sys/socket.h>` 中的套接字函数（`socket`、`bind`、`connect` 等）使用。

---

## 一、函数详解

`<sys/un.h>` **不包含任何函数声明**。Unix 域套接字的创建、绑定、连接等操作使用 `<sys/socket.h>` 中的通用套接字函数（如 `socket`、`bind`、`connect`、`accept`、`send`、`recv` 等），只需将 `domain` 参数指定为 `AF_UNIX` 或 `AF_LOCAL`，并将地址结构指定为 `struct sockaddr_un`。

**示例用法（服务器端）：**
```c
#include <sys/socket.h>
#include <sys/un.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
    struct sockaddr_un addr;
    memset(&addr, 0, sizeof(addr));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, "/tmp/mysocket", sizeof(addr.sun_path) - 1);
    unlink("/tmp/mysocket");  // 移除可能存在的旧文件
    bind(sockfd, (struct sockaddr*)&addr, sizeof(addr));
    listen(sockfd, 5);
    // ... accept, read, write, close
}
```

**客户端示例：**
```c
int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
struct sockaddr_un addr;
memset(&addr, 0, sizeof(addr));
addr.sun_family = AF_UNIX;
strncpy(addr.sun_path, "/tmp/mysocket", sizeof(addr.sun_path) - 1);
connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
```

---

## 二、结构体详解

### struct sockaddr_un

**定义：**
```c
struct sockaddr_un {
    sa_family_t sun_family;               // 地址族，必须为 AF_UNIX 或 AF_LOCAL
    char        sun_path[108];            // 套接字文件路径（以空字符结尾）
};
```

**作用：** 用于 Unix 域套接字的地址结构，指定一个文件系统路径作为套接字的地址。

**成员详解：**
- `sun_family`：地址族，必须设置为 `AF_UNIX` 或 `AF_LOCAL`（两者等价）。
- `sun_path`：套接字文件的路径（绝对路径或相对路径），以空字符 `'\0'` 结尾。长度最大为 108 字节（不同系统可能略有差异，Linux 上为 108，BSD 通常为 104）。

**使用注意事项：**
- 使用 `bind()` 时，会在文件系统中创建一个特殊文件（套接字文件）。绑定前建议使用 `unlink()` 删除可能存在的旧文件，否则 `bind` 可能失败（除非文件已被其他进程使用，但通常不会）。
- 套接字文件的权限由 `umask` 和 `bind` 时的文件创建模式决定，可通过 `chmod` 修改。
- 通信结束后，可以手动删除套接字文件（使用 `unlink()`），否则文件会保留在文件系统中。
- `sun_path` 的长度限制可能导致路径名过长时出错，需确保路径长度不超过 `sizeof(sun_path) - 1`。
- 路径名可以是**抽象套接字地址**（Linux 特有）：将 `sun_path` 的第一个字节设为 `'\0'`，后续字节作为抽象名称（不占用文件系统）。此时 `bind` 不会创建文件系统实体。

---

## 三、宏定义详解

### 1. 地址族常量

| 宏名称 | 作用 |
|--------|------|
| `AF_UNIX` | Unix 域套接字地址族 |
| `AF_LOCAL` | 与 `AF_UNIX` 相同，POSIX 标准名称 |

### 2. 路径长度相关

| 宏名称 | 作用 |
|--------|------|
| `UNIX_PATH_MAX` | 某些系统定义（非 POSIX），表示 `sun_path` 的最大长度。Linux 中通常不提供此宏，直接使用 `sizeof(((struct sockaddr_un*)0)->sun_path)`。 |

---

## 四、类型定义

- `sa_family_t`：地址族类型（定义在 `<sys/socket.h>`）。
- `struct sockaddr_un` 如上。

---

## 五、模板声明

`<sys/un.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

## 六、实现原理

Unix 域套接字在内核中实现，不经过网络协议栈，使用文件系统路径名进行寻址。通信时数据直接在内核中复制，比 TCP 环回（127.0.0.1）更高效。套接字类型可以是 `SOCK_STREAM`（面向连接，可靠，类似 TCP）或 `SOCK_DGRAM`（无连接，可靠，不丢失顺序，类似 UDP 但可靠）。`struct sockaddr_un` 中的路径名用于标识套接字端点。绑定到路径名时，内核会在文件系统中创建该特殊文件，后续连接和通信通过文件系统查找。

---

## 七、线程安全

`<sys/un.h>` 仅定义结构体和常量，没有函数，因此无线程安全问题。但使用 Unix 域套接字的函数（如 `socket`、`bind`、`connect`）的线程安全性与 `<sys/socket.h>` 中描述一致，均为线程安全。需要注意的是，对文件系统路径的操作（如 `unlink`）可能需要额外同步。

---
