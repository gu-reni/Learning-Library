# HTTP/HTTPS-牛客面经八股

> 来源：牛客网  |  共 16 题

## 1. HTTP/1.0、HTTP/1.1、HTTP/2.0、HTTP/3.0 的区别？
一、 HTTP/1.0 与 HTTP/2.0 的区别

#### 一句话总结

 HTTP/1.0 是“单车道、纯文字”的老国道；HTTP/2.0 则是“多车道、全双工、二进制”的高速公路。

#### 关键区别清单

 - 连接复用 • HTTP/1.0：一次请求一次 TCP 连接（缺省无 Keep‑Alive） • HTTP/2.0：一个 TCP 连接可同时承载多条流（Stream Multiplexing），节省握手开销

 - 报文格式 • HTTP/1.0：纯文本首部 + 空行 + 实体；解析靠换行符 • HTTP/2.0：全部拆成二进制帧（Frame），首部、数据、优先级都是帧

 - 首部压缩 • HTTP/1.0：首部随每个请求重复发送 • HTTP/2.0：HPACK 动态字典压缩（基于静态表+动态表），对重复头部可压缩至原体积的 20%-30%（如 Cookie 等高频字段），但对唯一头部压缩率较低

 - 并发阻塞 • HTTP/1.0：浏览器为同一域名开 2–6 条 TCP，仍受队头阻塞限制 • HTTP/2.0：同连接内多流并发，无应用层队头阻塞，但 TCP 层丢包仍会导致整体延迟（如单个 TCP 包丢失需重传所有流数据）

 - 服务器主动推送 • HTTP/1.0：只有客户端发起 • HTTP/2.0：Server Push 可提前推送关键资源（如 CSS/JS），但需客户端通过Accept-Push-Policy头接受，否则可能浪费带宽（如重复推送已缓存资源）

 - 优先级与流量控制 • HTTP/1.0：无 • HTTP/2.0：每个流可指定权重，且有窗口大小做流量控制

#### 形象类比

 想象你在老国道（HTTP/1.0）上开车：一条车道、一辆车（请求）走完才能放下一辆，还得反复交停车费（TCP/3 次握手）。

换到 HTTP/2.0 就像上高速：多车道并行，收费站变成 ETC（首部压缩），而且高速管理局（服务器）还能把你常用的加油卡（静态资源）直接递到车窗（Server Push）。

#### 影响与适用场景

 • 多资源的小页面差异不大，但动辄几十、上百资源的大型网页能显著减少握手与首部流量。

• 移动网络高时延场景能收到明显收益。

• 如果服务端与 CDN 已全面支持 HTTP/2，则默认启用，基本无兼容性成本。

---

 二、HTTP/2.0 与 HTTP/3.0 的区别

#### 一句话总结

 HTTP/2.0 是“跑在 TCP 上的多股车道高速”；HTTP/3.0 把底层换成 QUIC（UDP），“修了一条能换轮胎、永不堵车的新高铁”。

#### 关键区别清单

 - 传输层协议 • HTTP/2.0：TCP + TLS • HTTP/3.0：QUIC（基于 UDP，集成 TLS 1.3）

 - 连接与加密握手 • HTTP/2.0：TCP 3 次握手 + TLS 1.2/1.3 = 至少 1–2 RTT • HTTP/3.0：QUIC 首次 1-RTT，复用密钥 0-RTT，链路迁移无需重连

 - 队头阻塞 (Head-of-Line Blocking) • HTTP/2.0：TCP 级别的包丢失会堵住同连接内所有流 • HTTP/3.0：QUIC 每条流独立排序，丢包仅影响该流；同时支持前向纠错（FEC），通过冗余数据包恢复少量丢包，减少重传延迟

 - 连接迁移 • TCP 连接与四元组强绑定，IP/端口一变就得重建 • QUIC 用 Connection ID，移动网络 IP 变，连接仍在，类似‘快递单号’不变，即使收件地址（IP）变化，包裹（数据）仍能送达

 - 加密范围 • HTTP/2.0：TLS 保护的是“TCP 载荷” • HTTP/3.0：QUIC 把首部 + 流控 + ACK 都加密，带来更难的中间盒干预

 - 部署现状 • HTTP/2.0：浏览器 & CDN 已全面普及 • HTTP/3.0：Chrome/Firefox/Edge 已默认开启，但后端和防火墙需支持 UDP/QUIC

#### 形象类比

 把多流 HTTP/2 放 TCP，就像把多节车厢的子弹列车仍旧跑在老铁轨：轨道坏一截，全车减速。

HTTP/3/QUIC 相当于修了一条专用磁悬浮：每节车厢有独立悬浮系统，哪怕前车厢“抖”一下，后车厢照常飞；而且乘客中途换座（IP 切换）也不影响整车运行。

#### 影响与适用场景

 • 移动端频繁切网（蜂窝 / Wi‑Fi）的应用，HTTP/3 能减少重连卡顿。

• 高丢包、长 RTT 场景（跨洋、卫星网络）下，QUIC 的流独立重传优势明显。

• 服务端需开放 UDP 443 端口，或升级到支持 QUIC 的网关/CDN。

---

 三、HTTP/1.1 与 HTTP/2.0 的区别

 （很多面试官会把 1.1 与 2.0 的对比当重点；1.0 与 1.1 在 Keep‑Alive、Host 头、缓存语义等方面也有差异，但题目只要求 1.1 vs 2.0，下面直说。）

#### 一句话总结

 HTTP/1.1 是“一根水管 + 排队取水”；HTTP/2.0 把水管加粗成“多股细流并发 + 加压泵 + 伺服阀”。

#### 关键区别清单

 - 长连接与并发 • HTTP/1.1：默认 Keep‑Alive，但一条 TCP 连接同一时刻只能并发 1 个响应；尝试用 Pipelining 也因队头阻塞基本被禁用 • HTTP/2.0：多路复用通过二进制分帧实现，单连接可并行数百个流（受窗口大小和服务器处理能力限制），且流优先级可动态调整

 - 首部与性能 • 1.1：首部重复＋明文，Cookie/UA 动辄几百字节 • 2.0：HPACK 压缩 + 静态/动态字典，握手后首部几乎只发索引

 - 数据格式 • 1.1：纯文本，调试方便但解析慢 • 2.0：二进制帧，帧头固定 9 字节，可直接映射到状态机

 - 服务器推送 • 1.1：无 • 2.0：Server Push，可减少关键资源 1 个 RTT

 - 流量控制与优先级 • 1.1：全凭 TCP（粗粒度） • 2.0：应用层再加一层窗口 + 权重，细粒度调度

 - 安全部署 • 1.1：HTTPS 为可选 • 2.0：RFC 7540 规定浏览器仅支持 h2 over TLS（加密），明文仅限非浏览器场景（如 HTTP/2 over TCP 未广泛实现）

#### 形象类比

 HTTP/1.1 像一台只能单线程打印的打印机：虽然可以装更大纸盒（Keep‑Alive），但一次只能打印一页；

HTTP/2.0 就像多功能批处理打印机：同一根电源线（TCP）里，多份文档同时排队、压缩传输墨粉（首部），还能让打印机自己预热下一页（Server Push）。

#### 影响与适用场景

 • Web 首屏速度要求高、资源碎片多（上百静态文件）的站点，升级到 HTTP/2 后可观地减少 TTFB。

• 若业务已有「域名分片」、Sprite 合图、内联资源等 1.1 时代的“性能黑魔法”，迁移到 2.0 后应逐步取消，以免与多路复用相冲突。

---

#### 总结口诀

 1.1 → 2.0：文本变二进制、串行变并行、重复首部变压缩、客户端请求变“可被推送”。

2.0 → 3.0：TCP 变 QUIC、握手更快、丢包隔离、连接可迁移。

0→1→2→3 就像“修国道 → 加车道 → 上高速 → 换高铁”。

---

## 2. HTTP 常见状态码有哪些？
下面按 **1xx / 2xx / 3xx / 4xx / 5xx** 五大类，列出最常被问到的核心状态码，并配一句“场景口令”帮助记忆。

#### 1xx　信息性（握手阶段的“收到！”）

| 代码 | 口令 | 含义与场景 |
| --- | --- | --- |
| 100 Continue | “继续灌” | 客户端可先发送请求首部（含Expect: 100-continue），收到 100 后再发包体（RFC 7231），适用于大文件分块上传。 |
| 101 Switching Protocols | “换频道” | WebSocket 升级、HTTP/1.1 → HTTP/2 时常见。 |

---

#### 2xx　成功（服务器“OK！”）

| 代码 | 口令 | 含义与典型用法 |
| --- | --- | --- |
| 200 OK | “一切安好” | 最常见；GET/POST 都可能返回。 |
| 201 Created | “新建完成” | POST /users 创建用户；响应中一般给 Location 头。 |
| 202 Accepted | “我收下先” | 异步任务排队，如上传转码。 |
| 204 No Content | “办完了，没料” | 删除成功、不需要返回体。 |
| 206 Partial Content | “分段寄” | 断点续传，Range: bytes=... |

---

#### 3xx　重定向（服务器“去那边”）

| 代码 | 口令 | 含义与差异点 |
| --- | --- | --- |
| 301 Moved Permanently | “搬家永久” | 浏览器会缓存；SEO 友好。 |
| 302 Found | “临时搬家” | 老版浏览器照样改成 GET；早期最滥用。 |
| 303 See Other | “换 GET 拿” | POST 后重定向到 GET 资源（支付回跳常见）。 |
| 304 Not Modified | “缓存命中” | If‑None‑Match / If‑Modified‑Since 协商缓存。 |
| 307 Temporary Redirect | “临时搬家但保持方法” | 强制客户端使用原请求方法（如 POST 仍为 POST），与 302 的兼容性差异需注意旧代理行为。 |
| 308 Permanent Redirect | “永久搬家且保持方法” | 301 + 保留方法；HTTP/2 推广。 |

---

#### 4xx　客户端错误（服务器“你错了”）

| 代码 | 口令 | 典型场景 |
| --- | --- | --- |
| 400 Bad Request | “报文烂了” | JSON 语法错、请求头过大等。 |
| 401 Unauthorized | “先登录” | 缺失/失效 Token；配合 WWW‑Authenticate。 |
| 403 Forbidden | “我认得你，但不给” | 鉴权通过但无权限；IP 黑名单。 |
| 404 Not Found | “地址错了” | 经典“404 页面”。 |
| 405 Method Not Allowed | “动手方式错” | PUT 到只允许 GET 的 URL。 |
| 408 Request Timeout | “你太慢了” | 客户端未在时限内发完整请求。 |
| 409 Conflict | “版本冲突” | 编辑冲突、资源重复创建。 |
| 410 Gone | “永别了” | 资源永久删除，不会再有。 |
| 413 Payload Too Large | “包太大” | 上传超过限制。 |
| 415 Unsupported Media Type | “格式不懂” | Content‑Type 不被接受。 |
| 429 Too Many Requests | “别刷了” | 限流/防刷必备。 |

---

#### 5xx　服务端错误（服务器“我挂了”）

| 代码 | 口令 | 说明与排查方向 |
| --- | --- | --- |
| 500 Internal Server Error | “后台炸了” | 日志第一时间看 stack trace。 |
| 501 Not Implemented | “功能未上” | 服务器不支持当前方法。 |
| 502 Bad Gateway | “网关炸了” | 上游服务无响应（如超时、协议错误）或反向代理配置错误（如 DNS 解析失败）。 |
| 503 Service Unavailable | “临时停业” | 服务暂时不可用（如维护、限流），需通过Retry-After头告知客户端重试时间。 |
| 504 Gateway Timeout | “上游超时” | 反向代理等待后端 > timeout。 |
| 505 HTTP Version Not Supported | “版本太古” | 服务器不支持请求里的 HTTP 版本。 |

---

#### 一张图背口诀

```cpp
1xx 我收到了；2xx 全搞定； 
3xx 去别处；4xx 你先改； 
5xx 我先修。
```

---

#### 面试加分 Tips

 - 记住 **“401 要带 WWW‑Authenticate，403 则不带”**——很多人会混。

 - 当说到 **断点续传**，要提 206 + Range/Content‑Range。

 - 搭 API 时常用 **201 + Location** 返回新资源地址，胜过一股脑儿给 200。

## 3. HTTP 请求头中到底包含什么？
#### 

#### 一句话总结

 面试官常问：“HTTP 请求头会带哪些东西？”——实质上，它们就是客户端在「信封」上贴的各种标签，用来告诉服务器**“我是谁、要什么，以及怎么传”**。下面按层次展开说明。

#### 整体结构

```cpp
<请求行> → GET /index.html HTTP/1.1
<首部字段 1> → Host: www.example.com
<首部字段 2> → User-Agent: Mozilla/5.0 ...
...
<空行> → \r\n
<可选请求体> → { "name": "foo" }
```

 • 请求行(Start-Line)：方法 + 请求目标 + 协议版本

• 首部字段(Header)：一行一个 “Key: Value”

• 空行(CRLF)：标记头结束

• 请求体(Body)：如 JSON、表单、文件等

 HTTP/2/3 中，原请求行元素转为**伪头字段**（:method、:path等），普通首部字段（如User-Agent）仍保留。

## 4. HTTP 是基于 TCP 还是 UDP？
HTTP 依赖的传输层协议可以分两种情况来回答——

 - “传统”HTTP（HTTP/0.9、1.0、1.1，以及绝大多数线上 HTTP/2） 基本都跑在 **TCP** 之上。TCP 提供了： • 可靠传输（重传、排序） • 全双工字节流 • 拥塞 / 流量控制 这正好满足浏览器—服务器之间“要把一段报文按顺序、完整送到”的需求。

 - “新一代”HTTP/3 把底层换成了 **QUIC**，而 QUIC 又是“跑在 **UDP** 里的类 TCP 协议”。 简单记忆：HTTP/3 = HTTP/2 多路复用语义 + QUIC(UDP) 传输。原因有： • 避免 TCP 层队头阻塞（QUIC 的每条流独立确认、丢包互不影响）。 • 把 TLS 1.3 与握手合并，首包即可加密，减少 RTT。 • 通过 Connection ID 支持“IP/端口变了也能续传”（移动网络场景）。

#### 一句话总结

 “HTTP/1.x 和主流 HTTP/2 都基于 TCP；而 **HTTP/3 基于 UDP（具体说是基于 UDP 的 QUIC 协议）**。所以今天两种都存在，看版本而定。”

## 5. HTTP 常见字段有哪些？
下面给出一份“常考 HTTP 头字段 32 强速查表”。为了方便记忆，我按「功用」把它们分成 8 组，并在每组前加一句“口令式”提示。

### 0. 一句话总览

 HTTP 头（Header）≈ 信封上的各种标签：

• 谁寄的？（身份/定位）

• 要寄什么？（实体元信息）

• 怎样寄？（缓存/连接/分片）

• 给谁看？（安全/CORS）

• 途中还能改道吗？（重定向/协商）

### 1. 身份与定位 ——「这封信从哪儿来，要寄到哪儿去？」

| 字段 | 说明 | 面试高频考点 |
| --- | --- | --- |
| Host | 请求主机名 + 端口 | 虚拟主机必需；HTTP/1.1 强制要求 |
| Origin | 发起跨域请求时的源 | CORS、CSRF 防御 |
| Referer（标准字段名） | 上一个页面 URL | SEO、流量统计、可通过′same-origin′隐藏 |
| User-Agent | 浏览器/客户端标识 | UA 嗅探、移动端自适应 |

### 2. 实体元信息 ——「这包裹里装的啥？」

| 字段 | 说明 | 典型值 |
| --- | --- | --- |
| Content-Type | 实体 MIME | text/html; charset=utf-8 |
| Content-Length | 字节大小 | 必须是十进制 |
| Content-Encoding | 压缩算法 | gzip / br / deflate |
| Content-Language | 实体语言 | zh-CN / en-US |
| Content-Disposition | 下载文件名（inline显示、attachment强制下载）、防 MIME 类型嗅探攻击（需配合X-Content-Type-Options: nosniff） | attachment; filename="a.pdf" |
| Last-Modified | 最后修改时间 | 协商缓存 |
| ETag | 实体指纹 | "abc123"，强/弱校验 |
| Content-Range | 响应分段 | bytes 0-99/300 |

### 3. 缓存 & 协商 ——「快递能否存在驿站？」

| 字段 | 作用 | 关键语法 |
| --- | --- | --- |
| Cache-Control | 最核心缓存策略 | max-age, no-cache, must-revalidate |
| Expires | 绝对过期时间 | HTTP/1.0 遗产，受 Cache-Control 覆盖 |
| Pragma | 旧版 no-cache | 主要兼容 HTTP/1.0 |
| If-None-Match | 条件请求（ETag） | 服务端 200/304 |
| If-Modified-Since | 条件请求（时间） | 配合 Last-Modified |
| Vary | 缓存维度 | Vary: Accept-Encoding |

### 4. 连接 & 传输 ——「快递路线怎么走？」

| 字段 | 说明 | 面试切入点 |
| --- | --- | --- |
| Connection | 连接选项 / Keep-Alive | HTTP/1.x 中常设keep-alive |
| Upgrade | 协议升级 | WebSocket：Upgrade: websocket |
| Transfer-Encoding | 分块编码 | chunked时可省 Content-Length |
| TE | 指明接受的传输编码 | 与 Transfer-Encoding 区分 |
| Range / Accept-Ranges | 断点续传 | 客户端 / 服务器各自使用 |

### 5. 重定向 & 路由 ——「快递途中要不要中转？」

| 字段 | 说明 |
| --- | --- |
| Location | 3xx 响应新地址 |
| Content-Location | 资源实际地址（弱化版 Location） |
| Allow | 405 时列出合法方法 |
| Link | 预加载 / 资源提示 (HTTP/2 Push) |

### 6. 安全 & 认证 ——「确认身份、贴防拆胶带」

| 字段 | 作用 | 小贴士 |
| --- | --- | --- |
| Authorization | 客户端凭据 | Basic / Bearer / Digest |
| WWW-Authenticate | 401 返回的挑战 | Basic realm="xxx" |
| Cookie | 会话凭证 | 发自客户端 |
| Set-Cookie | 服务端写 Cookie | HttpOnly / Secure / SameSite |
| Strict-Transport-Security | HSTS | max-age=...; includeSubDomains |
| Content-Security-Policy | XSS 防护大杀器 | default-src 'self' |
| X-Frame-Options | 点击劫持防护 | DENY / SAMEORIGIN |
| X-XSS-Protection | 老版浏览器 XSS 过滤器 | 0/1; mode=block |

### 7. CORS 访问控制 ——「跨国快递要海关盖章」

| 字段（响应端居多） | 说明 |
| --- | --- |
| Access-Control-Allow-Origin | 允许的源 |
| Access-Control-Allow-Methods | 允许的方法列表 |
| Access-Control-Allow-Headers | 允许的自定义请求头 |
| Access-Control-Allow-Credentials | 是否允许携带 Cookie |
| Access-Control-Max-Age | 预检缓存时长 |
| Access-Control-Expose-Headers | 客户端可读取的额外响应头 |

### 8. 服务端信息 ——「快递公司与寄件日期」

| 字段 | 说明 |
| --- | --- |
| Server | 服务端软硬件标识 |
| Date | 响应生成时间 |
| Retry-After | 503/429 告知多久后再试 |
| Via | 多层代理链路记录 |

### 面试口诀（30 秒复盘）

 - Host / UA / Referer / Origin ——「我是谁，从哪来」。

 - Content-* / ETag / Range ——「我带了啥，拆第几段」。

 - Cache-Control / If-* ——「能不能寄放驿站」。

 - Location / Upgrade ——「半路改道、换轨道」。

 - Cookie / Auth / STS / CSP ——「验身份，贴封条」。

 - CORS 六兄弟 ——「过海关要盖章」。

 - Connection / Transfer-Encoding ——「走陆路还是分批邮」。

## 6. 谈谈 HTTP 的缓存机制，服务器如何判断缓存是否过期？
#### 

#### 一句话先回答

 HTTP 缓存＝“先看本地冰箱够不够新鲜(强缓存)；不够再打电话问超市要不要换(协商缓存)”。服务器判断是否过期靠 **Expires / Cache-Control 给出的“保质期”**，以及 **ETag / Last-Modified 做“指纹校验”**。

---

### 1. 参与者与两条主线

```cpp
浏览器 ←→ 代理 / CDN ←→ 源服务器
```

 • 浏览器 / 代理 ＝ **缓存层**：决定“要不要直接用旧货”。

• 源服务器 ：决定“旧货还能不能吃”。

 两条主线

 - **强缓存（freshness caching）** ‑ 缓存层直接用本地副本，不请求服务器，0 RTT。

 - **协商缓存（validation caching）** ‑ 缓存层带条件字段去问服务器，若未变返回 304，省体积但仍 1 RTT。

---

### 2. 强缓存：靠“保质期”判断

| 字段 | 作用 | 优先级 |
| --- | --- | --- |
| Cache-Control: max-age=60 | 剩余秒数 | ① |
| Cache-Control: s-maxage=300 | 专给共享缓存(CDN)的保质期 | 同级 |
| Expires: Fri, 21 Jul 2023 07:28:00 GMT | 绝对时间（HTTP/1.0 遗产） | ②，仅在无 max-age 时用 |

 判断流程（浏览器 / CDN 内部算法）

```cpp
freshness_life = # 以下取其一
 Cache-Control.max-age
 | s-maxage (若在共享缓存)
 | Expires-Date 差值
 | Heuristic 10% 规则 # 都没有时的猜测

current_age = Age 头 + (现在 - 缓存时间戳)
if current_age < freshness_life:
 直接命中
else:
 进入协商缓存
```

 Age 头由上一级代理写入，表示对象已在链路中存活多久。

---

### 3. 协商缓存：靠“指纹”校验

| 响应头 | 客户端对应请求头 | 用法 & 颗粒度 |
| --- | --- | --- |
| ETag: "v2-abcd" | If-None-Match | 强对比，字节级；最佳实践 |
| Last-Modified: Mon, 20 Jul 2023 06:00:00 GMT | If-Modified-Since | 秒级时间戳；易受时钟误差 |

 - **缓存层组装条件请求** ```cpp GET /logo.png HTTP/1.1 If-None-Match: "v2-abcd" If-Modified-Since: Mon, 20 Jul 2023 06:00:00 GMT ```

 - **源服务器判断** ```cpp if (资源指纹 == 请求头值): 304 Not Modified else: 200 OK + 新资源体 + 新 ETag/Last-Modified ```

 - **缓存层更新本地副本 + 重设保质期**。

---

### 4. 服务器如何生成“保质期”与“指纹”？

 - 保质期（Freshness） • **静态资源**：根据业务需要或构建哈希名做Cache-Control: max-age=31536000, immutable。 • **动态 API**：视业务一致性要求，可能给max-age=0, private, must-revalidate。 • CDN 针对动态路径可额外用s-maxage做边缘缓存。

 - 指纹（Validator） • **ETag**：文件 MD5、Git commit、对象版本号。 • **Last-Modified**：文件系统 mtime、数据库updated_at字段。 • 若资源更新频率 < 1 秒，可能只依赖 ETag；否则可二者并用。

---

### 5. 常见指令语义速述

| 指令 | 说明 |
| --- | --- |
| public / private | 是否允许代理缓存 |
| no-cache | 仍可缓存，但必须协商后才能用 |
| no-store | 禁止任何层存储（敏感信息） |
| must-revalidate | 到期后必须去源服务器问 |
| stale-while-revalidate=30 | 过期 ≤30s 期间仍可用旧副本并异步刷新 |
| stale-if-error=3600 | 源站 5xx 时，旧副本可再撑 1 小时 |

---

### 6. 整体时序图

```cpp
(首访)
C → S : GET /a.js
S → C : 200 OK
 Cache-Control: max-age=600
 ETag: "hash123"
 Date: t0

(t0 + 100s，再次访问)
C (本地 age=100 < 600) → 直接命中

(t0 + 700s，再次访问)
C : GET /a.js
 If-None-Match: "hash123"
S : 304 Not Modified
 Cache-Control: max-age=600
 ETag: "hash123"
C : 更新缓存时间戳，再走 600s
```

---

### 7. 面试常见陷阱

 - **“no-cache ≠ 不缓存”**：它是“可存但要每次协商”。真不缓存用no-store。

 - **“max-age=0 仍可协商”**：表示立刻过期，但不等于禁止缓存。

 - **动态接口也能缓存**：GraphQL 或 REST 只要幂等，也可用 ETag + max-age。

 - **304 也消耗 RTT**：在移动弱网下，更倾向给足过期时间 + 静态资源指纹。

---

#### 8. 记忆口诀

```cpp
强缓存看“保质期”
 max-age / Expires / s-maxage

协商缓存拼“指纹”
 ETag / Last-Modified
```

 掌握「保质期 + 指纹」机制，就能准确阐述“HTTP 是如何判断缓存是否过期”的完整流程。

## 7. HTTP 长连接 vs. 短连接的区别是？
一句话总结：
 
“短连接”= **一次请求一次 TCP**；“长连接”= **一个 TCP 多次请求**。前者像一次性纸杯，用完即丢；后者是保温杯，喝完还能续水。

---

#### 1. 基本概念

| 名称 | 关键特征 | HTTP/1.0 默认 | HTTP/1.1 默认 |
| --- | --- | --- | --- |
| 短连接 (non-persistent) | 请求–响应结束就FIN | ✅ | ❌ |
| 长连接 (persistent / Keep-Alive) | TCP 连接可复用多次 | 需显式Connection: keep-alive | 默认打开；如要关闭需Connection: close |

 注意：这里的 “Keep-Alive” 是 **HTTP 语义**，与操作系统内核的 **TCP keepalive 探活包** 不是一回事。

---

#### 2. 报文级触发方式

 - **开启长连接** ```cpp // HTTP/1.0 Connection: keep-alive ```

 - **关闭长连接** ```cpp // HTTP/1.1 Connection: close ```

 - **Keep-Alive 头**（可选，提示超时与复用上限） ```cpp Keep-Alive: timeout=5, max=100 ```

---

#### 3. 整体时序图

---

#### 4. 演进与关联协议

 - **HTTP/1.0** • 规范假定短连接；Connection: keep-alive为后续 RFC 扩展。

 - **HTTP/1.1** • 默认长连接，故浏览器能复用同一 TCP，减少 3 次握手。 • 但同一连接同一时刻只能飞一条请求，造成“队头阻塞”。

 - **HTTP/2** • 仍是长连接概念，但在单条 TCP 内用“多路复用”实现**并行多流**。

 - **HTTP/3 (QUIC)** • 长连接跑在 UDP + QUIC 上，可串流同时避免 TCP 队头阻塞；支持 0-RTT 恢复、连接迁移。

---

#### 5. 优缺点小结

| | 长连接 | 短连接 |
| --- | --- | --- |
| 性能 | ✅ 减少握手 RTT；节约端口 | ❌ 每次握手 + 慢启动 |
| 并发 | ⚠️ HTTP/1.x 仍受限：一条连接一次只能一个请求；需开多 TCP | ✅ 每次新建，天然并行 |
| 服务器资源 | ⚠️ 常驻 FD / 内存；需设空闲超时 | ✅ 用完即关，释放快 |
| 部署复杂度 | 需要正确的超时、连接池、负载均衡 sticky | 简单 |

---

#### 6. 超时 & 连接池实践

 - **浏览器** • Chrome 空闲 115 s 后断开；最多并发 6～10 条 TCP/域名（HTTP/1.x 时）。

 - **服务器** • Nginxkeepalive_timeout 75s；keepalive_requests 1000。

 - **后端 SDK** • JavaHttpClient/ Gohttp.Transport都默认启用连接池；需设置IdleConnTimeout。

---

#### 7. 常见面试陷阱

 - **“长连接=WebSocket？”** WebSocket 建立于 HTTP 升级后的独立协议，虽然也是长连接，但语义不等同。

 - **“开启长连接就无限复用？”** 服务器可随时Connection: close，或因 idle-timeout、max 请求数主动断开。

 - **“有长连接就不用多域名并发？”** 若升级到 HTTP/2，多路复用消除了此需求；HTTP/1.x 场景仍靠域名分片提升并发。

---

#### 8. 记忆口诀

```cpp
短连：一请一拆； 
长连：多请一拆； 
1.0 手动开，1.1 默认在； 
多路复用到 2，QUIC 飞到 3。
```

## 8. 从「敲下一个 URL」到「页面出现在屏幕」整条链路全景
一句话总结
 
“敲下 URL → 解析地址 → DNS 找人 → (TCP/QUIC+TLS) 打通线路 → 发 HTTP 请求 → 服务器回资源 → 浏览器边下边渲染 → 四次挥手收尾”。

---

### 全链路 7 大步骤

| # | 关键节点 | 细说可延伸内容 |
| --- | --- | --- |
| 1 | 地址栏解析 | URL 组成、浏览器缓存、Service-Worker 拦截、HSTS 强升 HTTPS |
| 2 | DNS 解析 | 递归/迭代、根/TLD/权威、DoH/DoT、DNS Cache、edns-client-subnet |
| 3 | 建立连接 | TCP 三次握手、TLS 1.3/QUIC 0-RTT、SYN 丢包/重传、SYN Cookies |
| 4 | 发送请求 | HTTP 1.x/2/3、请求⽅法、请求头、长连接&队头阻塞、代理链 |
| 5 | 服务器响应 | CDN 缓存、负载均衡、状态码、压缩、Cookie/Set-Cookie、ETag |
| 6 | 浏览器渲染 | HTML Parser、CSSOM、JS 执行、Layout→Paint→Composite、CLS/LCP |
| 7 | 关闭连接 | TCP 四次挥手、TIME_WAIT、Keep-Alive、HTTP/2 复⽤无需挥手 |

---

### 时序图

---

### 关键细节 & 面试延伸

 - **DNS 阶段** • 浏览器优先本地缓存→系统缓存→hosts→递归查询。 • 可插 DoH/DoT 保护隐私；preconnect抢跑解析。

 - **握手阶段** • TCP 三次握手：解决双方“收发能力”确认；SYN|ACK 可携带 MSS、Window Scale。 • TLS 1.3 把密钥交换并入握手，1-RTT；QUIC 把 TLS + 传输合一，0-RTT。

 - **HTTP 阶段** • 强缓存命中直接 200(from cache)。 • 协商缓存 If-None-Match → 304。 • HTTP/2 多路复用、HPACK 压缩；HTTP/3 用 UDP 消除队头阻塞。

 - **渲染阶段** • HTML 解析遇<script>（无 defer/async）会阻塞。 • 关键渲染路径：DOM → CSSOM → Render Tree → Layout → Paint → GPU Composite。 • 重排(Reflow)与重绘(Repaint)性能差异；content-visibility、will-change优化。

 - **挥手阶段** • 四次挥手：FIN、ACK、FIN、ACK；TIME_WAIT 确保迟到包被丢弃，避免旧连接干扰。 • HTTP Keep-Alive / HTTP-2 复用可复用连接，减少握手/挥手次数。

---

#### 记忆口诀

```text
DNS 找人 
握手通话 
HTTP 取货 
DOM+CSS 造骨架 
Layout 定位 
Paint 上色 
挥手收工
```

## 9. 什么是重定向？重定向与请求转发的区别？
#### 

#### 一句话总结

 重定向是服务器返回 3xx 让浏览器重新访问新 URL（地址栏会改变，需二次请求），而请求转发只在同一服务器内部把同一 request 转给其他资源处理（一次请求，地址栏不变）。

#### 1. 什么是“重定向” (Redirect)？

 - 概念 服务器返回 **3xx 状态码 + Location 头**，告诉浏览器“目标搬到另一条 URL 上”。浏览器随后会**自动再发一次请求**到新的地址。

 - 常见状态码 • 301 Moved Permanently（永久） • 302 Found（临时；最常见） • 303 See Other（POST-Redirect-GET 场景） • 307 / 308 保持原 HTTP 方法的临时 / 永久重定向

 - 关键特点 • **两次 HTTP 往返**：第一次请求原地址 → 3xx；第二次请求新地址 → 200 • 地址栏会改变，用户可看到跳转后的 URL • 可以跨域跨站跳转（A 站 → B 站） • 原request作用域的数据丢失，只能依靠 **URL 参数 / Cookie / Session** 传递信息

---

### 2. 什么是“请求转发” (Forward / Internal Dispatch)？

 - 概念 全程发生在服务器内部：Web 容器把 **同一次客户端请求** 交给另一个资源（Servlet、JSP、Controller …）继续处理，**客户端毫不知情**。

 - 技术入口（以 Java Servlet 为例）

```java
RequestDispatcher rd = request.getRequestDispatcher("/WEB-INF/view.jsp");
rd.forward(request, response);
```

 - 关键特点 • **只产生一次 HTTP 请求**；浏览器不知道发生过转发 • URL 不变，依旧显示最初输入的地址 • 只能在同一台服务器、同一 Web 应用内部使用 • 同一份request/response对象被下游资源共享 → 可以用request.setAttribute()传对象

---

### 3. 二者核心区别一张表

| 对比点 | 重定向 (Redirect) | 请求转发 (Forward) |
| --- | --- | --- |
| HTTP 层表现 | 3xx + Location，客户端再发第二次请求 | 仍为 200，一次往返 |
| URL 地址栏 | 会改变为目标 URL | 保持不变 |
| 作用域 | 新请求：不能直接拿到上次 request 中的属性 | 同一个 request，可共享属性 |
| 网络开销 | 至少多 1 次 RTT | 0 额外 RTT |
| 跨域能力 | 可以跨协议/域名/端口 | 仅限当前应用 |
| 典型用途 | PRG 模式、防刷新、SEO 链接搬迁、登录跳站 | MVC 内部视图渲染、过滤链、统一异常页 |
| 代码调用 | response.sendRedirect("/new") | dispatcher.forward(req, resp) |

---

### 4. 时序图直观对比

---

### 5. 选型口诀

```cpp
内部页面流转，用 Forward；
换域换地址，用 Redirect；
POST 提交防刷新 → 303 Redirect (PRG)；
SEO 链接永久搬家 → 301 Redirect。
```

 掌握“**是否让浏览器再发一次请求**”这一核心点，其它差异都能自然推导出来。

## 10. GET 与 POST 有什么区别?
#### 

#### 一句话总结

 **GET＝“只拿不改”**；**POST＝“提交或创建”**。二者在**语义、幂等性、缓存、参数位置、安全性**等方面都有官方规定，也有“江湖惯例”。

---

### 1. 标准语义（RFC 7231）

| 维度 | GET | POST |
| --- | --- | --- |
| 官方动作 | Retrieve（检索资源表示） | Submit / Process（提交数据） |
| 是否幂等 | 应当幂等且无副作用（Safe & Idempotent） | 无此保证，一般有副作用 |
| 状态变化 | 不应改变服务器资源 | 通常会创建/修改服务器资源 |

 “GET **按规范应设计为幂等**，但实际业务可能破坏此规则；POST 的幂等性需通过 **唯一请求标识符**（如订单号）或 **业务逻辑保证**。

 记忆口诀：**G**ET＝**G**rab，只抓；**P**OST＝**P**ush，推送数据。

---

### 2. 参数位置与编码

| 项目 | GET | POST |
| --- | --- | --- |
| 参数位置 | ?key=value置于 URL Query | 放在 请求体（Body），URL 可携带少量 Query |
| 典型编码 | application/x-www-form-urlencoded | x-www-form-urlencoded（表单默认）
multipart/form-data（文件上传）
application/json |
| 浏览器上限 | URL ⻓度受浏览器/服务器限制（2–8 KB 级别） | 理论无限，受服务器maxPostSize或 Nginxclient_max_body_size限制 |

---

### 3. 幂等 & 缓存

| 维度 | GET | POST |
| --- | --- | --- |
| 幂等性 | ✅ 多次请求应得到相同结果 | ❌ 可能产生多次插入、扣费等重复副作用 |
| 安全性 | ✅ Safe，不修改状态 | ❌ Unsafe，允许修改 |
| 浏览器缓存 | 可被自动缓存（ETag/Last-Modified） | 默认 不缓存；要缓存需显式Cache-Control+Vary |
| 前端重试 | 可大胆重发 | 重试需加 Token/幂等键防止重复写入 |

---

### 4. 可见性与安全

| 场景 | GET | POST |
| --- | --- | --- |
| 地址栏/书签 | ✅ 一目了然，可收藏 | ❌ URL 不变，看不到数据 |
| 日志 & 代理 | ✅ Query 会被记录到日志 | 体内数据多数代理不留存 |
| CSRF 风险 | ⚠️ 浏览器预取、外链都能触发 | ✅ 需同源 + 正确 Content-Type |
| 敏感信息泄漏 | ⚠️ 明文留在历史记录、Referer | 相对安全，但仍应用 HTTPS |

---

### 5. 状态码语义差异

 - GET 成功通常配 200 / 206（Range）。

 - POST 成功首选 **201 Created**（Location 指向新资源）或 **202 Accepted**（异步任务），返回 200 亦可。

 - 误用示例：对 POST 返回 204 但实际创建了资源，违反语义。

---

### 6. 实际开发场景

| 业务动作 | 推荐方法 | 备注 |
| --- | --- | --- |
| 查询商品列表 | GET | 可分页：/items?page=2 |
| 提交订单 | POST | JSON：POST /orders |
| RESTful 更新全量 | PUT | 幂等；PUT /users/123 |
| RESTful 更新部分 | PATCH | 非幂等；PATCH /users/123 |
| 幂等写场景 | POST + 幂等键或 PUT | 解决支付重试 |

---

### 7. 常见面试陷阱

 - **“GET 不能带请求体”** 规范 _允许_，但大多数服务器/中间件会忽略，不建议使用。

 - **“GET 比 POST 更不安全”** 真正的安全基石是 **TLS**；只是在 URL 暴露场景下 GET 更易泄漏。

 - **“POST 就一定非幂等”** 若业务方自行保证（如带唯一事务号），POST 也可以逻辑幂等。

 - **“POST 数据无限大”** 服务器仍需配置Content-Length上限来抵御大包攻击。

---

### 8. 速记对照表

| 口诀 | 含义 |
| --- | --- |
| 拿数据 → GET | 读操作，缓存友好 |
| 交数据 → POST | 写操作，副作用 |
| 能刷新不变 → GET | 幂等可重复 |
| 不可重试 → POST | 避免重复写 |

## 11. HTTP vs. HTTPS 有什么区别?
#### 

#### 一句话总结

 **HTTP 像明信片——谁都能偷看**；**HTTPS 像贴好身份证、加密又防篡改的快递箱**，既保证内容机密，也验证“寄件人”身份。

---

#### 1. 核心差异表

| 维度 | HTTP | HTTPS |
| --- | --- | --- |
| URL Scheme | http:// | https:// |
| 端口默认 | 80 | 443 |
| 传输层 | 直接跑在 TCP 之上 | 先经 TLS（SSL）再走 TCP；TLS 1.3 已与 QUIC 融合到 HTTP/3 |
| 加密 | ❌ 明文 | ✅ 对称加密（AES/ChaCha20） |
| 完整性校验 | ❌ 无 | ✅ MAC / AEAD，防篡改 |
| 身份认证 | ❌ 无 | ✅ 服务器证书（可选客户端证书） |
| 浏览器标识 | 无锁🔓 /Not Secure | 小锁🔒 / 绿色安全 |
| SEO & 新特性 | 排名普通；Service Worker 禁用 | 谷歌排名加分；PWA、HTTP/2/3、Web Bluetooth 等强制 HTTPS |
| 成本 | 证书零成本时代（Let’s Encrypt）依旧简易 | 需证书、配置、CPU 加解密 |
| 防御能力 | 易遭窃听、伪造、劫持（MITM） | 防窃听、防篡改、防冒充，但不解决业务逻辑漏洞 |

---

#### 2. HTTPS 工作原理

 - **握手阶段** TLS 1.2（经典）： ```cpp ClientHello → ServerHello → 证书 → 密钥交换 → Finish ``` 至少 2-RTT 才能发应用数据。

 - TLS 1.3：握手与密钥协商合并，仅 **1-RTT**；若复用会话可 **0-RTT**。

 - **建立对称密钥** 客户端用服务器公钥（证书里带）加密“会话随机数”，服务器解密后生成共享密钥 → 之后全部流量对称加密（速度快）。

 - **数据传输** 每一段记录都带 MAC / AEAD 标记，确保 **完整性**；同时序号递增，防回放。

---

#### 3. 典型安全收益

| 场景 | HTTP 风险 | HTTPS 防护 |
| --- | --- | --- |
| 公共 Wi-Fi | 嗅探账号密码 | 密文无法读取 |
| DNS 劫持/运营商插广告 | 可插 JS、篡改页面 | DNS 劫持可能导致连接到伪造服务器，但 HTTPS 通过证书链校验阻止连接建立；若劫持后插入广告，篡改内容会导致 MAC 校验失败 |
| 伪造钓鱼站 | 用户难分真假 | 证书链校验 + EV/锁标识 |
| Cookie 劫持 | SessionID 被监听 | SecureCookie 仅随 HTTPS 发送 |

---

#### 4. 记忆口诀

```cpp
加“ S ”三重益：S-Secure 加密机密 
 S-Seal 校验篡改 
 S-Sign 身份可信
端口 443 走 TLS，HSTS 锁 HTTPS。 
```

 牢记「加密 + 完整性 + 认证」三件套，即能清晰阐述 **HTTP 与 HTTPS 的所有区别**。

## 12. HTTPS 的「秘钥交换 + 证书校验」全流程
#### 

#### 一句话总结

 HTTPS 通过「证书像身份证、密钥交换像一次临时暗号本的当面约定」的握手流程，实现了“我是谁＋对话只让你看”双重保障。

### 0. 整体时序图

---

### 1. 证书校验：确认“你真的是你”

| 步骤 | 浏览器要做什么 | 好比 |
| --- | --- | --- |
| 提取证书 | 浏览器收到Certificate消息，拿到服务器证书链 | 收到一摞带公章的营业执照复印件 |
| 签名校验 | 逐级用本地“可信根”公钥验签 | 用政府盖章对照章纹 |
| 有效期 / 域名 | 证书中的NotBefore/NotAfter&CN/SAN需匹配 | 确认执照未过期、公司名一致 |
| 吊销检查 | OCSP / CRL 查询 | 问工商局：这执照是不是被吊销 |
| 结果 | 校验通过则继续，失败则弹红叉 | 章纹对不上＝假证，交易取消 |

 根证书是‘中央政府公章’，中间证书是‘省级部门公章’，服务器证书是‘企业执照’，需逐级盖章验证，浏览器 = “客户”，只有盖了真钢印的执照才可信。

---

### 2. 密钥交换：临时暗号本的诞生

 下面以现代常用的 ECDHE-RSA 为例（支持前向保密）。流程分三步：

 - 交换“随机因子” ClientHello 带随机数R_C，ServerHello 带随机数R_S。

 - 类比：两人各掷一次骰子，记录点数准备后续调味。

 - 生成「共同秘钥种子」(pre-master / shared secret) 服务器生成一对临时椭圆曲线密钥 (k_s,,P_s)，把公钥 P_s+ 自身证书一起发给客户端，并用私钥签名保证“这是我发的”。

 - 浏览器收到后，生成自己的临时私钥k_c、公钥P_c，计算共享秘密，基于椭圆曲线离散对数问题（ECDLP），临时私钥k_c 与对方公钥 P_s 的点乘运算生成共享密钥 text{Secret}=k_ccdot P_s = k_scdot P_c

 - **比喻**：双方各自选一把“万能钥匙柄”(私钥)，对方的“锁胚”(公钥)插进去后，魔法般磨出同一把钥匙齿形（共享秘钥），别人看不到磨痕过程。

 - 推导对话用的对称密钥 将 R_C, R_S,text{Secret}经过 PRF 导出 master secret，再分衍生出： • 加密密钥 (AES) • MAC 密钥 (HMAC) • IV 等

 - **性质**：即使明天服务器私钥泄露，因为用的是一次性临时密钥，今天的会话也解不开 —— 这就是“前向保密”。

---

### 3. 握手完结：开始加密说话

 - ClientKeyExchange 发完，客户端立即ChangeCipherSpec：告诉服务器“从下一条开始用我们约好的算法＋密钥”。

 - Finished消息携带握手所有内容的 HMAC；双方互验，无篡改即可认为握手成功。

 - 之后 HTTP 请求/响应都会套在 TLS 记录层，用对称密钥（如 AES-GCM）加密＋完整性校验。

---

### 4. 难点小结与形象比喻

| 难点 | 一句话解释 | 生动比喻 |
| --- | --- | --- |
| 公钥加密 vs 对称加密 | 前者解决“快递柜上锁”，后者解决“同一把锁反复开关” | 快递柜：我用公钥锁进去，你用私钥开；真正聊天像换上对讲机，频道提前对好 |
| 数字签名 | 用私钥生成指纹，任何人用公钥验真 | 私章+公章：只有老板私章能盖这印，别人能验印痕 |
| 前向保密 | 会话密钥与长期私钥脱钩，后者泄露也不怕历史被解密 | 一次性胶囊密码本：用完即焚 |
| 证书链 | 多级 CA 逐级背书 | 省级政府→市级→派出所的层层盖章 |

---

### 5. 常见面试延伸问答

 - 问：为什么不用服务器长期私钥直接做 Diffie-Hellman？ 答：私钥长期不变，失窃后历史流量可被解密；临时密钥可提供前向保密。

 - 问：证书里的公钥与 ServerKeyExchange 里的临时公钥有何区别？ 答：证书公钥用来验签 + 加密临时密钥，临时公钥只用于本次会话的 DH 计算。

 - 问：客户端如何验证握手未被篡改？ 答：最后Finished的 HMAC 覆盖全部握手记录；有人改任何一步都会导致验证失败。

---

#### 手写总结口诀（方便背诵）

 “先问版本随机数，后给证书查身书；

临时公钥验签过，共同密钥各自握；

换套加密说悄悄，前向保密最重要。”

## 13. HTTPS（TLS）里都用到了哪些加密算法？
#### 

#### 一句话总结

 HTTPS（TLS）就像一场“谍战流水线”：先靠密钥交换算法当面对暗号本，再用数字签名盖公章认证身份，随即把对称加密锁上保险箱，再贴上 MAC/哈希封条，最后用 PRF/HKDF 榨出更多密钥——多种加密算法分工协作，缺一不可。

### 1、整体分类总览

| 职责 | 在 TLS 中的位置 | 典型算法（粗体为主流） | 形象比喻 |
| --- | --- | --- | --- |
| 密钥交换（Key Exchange） | 握手阶段诞生会话密钥 | RSA（已弃用）、DHE / ECDHE、x25519、PSK | 当面掷骰子，约好暗号本 |
| 身份认证 / 数字签名 | 证书签名、ServerKeyExchange 签名 | RSA-PKCS#1 / RSA-PSS, ECDSA, Ed25519, Ed448, DSA(旧) | 盖公司公章 |
| 对称加密（Bulk Encryption） | 传输期加密 HTTP 内容 | AES-GCM/CBC/CCM, ChaCha20-Poly1305, 3DES, RC4(淘汰), Camellia | 把聊天内容塞进保险箱 |
| 消息完整性（MAC / AEAD Tag） | TLS 记录层校验篡改 | GHASH(AES-GCM), Poly1305, HMAC-SHA256/384, HMAC-SHA1(淘汰中) | 在箱口贴封条 |
| 哈希 / 摘要 | HMAC、签名、PRF 依赖 | SHA-256/384, SHA-512, SHA-1(弃用), MD5(淘汰), SHA-3, BLAKE2s | 压缩指纹 |
| 密钥派生 PRF / KDF | 派生 master secret 与子密钥 | TLS 1.2：PRF(P_hash)，TLS 1.3：HKDF-SHA256/384 | 榨汁机：把随机数 + 秘钥打成奶昔 |

 注：TLS 1.3 把“密钥交换/签名算法”与“对称算法”解耦，cipher suite 名称里只剩最后一类（例如TLS_AES_128_GCM_SHA256）。

---

### 2、各类型算法的职责与工作方式

#### 2.1 密钥交换：先确定“暗号本”

 - **DHE / ECDHE / x25519** 双方各抖一次骰子（私钥），用对方的骰杯（公钥）一撞，神奇地产生同一串数字（共享密钥）；旁人只看到杯子，看不到骰点，称为「前向保密」。

 - **RSA Key Transport**（TLS 1.0–1.2 老方法） 客户端直接把 pre-master 用服务器公钥锁进快递柜发过去；快递柜钥匙（私钥）长期不变，若失窃历史流量全军覆没，故已弃用。

#### 2.2 数字签名：给“身份”盖章

 证书链上的签名 & ServerKeyExchange 的签名常见：RSA-PKCS1_SHA256、ECDSA_secp256r1_sha256、Ed25519。

比喻：只有老板（私钥）能盖这枚章，任何人拿公章印痕（公钥）就能鉴别真假。

#### 2.3 对称加密：真正“装箱上锁”

 - **AES-GCM**（高速+带认证）、**ChaCha20-Poly1305**（移动端快）是当今主角。

 - AES-CBC、3DES、Camellia 仍在部分系统向后兼容。

 - RC4 因偏差严重被 RFC 7465 全面禁用。

#### 2.4 消息认证 (MAC / AEAD Tag)：给箱子贴封条

 - **AEAD** 算法（AES-GCM, ChaCha20-Poly1305）把“加密+完整性”一次打包，生成 16 字节 tag。

 - 旧 CBC 套件单独附HMAC-SHA256做封条。

#### 2.5 哈希函数：生成小而独特的“指纹”

 所有 HMAC、数字签名、PRF 都靠安全哈希。SHA-1 已被碰撞攻击退役；MD5 更是进博物馆。

#### 2.6 PRF / HKDF：榨出一串“子钥匙”

 TLS 1.3 用 **HKDF-SHA256/384**：

 - Extract：把共享秘密 + salt 炖成 master secret。

 - Expand：master secret 再拉丝，得到应用数据密钥、握手密钥等。 图示（伪代码）

```text
shared_secret ──HKDF-Extract──► master_secret
master_secret ──HKDF-Expand("handshake")──► hs_key
master_secret ──HKDF-Expand("application")──► app_key
```

 好比榨橙汁：先压出纯汁（Extract），再分别兑水做冰沙、果汁汽水（Expand）。

---

### 3、TLS 1.2 vs TLS 1.3 算法组合差异

| 版本 | 套餐写法 | 例子 | 说明 |
| --- | --- | --- | --- |
| TLS 1.2 | <密钥交换>_WITH_<对称算法>_<MAC> | ECDHE_RSA_WITH_AES_128_GCM_SHA256 | 三段式，长串绕口令 |
| TLS 1.3 | TLS_<对称算法>_<哈希> | TLS_AES_128_GCM_SHA256 | 只剩对称算法+哈希；密钥交换/签名单独协商 |

---

### 4、常见面试衍生问答

 - **为什么 AES-GCM/ChaCha20-Poly1305 被称为 AEAD？** 它们一次计算就输出密文 + 验证 tag ⇒ 同时保证机密性与完整性（Authenticated Encryption with Associated Data）。

 - **TLS 1.3 为何推荐 x25519 而非传统 ECDHE_secp256r1？** – 实现更短、更易写对常数时间；– 专利友好；– 避免曲线参数可疑。

 - **CBC + HMAC 组合有哪些坑？** 需处理填充 oracle、MAC-then-Encrypt 顺序等安全细节复杂，GCM/Poly1305 直接“自带封条”更省心。

---

### 5 口诀速记

 “先 ECDHE 抛骰子，RSA/ECDSA 来盖章；

AES/ChaCha 上锁箱，Poly/GHASH 贴封条；

SHA-256 做指纹，HKDF 榨密钥。”

## 14. WebSocket 简介 & 与 HTTP 的核心区别
### 

### 1. WebSocket 是什么？

 - **定义** WebSocket 是一种运行在 **TCP** 之上的全双工、长连接、低开销通信协议，最早由 RFC 6455 标准化。它让浏览器和服务器之间建立“**持续通话**”——像打电话一样你一句我一句，而不必像 HTTP 那样“写信往返”。

 - **基本特性一次握手、长期存活**：借助 HTTP Upgrade 只握手一次，随后复用同一 TCP 连接。

 - **双向实时**：浏览器 & 服务器都能主动推消息；延迟可低至毫秒级。

 - **帧格式轻量**：首部最小仅 2 Byte；支持文本帧、二进制帧、Ping/Pong 心跳。

 - **原生跨域**：只要服务器响应Sec-WebSocket-Accept，浏览器即可连，不受 Same-Origin 限制。

 - **加密**：wss://= WebSocket over TLS，安全性与 HTTPS 相当。

 - **典型应用场景** 线上聊天室、实时股价、在线游戏、协同编辑、IoT 推送、IM 即时通讯。

---

### 2. 握手流程（概要）

```text
① 浏览器 → 服务器 : GET /chat HTTP/1.1
 Host: example.com
 Upgrade: websocket
 Connection: Upgrade
 Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
 Sec-WebSocket-Version: 13
② 服务器 → 浏览器 : HTTP/1.1 101 Switching Protocols
 Upgrade: websocket
 Connection: Upgrade
 Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
③ 握手成功，双方开始按 **WebSocket 帧** 发送/接收数据
```

 *握手完后就 **脱离 HTTP** 语义，只保留底层 TCP 连接。* 

---

### 3. WebSocket vs. HTTP —— 8 大维度差异

| 维度 | HTTP (1.x/2.x) | WebSocket |
| --- | --- | --- |
| 通信模式 | 请求/响应，半双工 | 全双工，任意端可先发 |
| 连接生命周期 | 短连接（或 Keep-Alive），一事一连；HTTP/2 可多路复用 | 一次101升级后长驻 |
| 首部开销 | 每个请求动辄百字节 | 最小 2 Byte，且无重复首部 |
| 状态保持 | 无状态；靠 Cookie / Token 维护会话 | 连接本身天然有状态 |
| 服务器推送 | 1.x 需轮询 / SSE；2.x 支持 Server Push 但仅限资源 | 任意时刻可send消息 |
| 跨域限制 | 受浏览器同源策略 & CORS | 握手时不检查 Origin，需服务端自控 |
| 可靠性 | TCP 级别可靠，但一次请求丢包不影响后续请求 | 同一连接丢包会阻塞双方，需心跳探活 |
| 代理/缓存 | HTTP 代理、CDN、缓存丰富 | 需支持透明转发或显式Connection: Upgrade；中间节点多半旁路 |

---

#### 重点差异详解

 - **消息粒度** HTTP：以“完整报文”为单位；服务端必须等请求完整到达才能回应。

 - WebSocket：流式帧；可边生产边发送，适合大文件切片、实时流媒体。

 - **握手与协议升级** WebSocket 利用现有 80 / 443 端口 **复用** HTTP，穿透防火墙简单；升级后所有 HTTP 头缓存/代理机制失效。

 - **心跳保活** 浏览器每 30～60 s 发送Ping，服务器回Pong；若超时即重连，应用需做重连/容错逻辑。

 - **安全控制** 服务器需校验Origin，防止恶意网页盗连。

 - 推荐使用wss://避免中间人劫持；TLS 证书与 HTTPS 通用。

---

### 4. 何时选 WebSocket？何时仍用 HTTP？

| 需求 | 推荐协议 |
| --- | --- |
| 聊天室、股票行情、游戏 | WebSocket |
| 表单提交、REST 查询、文件下载 | HTTP/HTTPS |
| 服务器只需单向推送、量不大 | HTTP/2 Server Push 或 SSE |
| 网络不可长期保活（移动弱网） | 较短消息可考虑 HTTP 长轮询 |

---

### 5. 速记口诀

```text
HTTP：写信—收信；一句话来一句话回。
WebSocket：电话—随时说；一次拨号聊到底。
需要实时、双向、低延迟 → 选 WebSocket
一次性请求/资源传输 → 选 HTTP(S)
```

## 15. WebSocket 的工作过程是什么样的？
下面用「握手 → 建立 → 通信 → 保活 → 关闭」五个阶段，把一次典型的 WebSocket 会话拆给你看。配上关键报文、帧格式与注意事项，面试 & 实战都够用。

---

### 1. HTTP Upgrade 握手阶段（从 80/443“切轨”）

 - 浏览器（或其它客户端）先发一个 *特殊* 的 HTTP/1.1 请求： ```text GET /chat HTTP/1.1 Host: example.com Upgrade: websocket # 请求协议升级 Connection: Upgrade # 告诉代理“别断我 Upgrade 头” Sec-WebSocket-Key: N0m4== # 16 byte 随机串，Base64 Sec-WebSocket-Version: 13 # 当前唯一正式版本 Sec-WebSocket-Protocol: json # 可选，子协议协商 ```

 - 服务器校验后回101 Switching Protocols： ```text HTTP/1.1 101 Switching Protocols Upgrade: websocket Connection: Upgrade Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= # = Base64(SHA1(Key + GUID)) Sec-WebSocket-Protocol: json # 若同意子协议 ```

 - **握手成功** → 浏览器把这条 TCP 连接标记为 “WebSocket”。接下来就不走 HTTP 语义了，而是发送 **WebSocket 帧**。

 TLS 场景：若使用wss://，先是 TLS 握手，再进行上面的 HTTP Upgrade。

---

### 2. 帧格式与数据通道（全双工）

```cpp
┌────────┬────────┬───────────────┬─────────────┐
│ FIN/RSV/Op (1B) │ Mask+Len (1B) │ Ext-Payload │ Payload Data │
└────────┴────────┴───────────────┴─────────────┘
```

| 字段 | 说明 |
| --- | --- |
| FIN (1 bit) | 1=消息最后一帧，可做分片 |
| RSV1-3 (3 bit) | 扩展用，通常为 0 |
| Opcode (4 bit) | 0=继续帧，1=文本，2=二进制，8=Close，9=Ping，A=Pong |
| MASK (1 bit) | 浏览器→服务器必须为 1（安全混淆）；服务器→浏览器为 0 |
| Payload Len (7/7+16/7+64) | 0–125、126=后跟 16bit 长度、127=后跟 64bit |
| Mask-Key (4 B) | 仅当 MASK=1 时出现 |
| Payload Data | 正文；浏览器发出的先 XOR Mask-Key 再上网 |

---

### 3. 双向通信示例

 - **发送文本** ```javascript ws.send(JSON.stringify({msg: 'hello'})); ```

 - **接收消息** ```javascript ws.onmessage = evt => { const data = JSON.parse(evt.data); console.log('来自服务器:', data); }; ```

 - **二进制** ws.binaryType = 'arraybuffer'；服务器直接推Uint8Array。

---

### 4. 心跳保活 / 连接状态

| 帧类型 | 触发 | 用途 |
| --- | --- | --- |
| Ping (opcode 0x9) | 任意一端主动发 | 探测对方存活、测 RTT |
| Pong (0xA) | 接 Ping 或自发 | 回复心跳 |

 • 浏览器通常 30–60 s 发一次 Ping（由框架或业务层实现）。

• 若 **N 秒内**未收到任何数据/心跳，就认为断线 → 触发重连。

---

### 5. 关闭握手 & 断线重连

 - **优雅关闭** A 端：发送Close帧（opcode 0x8，含 2 字节状态码 + 可选原因）。

 - B 端：立刻回一个Close帧 → 连接进入 **CLOSED**。

 - 双方再由 TCP 交换FIN/ACK彻底释放。

 - **异常断开** 任何一端直接RST／网络掉线 → 浏览器会触发onclose，code=1006。

 - 前端常用指数退避做重连。

| 常用关闭码 | 含义 |
| --- | --- |
| 1000 | 正常关闭 |
| 1001 | 服务器下线或重启 |
| 1006 | 异常断线（只在客户端事件里可见） |
| 1008 | 业务级策略拒绝，如鉴权失败 |

---

### 6. 子协议（Sub-Protocol）

 - 在握手头Sec-WebSocket-Protocol中指定，比如json、protobuf、mqtt。

 - 服务器如同意，必须在 101 响应里回同名；否则按标准应拒绝连接 (400 Bad Request)。

---

### 7. 与代理 / CDN 交互

 - 必须让所有中间节点 **转发 Upgrade** 头；老旧 HTTP 代理会“吃”掉升级导致失败。

 - Nginx 需加：proxy_set_header Upgrade $http_upgrade; proxy_set_header Connection "Upgrade";

 - 多数 CDN 对wss已原生支持，并自动做多路复用。

---

### 8. 全流程时序图（简化版）

---

#### 速记口诀

```cpp
Upgrade 握成锁，101 把道过；
帧里 2 字节起，好分片双向说；
Ping-Pong 探活路，Close-ACK 再挥手。
```

## 16. SSE（Server-Sent Events）与 WebSocket 有什么区别?
一句话先记忆

**SSE＝“服务器单向广播的长 HTTP 流”**；**WebSocket＝“双向、实时的全双工专线”**。

如果只是 **服务器 → 浏览器** 的实时推送，用 SSE 足够；需要 **客户端 ↔ 服务器** 都能主动发消息，才上 WebSocket。

---

#### 1. 技术原理

| 维度 | SSE | WebSocket |
| --- | --- | --- |
| 标准 | HTML5 EventSource（WHATWG） | RFC 6455 |
| 传输层 | 纯 HTTP/HTTPS 长轮询流（text/event-stream） | HTTP Upgrade → 独立帧协议，长驻 TCP（ws:///wss://） |
| 建立方式 | EventSource直接发 GET 请求 | new WebSocket()，先 101 Upgrade |
| 通信方向 | 单向（服务器 → 客户端） | 全双工，任意端可先发 |
| 消息格式 | 纯文本：data: xxx\n\n，浏览器自动拆 | 二进制帧 / 文本帧；首部仅 2~14 B |
| 重连 | 浏览器内置自动重连（retry:指定间隔） | 需应用层自己实现重连逻辑 |
| 心跳 | 服务器可定期:\n\n注释行 | Ping / Pong 帧 |
| 代理 & CDN | 100% 兼容 HTTP 代理、缓存 | 代理必须支持Connection: Upgrade |
| 浏览器支持 | Chrome/Edge/Firefox/Safari 全支持；IE 不支持 | 主流浏览器全支持 |
| 二进制/文件 | 不支持；需 Base64 | 支持 ArrayBuffer & Blob |
| 子协议 | 无 | 可协商Sec-WebSocket-Protocol |
| CORS | 与普通 XHR 相同 | 握手时浏览器会带Origin:，服务端需校验 |

---

#### 2. 示例代码对比

```javascript
/* SSE：客户端 */
const es = new EventSource('/price/stream');
es.onmessage = e => console.log('价格 =', e.data);
es.onerror = () => console.log('断线，浏览器自动重连');

/* WebSocket：客户端 */
const ws = new WebSocket('wss://example.com/chat');
ws.onopen = () => ws.send('hello');
ws.onmessage = e => console.log('服务器说：', e.data);
ws.onclose = () => retryConnect();
```

---

#### 3. 典型使用场景

| 需求 | 首选 |
| --- | --- |
| 股票 / 新闻 / 日志实时推送（只下行） | SSE |
| 聊天室、IM、在线游戏（上下行交互） | WebSocket |
| 低功耗 IoT，需节省头开销 | WebSocket（二进制） |
| 受限于公司代理、防火墙，仅放行 80/443 | SSE 更易打通 |
| 需要断线自动续传最新事件 ID | SSE（Last-Event-ID） |

---

#### 4. 优缺点速览

| | SSE 优势 | SSE 局限 |
| --- | --- | --- |
| ✅ 长连接仍走 HTTP：无额外端口/协议 | ❌ 只能服务器→客户端 | |
| ✅ 浏览器自动重连、事件 ID 续传 | ❌ 纯文本，不支持二进制 | |
| ✅ 头部最小，仅 6 字节 | ❌ 同源限制下需 CORS | |

| | WebSocket 优势 | WebSocket 局限 |
| --- | --- | --- |
| ✅ 全双工 + 二进制流 | ❌ 必须自己做心跳、重连 | |
| ✅ 帧开销更小（2 B 起） | ❌ 部分老旧代理不支持 Upgrade | |
| ✅ 子协议可扩展（MQTT、WAMP） | ❌ 前端实现复杂度略高 | |

---

#### 5. 选型口诀

```cpp
只下行→用 SSE； 双向聊→WebSocket；
HTTP 友好想省事→SSE； 要二进制高实时→WS。
```

---
