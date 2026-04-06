## `<sys/inotify.h>` 头文件详解（Linux 特有接口）

`<sys/inotify.h>` 是 Linux 特有的头文件，提供了 **inotify** 文件系统事件监控机制。inotify 允许应用程序监控文件系统中的事件（如文件创建、删除、修改、移动等），并实时获取通知。相比于过时的 `dnotify`，inotify 具有更好的可扩展性和易用性，广泛应用于文件管理器、同步工具、安全监控等场景。

---

## 一、函数详解

### 1. inotify_init / inotify_init1

**函数原型：**
```c
int inotify_init(void);
int inotify_init1(int flags);
```

**作用：** 创建一个 inotify 实例，返回一个文件描述符，后续通过该描述符读取事件。

**参数：**
- `inotify_init`：无参数。
- `inotify_init1`：`flags` 可以是 `0` 或以下值的按位或：
  - `IN_CLOEXEC`：设置 close-on-exec 标志。
  - `IN_NONBLOCK`：设置非阻塞模式。

**返回值：** 成功返回非负文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/inotify.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main() {
    int fd = inotify_init1(IN_NONBLOCK | IN_CLOEXEC);
    if (fd == -1) {
        perror("inotify_init1");
        return 1;
    }
    close(fd);
    return 0;
}
```

**实现原理：** 系统调用进入内核，创建一个 inotify 实例（`struct inotify_device`），初始化事件队列，分配匿名文件描述符，并关联到进程的文件描述符表。

**线程安全提示：** 是线程安全的。多个线程可同时创建不同的 inotify 实例。

---

### 2. inotify_add_watch

**函数原型：**
```c
int inotify_add_watch(int fd, const char *pathname, uint32_t mask);
```

**作用：** 向 inotify 实例中添加一个监控项（watch），返回一个唯一的监视描述符（wd）。

**参数：**
- `fd`：inotify 实例的文件描述符。
- `pathname`：要监控的文件或目录路径。
- `mask`：要监控的事件掩码，可以是多个 `IN_*` 标志的按位组合。

**返回值：** 成功返回一个非负的监视描述符（wd），失败返回 -1 并设置 `errno`。

**示例用法：**
```c
int wd = inotify_add_watch(fd, "/home/user", IN_CREATE | IN_DELETE);
if (wd == -1) perror("inotify_add_watch");
```

**实现原理：** 内核根据 `pathname` 查找对应的 inode，为该 inode 创建一个 `inotify_watch` 结构体，关联到 inotify 实例和事件掩码，并返回 wd。同一个 inode 上多个 watch 会共享。

**线程安全提示：** 线程安全。对同一个 `fd` 的并发 `inotify_add_watch` 由内核同步保护。

---

### 3. inotify_rm_watch

**函数原型：**
```c
int inotify_rm_watch(int fd, int wd);
```

**作用：** 从 inotify 实例中移除一个监控项。

**参数：**
- `fd`：inotify 实例的文件描述符。
- `wd`：由 `inotify_add_watch` 返回的监视描述符。

**返回值：** 成功返回 0，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
if (inotify_rm_watch(fd, wd) == -1) perror("inotify_rm_watch");
```

**实现原理：** 内核根据 wd 查找 `inotify_watch` 结构体，将其从 inode 的监视链表和实例的红黑树中移除，并释放资源。

**线程安全提示：** 线程安全。

---

### 4. 通过 `read()` 读取事件

`inotify_init()` 返回的文件描述符支持 `read()` 操作，用于读取一个或多个 `inotify_event` 结构体。

**作用：** 从事件队列中读取事件，阻塞直到有事件发生（除非设置了非阻塞）。

**示例用法：**
```c
char buf[4096];
ssize_t len = read(fd, buf, sizeof(buf));
if (len > 0) {
    struct inotify_event *event = (struct inotify_event *)buf;
    while ((char *)event < buf + len) {
        if (event->mask & IN_CREATE)
            printf("File %s created\n", event->name);
        event = (struct inotify_event *)((char *)event + sizeof(*event) + event->len);
    }
}
```

**实现原理：** 当监控的文件系统对象上发生事件时，内核构建 `inotify_event` 结构体并放入事件队列。`read()` 将队列中的数据拷贝到用户空间。

**线程安全提示：** 对同一 `fd` 的并发 `read` 需要外部同步，建议单线程读取。

---

## 二、结构体详解

### struct inotify_event

**定义：**
```c
struct inotify_event {
    int      wd;       // 监视描述符
    uint32_t mask;     // 事件掩码
    uint32_t cookie;   // 用于关联相关事件（如移动）
    uint32_t len;      // name 字段的长度
    char     name[];   // 可变长度的文件名（仅当事件关联到目录内的文件时存在）
};
```

**作用：** 描述一个 inotify 事件，从文件描述符读取得到。

**成员详解：**
- `wd`：触发事件的监视描述符。
- `mask`：事件类型掩码，见下文宏定义。
- `cookie`：用于关联两个事件（例如 `IN_MOVED_FROM` 和 `IN_MOVED_TO` 会共享同一个 cookie）。
- `len`：`name` 字段的实际长度（不含终止符）。若为 0，则表示没有 name 字段。
- `name`：可变数组，存储产生事件的文件名（仅当监控的是目录且事件与目录内文件相关时有效）。

---

## 三、宏定义详解

### 事件掩码（用于 `inotify_add_watch` 的 `mask` 和 `inotify_event.mask`）

| 宏名 | 作用 |
|------|------|
| `IN_ACCESS` | 文件被读取 |
| `IN_MODIFY` | 文件被修改 |
| `IN_ATTRIB` | 文件元数据（权限、时间戳等）被修改 |
| `IN_CLOSE_WRITE` | 以可写方式打开的文件被关闭 |
| `IN_CLOSE_NOWRITE` | 以只读方式打开的文件被关闭 |
| `IN_CLOSE` | `IN_CLOSE_WRITE` 或 `IN_CLOSE_NOWRITE` |
| `IN_OPEN` | 文件被打开 |
| `IN_MOVED_FROM` | 文件从监控目录中被移出 |
| `IN_MOVED_TO` | 文件被移入监控目录 |
| `IN_MOVE` | `IN_MOVED_FROM` 或 `IN_MOVED_TO` |
| `IN_CREATE` | 在监控目录中创建了文件/子目录 |
| `IN_DELETE` | 在监控目录中删除了文件/子目录 |
| `IN_DELETE_SELF` | 被监控的对象本身被删除 |
| `IN_MOVE_SELF` | 被监控的对象本身被移动 |
| `IN_UNMOUNT` | 被监控的文件系统被卸载 |
| `IN_Q_OVERFLOW` | 事件队列溢出（事件丢失） |
| `IN_IGNORED` | 监控项被移除（例如文件被删除） |
| `IN_ONLYDIR` | 仅当路径是目录时才监控（用于 `inotify_add_watch`） |
| `IN_DONT_FOLLOW` | 不跟随符号链接 |
| `IN_EXCL_UNLINK` | 当监控对象被 unlink 后不再产生事件 |
| `IN_MASK_ADD` | 将新事件掩码添加到现有的掩码上 |
| `IN_ONESHOT` | 事件触发后自动移除监控项 |

---

### 辅助宏

| 宏名 | 作用 |
|------|------|
| `IN_ISDIR` | 事件发生的对象是一个目录（出现在 `mask` 中） |

---

## 四、类型定义

- `inotify_event` 结构体。
- 无额外的类型别名。

---

## 五、示例用法

以下是一个完整的示例，监控 `/tmp` 目录下的文件创建和删除事件：

```c
#include <sys/inotify.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

#define EVENT_SIZE  (sizeof(struct inotify_event))
#define BUF_LEN     (1024 * (EVENT_SIZE + 16))

int main() {
    int fd = inotify_init1(IN_CLOEXEC);
    if (fd == -1) {
        perror("inotify_init1");
        return 1;
    }

    int wd = inotify_add_watch(fd, "/tmp", IN_CREATE | IN_DELETE);
    if (wd == -1) {
        perror("inotify_add_watch");
        close(fd);
        return 1;
    }

    char buf[BUF_LEN];
    while (1) {
        ssize_t len = read(fd, buf, sizeof(buf));
        if (len == -1 && errno != EAGAIN) {
            perror("read");
            break;
        }
        if (len <= 0) continue;

        struct inotify_event *event = (struct inotify_event *)buf;
        while ((char *)event < buf + len) {
            if (event->mask & IN_CREATE)
                printf("Created: %s\n", event->name);
            else if (event->mask & IN_DELETE)
                printf("Deleted: %s\n", event->name);
            event = (struct inotify_event *)((char *)event + EVENT_SIZE + event->len);
        }
    }

    inotify_rm_watch(fd, wd);
    close(fd);
    return 0;
}
```

---
