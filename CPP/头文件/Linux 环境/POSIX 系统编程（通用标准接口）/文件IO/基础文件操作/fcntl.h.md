# `<fcntl.h>` 核心函数、结构体与宏详解（Linux / POSIX）

## 说明
`<fcntl.h>` 是 POSIX 标准定义的头文件，提供了文件控制操作和文件描述符管理功能。它包含了 `open`、`creat`、`fcntl` 等函数声明，以及文件打开标志、文件状态标志、文件锁等宏定义。本文档按照统一格式整理其中的函数、结构体、宏等信息，并说明内部原理与线程安全性。

---

## 一、函数详解

### 1. open 函数
**函数原型：**
```c
int open(const char *pathname, int flags, ... /* mode_t mode */);
```
**作用：** 打开或创建文件，返回文件描述符。  
**参数：**  
- `pathname`：文件路径。  
- `flags`：文件打开标志（必选：`O_RDONLY`、`O_WRONLY`、`O_RDWR` 之一，可选：`O_CREAT`、`O_TRUNC`、`O_APPEND`、`O_NONBLOCK` 等）。  
- `mode`：当指定 `O_CREAT` 时，用于设置文件权限（如 `0644`）。  
**返回值：** 成功返回非负文件描述符，失败返回 -1。  
**示例用法：**
```c
int fd = open("file.txt", O_RDWR | O_CREAT | O_TRUNC, 0644);
if (fd == -1) { perror("open"); exit(1); }
```
**实现原理：**  
1. 系统调用进入内核，根据路径名查找 inode。  
2. 检查进程权限是否满足 `flags` 所要求的访问模式。  
3. 分配新的文件描述符，创建文件表项（包含当前偏移量、文件状态标志等）。  
4. 若指定 `O_CREAT` 且文件不存在，则创建新文件并分配 inode。  
5. 返回文件描述符。  
**线程安全提示：** `open()` 是线程安全的，但多个线程同时打开同一文件可能导致不同描述符，无冲突。

---

### 2. creat 函数
**函数原型：**
```c
int creat(const char *pathname, mode_t mode);
```
**作用：** 创建并打开一个文件（等价于 `open(pathname, O_WRONLY | O_CREAT | O_TRUNC, mode)`）。  
**返回值：** 成功返回文件描述符，失败返回 -1。  
**示例用法：**
```c
int fd = creat("newfile.txt", 0644);
```
**实现原理：** 内部调用 `open` 实现，无额外逻辑。  
**线程安全提示：** 同 `open`。

---

### 3. fcntl 函数
**函数原型：**
```c
int fcntl(int fd, int cmd, ... /* arg */);
```
**作用：** 对打开的文件描述符执行控制操作，如复制描述符、获取/设置文件状态标志、获取/设置文件锁等。  
**参数：**  
- `fd`：文件描述符。  
- `cmd`：命令（见宏部分）。  
- `arg`：可选参数，取决于 `cmd`（如新文件描述符、文件状态标志、`struct flock *` 等）。  
**返回值：** 取决于 `cmd`，失败返回 -1。  
**示例用法：**
```c
// 复制文件描述符
int newfd = fcntl(oldfd, F_DUPFD, 0);

// 获取文件状态标志
int flags = fcntl(fd, F_GETFL);
// 设置非阻塞标志
flags |= O_NONBLOCK;
fcntl(fd, F_SETFL, flags);

// 设置写锁
struct flock lock = {
    .l_type = F_WRLCK,
    .l_whence = SEEK_SET,
    .l_start = 0,
    .l_len = 0,   // 锁定整个文件
};
fcntl(fd, F_SETLK, &lock);
```
**实现原理：**  
系统调用，内核根据 `cmd` 执行相应操作：  
- `F_DUPFD`：分配新的文件描述符指向同一文件表项。  
- `F_GETFD`/`F_SETFD`：获取/设置文件描述符标志（如 `FD_CLOEXEC`）。  
- `F_GETFL`/`F_SETFL`：获取/设置文件状态标志（如 `O_NONBLOCK`）。  
- `F_GETLK`/`F_SETLK`/`F_SETLKW`：查询、设置或等待文件锁（记录锁）。  
**线程安全提示：** `fcntl()` 是线程安全的。文件锁操作是系统级同步，多线程对同一文件设置锁时，内核会保证原子性。

---

## 二、结构体详解

### 1. struct flock
**定义：**
```c
struct flock {
    short l_type;    // 锁类型：F_RDLCK, F_WRLCK, F_UNLCK
    short l_whence;  // 偏移基准：SEEK_SET, SEEK_CUR, SEEK_END
    off_t l_start;   // 起始偏移量
    off_t l_len;     // 锁定长度（0 表示到文件末尾）
    pid_t l_pid;     // 持有锁的进程 ID（仅 F_GETLK 返回）
};
```
**作用：** 用于描述文件锁（记录锁），配合 `fcntl` 的 `F_GETLK`、`F_SETLK`、`F_SETLKW` 命令。  
**成员详解：**  
- `l_type`：锁类型，取值 `F_RDLCK`（共享读锁）、`F_WRLCK`（独占写锁）、`F_UNLCK`（解锁）。  
- `l_whence`：与 `lseek` 的 `whence` 相同，定义偏移量的起始位置。  
- `l_start`：相对于 `l_whence` 的起始偏移。  
- `l_len`：锁定的字节数，若为 0 则锁定从 `l_start` 到文件末尾。  
- `l_pid`：仅在 `F_GETLK` 时被内核填充，表示持有锁的进程 ID。  

**使用注意事项：**  
- 读锁与读锁共享，读锁与写锁互斥，写锁与任何锁互斥。  
- 进程退出时自动释放持有的所有锁。  
- 锁不会被子进程继承（`fork` 后子进程不继承锁）。

---

## 三、宏定义详解

### 1. 文件打开标志（用于 `open` 和 `fcntl` 的 `F_GETFL`/`F_SETFL`）
| 宏名称 | 作用 |
|--------|------|
| `O_RDONLY` | 只读打开 |
| `O_WRONLY` | 只写打开 |
| `O_RDWR` | 读写打开 |
| `O_CREAT` | 若文件不存在则创建 |
| `O_EXCL` | 与 `O_CREAT` 配合，若文件已存在则失败（原子创建） |
| `O_TRUNC` | 若文件存在且以写方式打开，则清空文件 |
| `O_APPEND` | 每次写入追加到文件末尾（原子操作） |
| `O_NONBLOCK` | 非阻塞模式（对管道、设备、套接字有效） |
| `O_SYNC` | 写操作同步到磁盘（`write` 返回前数据已落盘） |
| `O_DSYNC` | 同步数据，但不同步元数据（若不影响后续读取） |
| `O_RSYNC` | 读操作同步（与写同步结合） |
| `O_CLOEXEC` | 设置 `FD_CLOEXEC` 标志，在 `exec` 时自动关闭 |
| `O_DIRECT` | 直接 I/O（绕过页缓存，Linux 特有） |
| `O_DIRECTORY` | 若路径不是目录则失败 |
| `O_NOFOLLOW` | 若路径是符号链接则失败 |
| `O_TMPFILE` | 创建临时文件（Linux 3.11+） |

---

### 2. fcntl 命令（cmd）
| 宏名称 | 作用 |
|--------|------|
| `F_DUPFD` | 复制文件描述符，返回最小的未使用描述符，且不小于 `arg` |
| `F_DUPFD_CLOEXEC` | 复制文件描述符并设置 `FD_CLOEXEC` 标志 |
| `F_GETFD` | 获取文件描述符标志（当前仅 `FD_CLOEXEC`） |
| `F_SETFD` | 设置文件描述符标志 |
| `F_GETFL` | 获取文件状态标志（如 `O_APPEND`、`O_NONBLOCK`） |
| `F_SETFL` | 设置文件状态标志（仅允许修改 `O_APPEND`、`O_NONBLOCK` 等部分标志） |
| `F_GETLK` | 查询文件锁：若指定区域已被锁定，则通过 `struct flock` 返回锁信息，否则 `l_type` 设为 `F_UNLCK` |
| `F_SETLK` | 设置或释放文件锁（非阻塞），若锁冲突则立即返回 `-1`，`errno` 为 `EAGAIN` |
| `F_SETLKW` | 设置或释放文件锁（阻塞），若锁冲突则等待直到可用 |
| `F_GETOWN` | 获取接收 `SIGIO`/`SIGURG` 信号的进程/进程组 ID |
| `F_SETOWN` | 设置接收 `SIGIO`/`SIGURG` 信号的进程/进程组 ID |
| `F_GETSIG` | 获取发送信号（Linux 扩展） |
| `F_SETSIG` | 设置发送信号（Linux 扩展） |

---

### 3. 文件锁类型（用于 `struct flock.l_type`）
| 宏名称 | 作用 |
|--------|------|
| `F_RDLCK` | 共享读锁 |
| `F_WRLCK` | 独占写锁 |
| `F_UNLCK` | 解锁 |

---

### 4. 文件描述符标志（用于 `F_GETFD`/`F_SETFD`）
| 宏名称 | 作用 |
|--------|------|
| `FD_CLOEXEC` | 执行 `exec` 时关闭该文件描述符 |

---

### 5. 其他相关宏
| 宏名称 | 作用 |
|--------|------|
| `AT_FDCWD` | 用于 `*at` 系列函数（如 `openat`），表示相对于当前目录 |
| `AT_SYMLINK_NOFOLLOW` | 用于 `*at` 系列函数，不跟随符号链接 |
| `AT_REMOVEDIR` | 用于 `unlinkat`，删除目录 |
| `AT_EACCESS` | 用于 `faccessat`，使用有效用户 ID 检查权限 |

---

## 四、类型定义
- `mode_t`：文件权限类型（通常定义在 `<sys/types.h>`）。
- `off_t`：文件偏移量类型。
- `pid_t`：进程 ID 类型。
- `struct flock` 结构体。

---

## 五、模板声明
`<fcntl.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---