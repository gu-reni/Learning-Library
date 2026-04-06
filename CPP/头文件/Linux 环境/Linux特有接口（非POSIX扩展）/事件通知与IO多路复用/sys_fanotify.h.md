## `<sys/fanotify.h>` 头文件详解（Linux 特有接口）

`<sys/fanotify.h>` 是 Linux 特有的头文件，提供了 **fanotify**（File System Notification）API，用于监控文件系统事件并支持访问控制决策。相比于传统的 `inotify`，fanotify 功能更加强大，能够监控文件系统的访问、修改、创建、删除等事件，并在访问发生前决定是否允许该操作，这使得它在安全软件（如病毒扫描、恶意软件防护）和分层存储管理等场景中具有不可替代的优势[reference:0][reference:1]。

---

## 一、函数详解

### 1. fanotify_init

**函数原型：**
```c
int fanotify_init(unsigned int flags, unsigned int event_f_flags);
```

**作用：** 创建并初始化一个 fanotify 通知组，返回一个文件描述符，用于后续的事件队列操作和事件监控[reference:2]。

**参数：**
- `flags`：指定通知组的特性和行为，是一个多比特位掩码。主要包含以下几个部分：
  - **通知类**（必须且仅能选择一个）：定义监听程序的优先级，当多个监听者同时存在时，内核将按此顺序分发事件。
    - `FAN_CLASS_PRE_CONTENT`：最高优先级。适用于需要在文件最终数据被访问前进行操作的程序，例如分层存储管理器。使用此标志需要 `CAP_SYS_ADMIN` 能力[reference:3]。
    - `FAN_CLASS_CONTENT`：中等优先级。适用于需要在文件内容已经存在后访问文件的程序，例如恶意软件检测工具。使用此标志需要 `CAP_SYS_ADMIN` 能力[reference:4]。
    - `FAN_CLASS_NOTIF`：最低优先级，是默认值。仅接收通知事件，无法进行权限决策[reference:5]。
  - **行为标志**：控制文件描述符的行为。
    - `FAN_CLOEXEC`：在 `exec` 时自动关闭该文件描述符[reference:6]。
    - `FAN_NONBLOCK`：将事件队列设置为非阻塞模式[reference:7]。
  - **扩展标志**（Linux 3.15+）：
    - `FAN_UNLIMITED_QUEUE`：移除对事件队列大小的限制（需要 `CAP_SYS_ADMIN`）。
    - `FAN_UNLIMITED_MARKS`：移除对标记数量的限制（需要 `CAP_SYS_ADMIN`）。
- `event_f_flags`：指定在事件读取时传递给 `read(2)` 调用的文件状态标志。常用的标志包括 `O_RDONLY`、`O_LARGEFILE` 等。对于权限事件，该值必须设为 `O_RDWR`[reference:8]。

**返回值：** 成功时返回一个非负文件描述符，失败时返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/fanotify.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

int main() {
    // 初始化 fanotify，使用默认的通知类（FAN_CLASS_NOTIF）和 O_RDONLY 事件标志
    int fan_fd = fanotify_init(FAN_CLOEXEC, O_RDONLY);
    if (fan_fd == -1) {
        perror("fanotify_init");
        return 1;
    }
    printf("fanotify initialized with fd: %d\n", fan_fd);
    // 后续使用 fan_fd 进行标记和事件读取
    close(fan_fd);
    return 0;
}
```

**实现原理：** `fanotify_init()` 是一个系统调用。内核会创建一个 `fanotify_group` 结构体，分配一个文件描述符指向该组的事件队列。该组包含一个等待队列和一组标记（marks），用于后续通过 `fanotify_mark()` 注册感兴趣的监控目标[reference:9]。

**线程安全提示：** 函数本身是线程安全的，但返回的文件描述符在跨线程共享时，需要使用者自行进行同步保护。

---

### 2. fanotify_mark

**函数原型：**
```c
int fanotify_mark(int fanotify_fd, unsigned int flags, uint64_t mask,
                  int dirfd, const char *pathname);
```

**作用：** 向一个 fanotify 通知组中添加、移除或修改一个监控标记（mark），指定要监控的文件、目录、挂载点或文件系统，以及要报告或忽略的事件类型[reference:10]。

**参数：**
- `fanotify_fd`：由 `fanotify_init()` 返回的文件描述符。
- `flags`：控制操作类型的标志位，可取以下值的按位组合：
  - `FAN_MARK_ADD`：添加一个新的标记[reference:11]。
  - `FAN_MARK_REMOVE`：移除一个已有的标记[reference:12]。
  - `FAN_MARK_FLUSH`：清除某个范围内的所有标记。
  - `FAN_MARK_DONT_FOLLOW`：如果 `pathname` 是一个符号链接，则监控链接本身，而非其指向的目标[reference:13]。
  - `FAN_MARK_ONLYDIR`：仅当 `pathname` 是一个目录时才进行标记，否则失败[reference:14]。
  - `FAN_MARK_MOUNT`：监控整个挂载点（mount point）上的所有对象[reference:15]。
  - `FAN_MARK_FILESYSTEM`：监控整个文件系统的所有对象（Linux 4.20+）。
  - `FAN_MARK_IGNORE`：设置一个忽略掩码（ignore mask），用于排除特定事件。
  - `FAN_MARK_IGNORED_SURV_MODIFY`：使忽略掩码在文件或目录被修改后依然有效。
- `mask`：要监控的事件掩码，可以是多个 `FAN_*` 事件的按位组合（参见下文宏定义）。
- `dirfd`：当 `pathname` 是相对路径时，作为参考目录的文件描述符。可以设为 `AT_FDCWD` 表示使用当前工作目录。
- `pathname`：要监控的对象路径，可以为 `NULL` 来配合 `dirfd` 使用。

**返回值：** 成功时返回 0，失败时返回 -1 并设置 `errno`。

**示例用法（添加监控）：**
```c
// 假设 fan_fd 是已经初始化的 fanotify 文件描述符
// 监控路径 "/home/user" 下的访问和修改事件，并递归监控子目录
if (fanotify_mark(fan_fd,
                  FAN_MARK_ADD | FAN_MARK_MOUNT,    // 添加标记，并监控整个挂载点
                  FAN_ACCESS | FAN_MODIFY,          // 监控访问和修改事件
                  AT_FDCWD,                         // 使用当前工作目录
                  "/home/user") == -1) {
    perror("fanotify_mark");
}
```

**实现原理：** 该系统调用在内核中查找或创建对应的监控条目（`fanotify_mark_entry`），并将其与通知组关联。标记可以基于 inode、挂载点或整个文件系统，内核会在这些对象上设置钩子（hook），当事件发生时触发回调函数将事件入队[reference:16]。

**线程安全提示：** 线程安全。多个线程可以同时对同一个 `fanotify_fd` 调用 `fanotify_mark`，内核会通过锁机制保证操作的原子性。

---

### 3. 通过 `read()` / `write()` 处理事件

`fanotify_init()` 返回的文件描述符支持标准的 `read(2)` 和 `write(2)` 系统调用。

- **`read(fan_fd, buf, len)`**：从事件队列中读取一个或多个 `fanotify_event_metadata` 结构体。该操作通常是阻塞的，除非在 `fanotify_init()` 中设置了 `FAN_NONBLOCK` 或文件描述符被设置为非阻塞模式[reference:17]。

- **`write(fan_fd, buf, len)`**：用于对权限事件（permission event）进行响应。当接收到一个 `FAN_ACCESS_PERM` 或 `FAN_OPEN_PERM` 事件时，应用程序需要通过 `write()` 向内核写入一个 `fanotify_response` 结构体，告知内核是允许还是拒绝该访问操作[reference:18]。

**示例用法（事件循环）：**
```c
#include <sys/fanotify.h>
#include <stdio.h>
#include <unistd.h>

#define BUF_SIZE 4096

int main() {
    int fan_fd = fanotify_init(FAN_CLASS_NOTIF, O_RDONLY);
    if (fan_fd == -1) return 1;

    // 监控整个根文件系统的访问和修改
    if (fanotify_mark(fan_fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_ACCESS | FAN_MODIFY,
                      AT_FDCWD, "/") == -1) return 1;

    char buf[BUF_SIZE];
    while (1) {
        ssize_t len = read(fan_fd, buf, sizeof(buf));
        if (len == -1) {
            perror("read");
            break;
        }
        struct fanotify_event_metadata *metadata = (struct fanotify_event_metadata *)buf;
        while (FAN_EVENT_OK(metadata, len)) {
            if (metadata->mask & FAN_ACCESS)
                printf("File accessed (PID %d)\n", metadata->pid);
            if (metadata->mask & FAN_MODIFY)
                printf("File modified (PID %d)\n", metadata->pid);
            // 关闭事件中携带的文件描述符（如有）
            if (metadata->fd != FAN_NOFD)
                close(metadata->fd);
            metadata = FAN_EVENT_NEXT(metadata, len);
        }
    }
    close(fan_fd);
    return 0;
}
```

**实现原理：** 当监控的文件系统对象上发生事件时，内核会分配一个 `fanotify_event` 结构体，填充事件信息（如 mask、pid、文件描述符等），并将其放入事件队列。应用程序通过 `read()` 从队列中取出事件，内核将数据从内核空间拷贝到用户空间。对于权限事件，应用程序通过 `write()` 将决策结果写回内核，内核据此决定是否放行该操作。

**线程安全提示：** 对同一 `fan_fd` 的并发 `read` 和 `write` 操作需要外部同步，否则可能导致事件混乱或丢失。建议使用一个专用的线程处理事件循环，或者使用锁保护。

---

## 二、结构体详解

### 1. struct fanotify_event_metadata

**定义：**
```c
struct fanotify_event_metadata {
    __u32 event_len;      // 整个事件结构体的长度（包含 header 和数据）
    __u8 vers;            // 结构体版本号，必须与 FANOTIFY_METADATA_VERSION 匹配
    __u8 reserved;        // 保留字段
    __u16 metadata_len;   // 头部长度
    __aligned_u64 mask;   // 事件掩码，表示发生的事件类型
    __s32 fd;             // 指向被操作文件的文件描述符，若为 FAN_NOFD 则无效
    __s32 pid;            // 触发事件的进程 ID
};
```

**作用：** 描述一个 fanotify 事件的基本信息。当从 fanotify 文件描述符读取事件时，应用程序会获得一个或多个这样的结构体[reference:19]。

**成员详解：**
- `event_len`：整个事件结构体的长度，包含头部和可选的扩展数据（如文件路径）。
- `vers`：结构体版本号，应用程序应将其与 `FANOTIFY_METADATA_VERSION` 进行比较以确保兼容。
- `metadata_len`：元数据头部的长度。
- `mask`：事件的类型掩码，可以是多个 `FAN_*` 标志的组合。
- `fd`：指向被操作文件的文件描述符。应用程序可以通过它读取文件内容或获取更多信息，使用完毕后必须调用 `close()` 关闭。若值为 `FAN_NOFD` 则表示没有可用的文件描述符。
- `pid`：触发事件的进程 ID。应用程序可以通过 `/proc/pid` 获取进程的详细信息[reference:20]。

---

### 2. struct fanotify_response

**定义：**
```c
struct fanotify_response {
    __s32 fd;       // 对应事件中的 fd
    __u32 response; // 响应值，可取 FAN_ALLOW 或 FAN_DENY
};
```

**作用：** 用于对权限事件（permission event）进行响应，通过 `write()` 系统调用写入到 fanotify 文件描述符。

**成员详解：**
- `fd`：对应事件中的 `fd` 字段，用于关联响应与特定事件。
- `response`：决策结果，可以是 `FAN_ALLOW`（允许访问）或 `FAN_DENY`（拒绝访问）。

---

### 3. struct fanotify_event_info_header

**定义：**
```c
struct fanotify_event_info_header {
    __u16 info_type;    // 信息类型，如 FAN_EVENT_INFO_TYPE_FID
    __u16 len;          // 该信息块的总长度
};
```

**作用：** 当请求了 `FAN_REPORT_FID` 等标志时，事件结构体后可能会附加多个信息块（info chunks），该结构体是这些信息块的通用头部[reference:21]。

---

### 4. struct fanotify_event_info_fid

**定义：**
```c
struct fanotify_event_info_fid {
    struct fanotify_event_info_header hdr;
    __kernel_fsid_t fsid;           // 文件系统 ID
    unsigned char handle[0];        // 文件句柄（可变长度）
};
```

**作用：** 用于报告与事件相关的文件标识符（file identifier），在需要获取被操作文件的路径时特别有用。

---

### 5. struct fanotify_event_info_error

**定义：**
```c
struct fanotify_event_info_error {
    struct fanotify_event_info_header hdr;
    __s32 error;        // 错误码
    __u32 error_count;  // 错误计数
};
```

**作用：** 当监控到文件系统错误时（通过 `FAN_FS_ERROR` 事件），该结构体用于传递错误信息（Linux 5.1+）。

---

## 三、宏定义详解

### 1. 事件类型掩码（用于 `fanotify_mark` 的 `mask` 和 `fanotify_event_metadata` 的 `mask`）

| 宏名 | 作用 |
|------|------|
| `FAN_ACCESS` | 文件被访问（读取）[reference:22] |
| `FAN_MODIFY` | 文件被修改 |
| `FAN_CLOSE_WRITE` | 可写方式打开的文件被关闭 |
| `FAN_CLOSE_NOWRITE` | 只读方式打开的文件被关闭 |
| `FAN_CLOSE` | 文件被关闭（包含上面两者） |
| `FAN_OPEN` | 文件被打开 |
| `FAN_OPEN_EXEC` | 文件被执行（Linux 5.0+） |
| `FAN_ATTRIB` | 文件元数据（如权限、所有者）被修改 |
| `FAN_CREATE` | 在监控目录中创建了文件或子目录（Linux 5.1+） |
| `FAN_DELETE` | 在监控目录中删除了文件或子目录（Linux 5.1+） |
| `FAN_DELETE_SELF` | 被监控的对象本身被删除（Linux 5.1+） |
| `FAN_MOVE_SELF` | 被监控的对象本身被移动（Linux 5.1+） |
| `FAN_MOVED_FROM` | 文件从监控目录中被移出（Linux 5.1+） |
| `FAN_MOVED_TO` | 文件被移入监控目录（Linux 5.1+） |
| `FAN_MOVE` | `FAN_MOVED_FROM` 或 `FAN_MOVED_TO` 的统称（Linux 5.1+） |
| `FAN_Q_OVERFLOW` | 事件队列溢出，表示有事件丢失 |
| `FAN_ACCESS_PERM` | 应用程序需要决定是否允许访问（权限事件） |
| `FAN_OPEN_PERM` | 应用程序需要决定是否允许打开（权限事件） |
| `FAN_OPEN_EXEC_PERM` | 应用程序需要决定是否允许执行（权限事件，Linux 5.0+） |
| `FAN_ONDIR` | 仅在目录上报告事件（通常与其它事件配合使用） |
| `FAN_EVENT_ON_CHILD` | 对于目录，监控其子对象的事件[reference:23] |
| `FAN_FS_ERROR` | 报告文件系统错误（Linux 5.1+） |

---

### 2. 通知类（用于 `fanotify_init` 的 `flags`）

| 宏名 | 作用 |
|------|------|
| `FAN_CLASS_NOTIF` | 默认值。仅接收通知事件，无法进行权限决策[reference:24] |
| `FAN_CLASS_CONTENT` | 接收通知事件和权限事件，适用于需要在文件内容已存在后访问文件的程序[reference:25] |
| `FAN_CLASS_PRE_CONTENT` | 接收通知事件和权限事件，适用于需要在文件最终数据被访问前进行操作的场景[reference:26] |

---

### 3. 初始化标志（用于 `fanotify_init` 的 `flags`）

| 宏名 | 作用 |
|------|------|
| `FAN_CLOEXEC` | 在 `exec` 时自动关闭文件描述符[reference:27] |
| `FAN_NONBLOCK` | 设置非阻塞模式[reference:28] |
| `FAN_UNLIMITED_QUEUE` | 移除对事件队列大小的限制（需要 `CAP_SYS_ADMIN`）[reference:29] |
| `FAN_UNLIMITED_MARKS` | 移除对标记数量的限制（需要 `CAP_SYS_ADMIN`）[reference:30] |
| `FAN_REPORT_TID` | 报告触发事件的线程 ID，而非进程 ID（Linux 4.20+） |
| `FAN_REPORT_FID` | 在事件中包含文件标识符（用于获取路径） |
| `FAN_REPORT_DIR_FID` | 在事件中同时包含目录和文件的标识符 |
| `FAN_REPORT_NAME` | 在 `FAN_MOVE` 和 `FAN_CREATE` 事件中报告新文件名（Linux 5.2+） |

---

### 4. 标记操作标志（用于 `fanotify_mark` 的 `flags`）

| 宏名 | 作用 |
|------|------|
| `FAN_MARK_ADD` | 添加一个新的标记[reference:31] |
| `FAN_MARK_REMOVE` | 移除一个已有的标记[reference:32] |
| `FAN_MARK_FLUSH` | 清除某个范围内的所有标记 |
| `FAN_MARK_DONT_FOLLOW` | 不跟随符号链接[reference:33] |
| `FAN_MARK_ONLYDIR` | 仅当 `pathname` 是目录时才进行标记[reference:34] |
| `FAN_MARK_MOUNT` | 监控整个挂载点[reference:35] |
| `FAN_MARK_FILESYSTEM` | 监控整个文件系统（Linux 4.20+） |
| `FAN_MARK_IGNORE` | 设置一个忽略掩码，排除特定事件 |
| `FAN_MARK_IGNORED_SURV_MODIFY` | 使忽略掩码在文件或目录被修改后依然有效 |

---

### 5. 响应值（用于 `struct fanotify_response` 的 `response`）

| 宏名 | 作用 |
|------|------|
| `FAN_ALLOW` | 允许本次访问操作 |
| `FAN_DENY` | 拒绝本次访问操作 |

---

### 6. 辅助宏

| 宏名 | 作用 |
|------|------|
| `FAN_EVENT_OK(meta, len)` | 检查 `meta` 指针指向的事件是否在缓冲区 `len` 范围内 |
| `FAN_EVENT_NEXT(meta, len)` | 获取下一个事件的指针 |

---

## 四、类型定义

- `fanotify_event_metadata_t`：通常定义为 `struct fanotify_event_metadata`。
- `fanotify_response_t`：通常定义为 `struct fanotify_response`。
- `fanotify_event_info_header_t`：通常定义为 `struct fanotify_event_info_header`。
- `fanotify_event_info_fid_t`：通常定义为 `struct fanotify_event_info_fid`。
- `fanotify_event_info_error_t`：通常定义为 `struct fanotify_event_info_error`。

---

## 五、示例用法

以下是一个完整的示例，演示如何使用 fanotify 监控整个根文件系统的访问和修改事件：

```c
#include <sys/fanotify.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

#define BUF_SIZE 4096

int main() {
    // 1. 初始化 fanotify 组
    int fan_fd = fanotify_init(FAN_CLOEXEC | FAN_CLASS_NOTIF, O_RDONLY);
    if (fan_fd == -1) {
        perror("fanotify_init");
        return 1;
    }

    // 2. 添加监控标记：监控整个根文件系统的访问和修改事件
    if (fanotify_mark(fan_fd, FAN_MARK_ADD | FAN_MARK_MOUNT,
                      FAN_ACCESS | FAN_MODIFY,
                      AT_FDCWD, "/") == -1) {
        perror("fanotify_mark");
        close(fan_fd);
        return 1;
    }

    printf("Monitoring / for access and modify events...\n");

    char buf[BUF_SIZE];
    while (1) {
        ssize_t len = read(fan_fd, buf, sizeof(buf));
        if (len == -1) {
            perror("read");
            break;
        }

        struct fanotify_event_metadata *metadata = (struct fanotify_event_metadata *)buf;
        while (FAN_EVENT_OK(metadata, len)) {
            if (metadata->mask & FAN_ACCESS) {
                printf("File accessed (PID %d, fd=%d)\n", metadata->pid, metadata->fd);
            }
            if (metadata->mask & FAN_MODIFY) {
                printf("File modified (PID %d, fd=%d)\n", metadata->pid, metadata->fd);
            }
            // 关闭事件中携带的文件描述符
            if (metadata->fd != FAN_NOFD) {
                close(metadata->fd);
            }
            metadata = FAN_EVENT_NEXT(metadata, len);
        }
    }

    close(fan_fd);
    return 0;
}
```

---

## 六、头文件分类

```
Linux 环境 / Linux 特有接口 / 其他系统调用扩展
```