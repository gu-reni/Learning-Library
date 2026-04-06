## `<netinet/tcp.h>` 头文件详解（Linux / POSIX）

`<netinet/tcp.h>` 是 Linux 及许多 POSIX 系统提供的头文件，用于定义 TCP 协议相关的套接字选项、结构体和常量。它主要配合 `setsockopt()` 和 `getsockopt()` 使用，允许应用程序精细控制 TCP 连接的行为（如禁用 Nagle 算法、设置保活参数、获取连接信息等）。

---

## 一、函数详解

`<netinet/tcp.h>` **不包含任何函数声明**。它仅提供宏、常量和一个可选的结构体 `tcp_info`。所有操作都通过通用套接字选项函数 `setsockopt()` 和 `getsockopt()` 完成，调用时 `level` 参数应指定为 `IPPROTO_TCP`。

**示例用法（设置 TCP_NODELAY）：**
```c
#include <netinet/tcp.h>
#include <sys/socket.h>

int flag = 1;
if (setsockopt(sockfd, IPPROTO_TCP, TCP_NODELAY, &flag, sizeof(flag)) == -1) {
    perror("setsockopt");
}
```

**示例用法（获取 TCP_INFO）：**
```c
#include <netinet/tcp.h>
#include <sys/socket.h>
#include <stdio.h>

struct tcp_info info;
socklen_t len = sizeof(info);
if (getsockopt(sockfd, IPPROTO_TCP, TCP_INFO, &info, &len) == 0) {
    printf("RTT = %u us\n", info.tcpi_rtt);
}
```

---

## 二、结构体详解

### struct tcp_info（Linux 特有）

**定义（简化）：**
```c
struct tcp_info {
    uint8_t  tcpi_state;        // TCP 状态（ESTABLISHED, SYN_SENT 等）
    uint8_t  tcpi_ca_state;     // 拥塞控制状态
    uint8_t  tcpi_retransmits;  // 重传次数
    uint8_t  tcpi_probes;       // 保活探测次数
    uint8_t  tcpi_backoff;      // 退避指数
    uint8_t  tcpi_options;      // 选项标志
    uint8_t  tcpi_snd_wscale;   // 发送窗口缩放因子
    uint8_t  tcpi_rcv_wscale;   // 接收窗口缩放因子

    uint32_t tcpi_rto;          // 重传超时（微秒）
    uint32_t tcpi_ato;          // 延迟确认超时（微秒）
    uint32_t tcpi_snd_mss;      // 发送 MSS
    uint32_t tcpi_rcv_mss;      // 接收 MSS

    uint32_t tcpi_unacked;      // 未确认的数据包数
    uint32_t tcpi_sacked;       // 已接收 SACK 的数据包数
    uint32_t tcpi_lost;         // 丢失的数据包数
    uint32_t tcpi_retrans;      // 重传的数据包数
    uint32_t tcpi_fackets;      // FACK 计数（已废弃）

    uint32_t tcpi_last_data_sent;      // 最后发送数据的时间
    uint32_t tcpi_last_ack_sent;       // 最后发送 ACK 的时间
    uint32_t tcpi_last_data_recv;      // 最后接收数据的时间
    uint32_t tcpi_last_ack_recv;       // 最后接收 ACK 的时间

    uint32_t tcpi_pmtu;         // 路径 MTU
    uint32_t tcpi_rcv_ssthresh; // 接收端慢启动阈值
    uint32_t tcpi_rtt;          // 平滑 RTT（微秒）
    uint32_t tcpi_rttvar;       // RTT 方差（微秒）
    uint32_t tcpi_snd_ssthresh; // 发送端慢启动阈值
    uint32_t tcpi_snd_cwnd;     // 发送拥塞窗口
    uint32_t tcpi_advmss;       // 通告的 MSS
    uint32_t tcpi_reordering;   // 重排序距离

    uint32_t tcpi_rcv_rtt;      // 接收端 RTT 估计（微秒）
    uint32_t tcpi_rcv_space;    // 接收端缓冲区空间

    uint32_t tcpi_total_retrans; // 总重传次数（可能扩展）
    // ... 更多字段（随内核版本增加）
};
```

**作用：** 通过 `TCP_INFO` 选项获取 TCP 连接的内部状态和统计信息，常用于诊断、性能监控。

**成员详解：**
- `tcpi_state`：TCP 状态（如 `TCP_ESTABLISHED`、`TCP_SYN_SENT`），可参考 `<netinet/tcp.h>` 中的 `TCP_ESTABLISHED` 等宏（通常定义在 `<netinet/tcp.h>` 或 `<linux/tcp.h>`）。
- `tcpi_rtt`：平滑往返时间（微秒），是监控网络延迟的重要指标。
- `tcpi_snd_cwnd`：当前拥塞窗口大小（单位：MSS）。
- `tcpi_unacked`：尚未收到 ACK 的数据包数。
- 更多字段参考 Linux 内核文档。

**使用注意事项：**
- `struct tcp_info` 是 Linux 特有的，非 POSIX 标准，移植性较差。
- 其大小和字段可能随内核版本变化，建议使用 `getsockopt` 时动态获取实际长度。

---

## 三、宏定义详解

### 1. TCP 套接字选项（optname）

| 宏名称 | 作用 | 数据类型 | 说明 |
|--------|------|----------|------|
| `TCP_NODELAY` | 禁用 Nagle 算法 | `int` | 设置为 1 时禁止 Nagle 算法，小数据包立即发送，减少延迟但可能增加网络小包数量。 |
| `TCP_MAXSEG` | 设置/获取最大段大小（MSS） | `int` | 限制 TCP 段的最大大小，影响分片。通常由内核自动选择。 |
| `TCP_CORK` | 启用/禁用 Corking（塞子） | `int` | 启用后发送的数据会累积直到“满”或禁用，提高吞吐量。与 `TCP_NODELAY` 互斥。 |
| `TCP_KEEPIDLE` | 保活空闲时间（秒） | `int` | 连接空闲多久后开始发送保活探测。Linux 上默认 7200 秒。 |
| `TCP_KEEPINTVL` | 保活探测间隔（秒） | `int` | 两次保活探测之间的间隔。Linux 上默认 75 秒。 |
| `TCP_KEEPCNT` | 保活探测次数 | `int` | 放弃连接前发送的最大保活探测次数。Linux 上默认 9 次。 |
| `TCP_NOTSENT_LOWAT` | 设置未发送数据的低水位标记 | `int` | 当写入队列低于此值时，`select`/`poll` 指示可写。 |
| `TCP_QUICKACK` | 启用/禁用快速确认 | `int` | 启用后立即发送 ACK，禁用后延迟确认。 |
| `TCP_USER_TIMEOUT` | 用户超时（毫秒） | `unsigned int` | 在未确认数据超时后强制关闭连接。 |
| `TCP_INFO` | 获取 TCP 连接信息 | `struct tcp_info` | 只读，用于获取内核维护的 TCP 统计信息。 |
| `TCP_CONGESTION` | 获取/设置拥塞控制算法名称 | `char[]` | 例如 `"cubic"`、`"reno"`。 |
| `TCP_DEFER_ACCEPT` | 延迟 accept 直到有数据 | `int` | 监听套接字选项，仅在接收到第一个数据后才完成连接。 |
| `TCP_SYNCNT` | SYN 重传次数 | `int` | 设置 SYN 报文的重传次数。 |
| `TCP_WINDOW_CLAMP` | 限制接收窗口大小 | `int` | 限制通告的接收窗口最大值。 |

**示例（设置保活参数）：**
```c
int keepidle = 60;   // 60 秒空闲开始探测
int keepintvl = 10;  // 探测间隔 10 秒
int keepcnt = 3;     // 最多 3 次探测
setsockopt(sockfd, IPPROTO_TCP, TCP_KEEPIDLE, &keepidle, sizeof(keepidle));
setsockopt(sockfd, IPPROTO_TCP, TCP_KEEPINTVL, &keepintvl, sizeof(keepintvl));
setsockopt(sockfd, IPPROTO_TCP, TCP_KEEPCNT, &keepcnt, sizeof(keepcnt));
```

---

### 2. 其他常量

| 宏名称 | 作用 | 说明 |
|--------|------|------|
| `TCP_ESTABLISHED` | TCP 状态：已建立连接 | 可能定义在 `<netinet/tcp.h>` 中，用于 `tcp_info.tcpi_state`。 |
| `TCP_SYN_SENT` | TCP 状态：已发送 SYN | 同上。 |
| `TCP_SYN_RECV` | TCP 状态：已收到 SYN | 同上。 |
| `TCP_FIN_WAIT1`、`TCP_FIN_WAIT2` 等 | 其他 TCP 状态 | 同上。 |

注意：这些状态宏有时定义在 `<linux/tcp.h>` 中，但 `<netinet/tcp.h>` 可能包含它们。

---

## 四、类型定义

- `struct tcp_info`：TCP 信息结构体（Linux 特有）。
- 没有其他独立类型。

---

## 五、模板声明

`<netinet/tcp.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

## 六、实现原理

- `TCP_NODELAY`：禁用 Nagle 算法。Nagle 算法将小数据包合并发送以减少网络拥塞，但对延迟敏感应用（如游戏、实时通信）不利。禁用后，每次 `send` 调用都会立即发送 TCP 段。
- `TCP_CORK`：与 `TCP_NODELAY` 相反，强制累积数据直到显式禁用，可减少小包数量，提高吞吐量。
- 保活选项（`TCP_KEEPIDLE`、`TCP_KEEPINTVL`、`TCP_KEEPCNT`）：内核通过定时器检查连接是否存活，在空闲时间达到 `keepidle` 后开始发送探测报文，探测间隔 `keepintvl`，探测次数达到 `keepcnt` 后断开连接。
- `TCP_INFO`：内核将 TCP 控制块中的内部统计信息复制到用户空间提供的 `tcp_info` 结构体中，用于调试和监控。

---

## 七、线程安全

`<netinet/tcp.h>` 只定义宏和结构体，没有函数。使用 `setsockopt`/`getsockopt` 时，这些函数本身是线程安全的，但并发修改同一套接字的相同选项可能产生竞争（例如多个线程同时设置 `TCP_NODELAY` 或读取 `TCP_INFO` 不会有内核冲突，但最终值可能取决于最后一个写入）。

---
