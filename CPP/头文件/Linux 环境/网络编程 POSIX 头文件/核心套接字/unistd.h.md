## `<unistd.h>` 头文件详解（Linux / POSIX）

`<unistd.h>` 是 POSIX 标准定义的系统编程核心头文件，提供了对操作系统 API 的访问，包括文件 I/O、进程控制、系统标识、用户信息、时间、环境等。它定义了大量的函数、宏和类型，是编写可移植 POSIX 程序的基础。

---

## 一、函数详解

### 1. read 函数

**函数原型：**
```c
ssize_t read(int fd, void *buf, size_t count);
```

**作用：** 从文件描述符 `fd` 中读取最多 `count` 字节数据到 `buf` 指向的缓冲区。

**参数：**
- `fd`：文件描述符（文件、管道、套接字、终端等）。
- `buf`：指向存储读取数据的缓冲区。
- `count`：要读取的最大字节数。

**返回值：**
- 成功：返回实际读取的字节数（0 表示到达文件末尾或对端关闭连接）。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
char buffer[1024];
ssize_t n = read(fd, buffer, sizeof(buffer) - 1);
if (n > 0) { buffer[n] = '\0'; printf("Read: %s\n", buffer); }
else if (n == 0) { printf("End of file\n"); }
else { perror("read"); }
```

**实现原理：**
系统调用。内核根据 fd 找到文件对象，调用对应驱动或文件系统的读方法，将数据从内核拷贝到用户空间。对于普通文件，从页缓存读取；对于套接字，从接收缓冲区读取。若数据不足且阻塞模式，则进程睡眠。

**线程安全提示：**
线程安全。多个线程同时读取同一 fd 时，内核保证每次 `read` 的原子性，但数据可能交叉，需应用层同步。

---

### 2. write 函数

**函数原型：**
```c
ssize_t write(int fd, const void *buf, size_t count);
```

**作用：** 将 `buf` 指向的缓冲区中的最多 `count` 字节写入文件描述符 `fd`。

**参数：**
- `fd`：文件描述符。
- `buf`：指向要写入的数据。
- `count`：要写入的字节数。

**返回值：**
- 成功：返回实际写入的字节数（可能小于 `count`）。
- 失败：返回 -1，并设置 `errno`。

**示例用法：**
```c
char *msg = "Hello\n";
ssize_t n = write(STDOUT_FILENO, msg, strlen(msg));
if (n == -1) perror("write");
```

**实现原理：**
系统调用。数据从用户空间拷贝到内核缓冲区（页缓存或套接字发送缓冲区）。普通文件异步写入磁盘；套接字由协议栈发送；管道写入管道缓冲区。若缓冲区满，则阻塞。

**线程安全提示：**
线程安全。多个线程同时写同一 fd 时，内核保证每次 `write` 的原子性（数据不交错），但顺序可能任意。对于普通文件，使用 `O_APPEND` 可保证写入都在末尾。

---

### 3. close 函数

**函数原型：**
```c
int close(int fd);
```

**作用：** 关闭文件描述符 `fd`，释放相关资源。当引用计数为零时，对于套接字和管道会触发连接终止。

**参数：**
- `fd`：要关闭的文件描述符。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
close(fd);
```

**实现原理：**
系统调用。减少文件描述符在内核中的引用计数；若归零，则释放文件对象，关闭设备或连接。对于 TCP，会发送 FIN 开始四次挥手（或根据 `SO_LINGER` 选项）。

**线程安全提示：**
线程安全，但并发关闭可能导致其他线程使用无效 fd，需同步。

---

### 4. lseek 函数

**函数原型：**
```c
off_t lseek(int fd, off_t offset, int whence);
```

**作用：** 重新定位文件描述符 `fd` 的读写位置（文件偏移量）。

**参数：**
- `fd`：文件描述符（必须支持随机访问，如普通文件）。
- `offset`：偏移量，相对于 `whence` 指定的位置。
- `whence`：基准位置，可取 `SEEK_SET`（文件开头）、`SEEK_CUR`（当前位置）、`SEEK_END`（文件末尾）。

**返回值：**
- 成功：返回新的文件偏移量（从文件开头计算）。
- 失败：返回 `(off_t)-1`，并设置 `errno`。

**示例用法：**
```c
off_t pos = lseek(fd, 0, SEEK_END);  // 获取文件大小
```

**实现原理：**
系统调用。修改文件对象中的当前偏移量字段。

**线程安全提示：**
线程安全，但多个线程同时操作同一 fd 的偏移量会相互干扰，建议使用 `pread`/`pwrite`。

---

### 5. pipe 函数

**函数原型：**
```c
int pipe(int pipefd[2]);
```

**作用：** 创建一个管道，用于进程间通信。`pipefd[0]` 用于读，`pipefd[1]` 用于写。

**参数：**
- `pipefd`：长度为 2 的整数数组，用于接收两个文件描述符。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
int fd[2];
if (pipe(fd) == -1) { perror("pipe"); exit(1); }
pid_t pid = fork();
if (pid == 0) {
    close(fd[1]);
    char buf[256];
    read(fd[0], buf, sizeof(buf));
    close(fd[0]);
} else {
    close(fd[0]);
    write(fd[1], "hello", 5);
    close(fd[1]);
}
```

**实现原理：**
内核创建管道对象（环形缓冲区），分配两个文件描述符指向它。写入的数据放入缓冲区，读取的数据从缓冲区取出。当缓冲区满时写入阻塞，空时读取阻塞。

**线程安全提示：**
`pipe()` 本身线程安全。创建后对管道的读写操作与普通文件 I/O 类似，多线程读写需同步。

---

### 6. dup / dup2 函数

**函数原型：**
```c
int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

**作用：**
- `dup`：复制文件描述符 `oldfd`，返回新的文件描述符（当前最小未使用值）。
- `dup2`：将 `oldfd` 复制到 `newfd`，若 `newfd` 已打开则先关闭它。

**返回值：**
- 成功：返回新文件描述符（`dup2` 返回 `newfd`）。
- 失败：返回 -1。

**示例用法：**
```c
int newfd = dup(oldfd);
dup2(oldfd, STDOUT_FILENO);  // 重定向标准输出
```

**实现原理：**
系统调用。在进程文件描述符表中复制表项，增加文件对象引用计数。`dup2` 会先关闭 `newfd` 指向的文件（若打开）。

**线程安全提示：**
线程安全，但 `dup2` 关闭 `newfd` 可能影响其他线程对该文件描述符的使用。

---

### 7. fork 函数

**函数原型：**
```c
pid_t fork(void);
```

**作用：** 创建一个子进程。子进程是父进程的副本，拥有独立的内存空间，但共享文件描述符等资源。

**返回值：**
- 成功：在父进程中返回子进程 PID；在子进程中返回 0。
- 失败：返回 -1。

**示例用法：**
```c
pid_t pid = fork();
if (pid == -1) { perror("fork"); exit(1); }
else if (pid == 0) {
    // 子进程代码
    execlp("/bin/ls", "ls", NULL);
} else {
    // 父进程代码
    wait(NULL);
}
```

**实现原理：**
系统调用。内核为子进程分配 PCB，复制父进程的页表（写时复制），复制文件描述符表（引用计数增加）。子进程获得新的 PID，父进程 ID 设为父进程的 PID。

**线程安全提示：**
`fork()` 本身线程安全，但在多线程程序中需注意：`fork` 后子进程中只有调用线程存在，其他线程不会复制。应在 `fork` 前确保互斥锁等资源状态正确，或使用 `pthread_atfork`。

---

### 8. exec 族函数

**函数原型（常见）：**
```c
int execl(const char *path, const char *arg, ...);
int execlp(const char *file, const char *arg, ...);
int execle(const char *path, const char *arg, ..., char *const envp[]);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execve(const char *path, char *const argv[], char *const envp[]);
```

**作用：** 将当前进程映像替换为新程序。`exec` 执行后，原进程代码、数据、堆栈被新程序覆盖，但进程 ID、文件描述符等保留（除非设置了 `FD_CLOEXEC`）。

**参数：**
- `path` / `file`：可执行文件路径（`exec*` 中若包含 `/` 则直接作为路径，否则在 `PATH` 中搜索）。
- `arg` / `argv`：参数列表，第一个参数通常是程序名，最后一个必须为 `NULL`。
- `envp`：环境变量数组。

**返回值：**
- 成功：不返回。
- 失败：返回 -1。

**示例用法：**
```c
execlp("ls", "ls", "-l", NULL);
perror("execlp");  // 只有失败才会执行到这里
```

**实现原理：**
系统调用 `execve` 是真正实现，其他 `exec` 函数是对它的封装。内核检查可执行文件格式，释放原进程的内存映射，加载新程序的段和共享库，初始化堆栈（参数、环境变量），设置程序入口点。

**线程安全提示：**
安全。调用 `exec` 后，原进程的所有线程都被销毁（仅保留调用线程），因此多线程程序在调用 `exec` 前应确保没有其他线程正在执行关键操作。建议在 `fork` 后立即 `exec`。

---

### 9. getpid / getppid 函数

**函数原型：**
```c
pid_t getpid(void);
pid_t getppid(void);
```

**作用：**
- `getpid`：返回当前进程的进程 ID。
- `getppid`：返回父进程的进程 ID。

**返回值：** 总是成功，返回相应的进程 ID。

**示例用法：**
```c
printf("PID: %d, PPID: %d\n", getpid(), getppid());
```

**实现原理：**
从进程控制块中读取相应字段，通常为内联函数或快速系统调用。

**线程安全提示：**
完全线程安全（只读）。

---

### 10. getuid / geteuid / getgid / getegid 函数

**函数原型：**
```c
uid_t getuid(void);      // 实际用户 ID
uid_t geteuid(void);     // 有效用户 ID
gid_t getgid(void);      // 实际组 ID
gid_t getegid(void);     // 有效组 ID
```

**作用：** 获取进程的用户 ID 和组 ID，用于权限检查。

**返回值：** 总是成功，返回对应的 ID。

**线程安全提示：** 完全线程安全。

---

### 11. alarm / pause / sleep / usleep 函数

**函数原型：**
```c
unsigned int alarm(unsigned int seconds);
int pause(void);
unsigned int sleep(unsigned int seconds);
int usleep(useconds_t usec);
```

**作用：**
- `alarm`：设置定时器，在 `seconds` 秒后向进程发送 `SIGALRM` 信号。
- `pause`：使进程挂起直到收到一个信号。
- `sleep`：使进程休眠指定的秒数，或直到被信号中断。
- `usleep`：微秒级休眠（已废弃，推荐 `nanosleep`）。

**示例用法：**
```c
alarm(5);
pause();   // 等待 SIGALRM
```

**实现原理：**
内核为进程维护定时器。`alarm` 设置定时器，到期时发送 `SIGALRM`。`pause` 将进程置于可中断睡眠状态。

**线程安全提示：**
安全，但 `pause` 挂起整个进程。多线程环境下推荐使用 `sigsuspend` 或 `nanosleep`。

---

### 12. fsync / fdatasync 函数

**函数原型：**
```c
int fsync(int fd);
int fdatasync(int fd);
```

**作用：**
- `fsync`：将文件描述符 `fd` 对应的文件所有修改过的数据和元数据同步到磁盘。
- `fdatasync`：只同步数据，不同步元数据（除非元数据对后续数据访问必要）。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
write(fd, buf, len);
fsync(fd);  // 确保数据落盘
```

**实现原理：**
内核将文件对应页缓存中的脏页写入磁盘，`fsync` 还会同步索引节点元数据。函数阻塞直到写入完成。

**线程安全提示：**
线程安全。多个线程可以同时同步不同文件描述符。

---

### 13. truncate / ftruncate 函数

**函数原型：**
```c
int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
```

**作用：** 将文件截断为指定长度（若原文件更大则截断，若更小则填充零）。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
truncate("file.txt", 1024);
ftruncate(fd, 2048);
```

**实现原理：**
修改文件 inode 中的文件大小字段。截断时释放超出部分的磁盘块；扩展时新区域填充为零。

**线程安全提示：**
安全，但多线程操作同一文件需外部同步。

---

### 14. chdir / fchdir / getcwd 函数

**函数原型：**
```c
int chdir(const char *path);
int fchdir(int fd);
char *getcwd(char *buf, size_t size);
```

**作用：**
- `chdir`：改变当前工作目录到 `path`。
- `fchdir`：改变当前工作目录到 `fd` 指向的目录。
- `getcwd`：获取当前工作目录的绝对路径。

**返回值：**
- `chdir`/`fchdir`：成功返回 0，失败返回 -1。
- `getcwd`：成功返回指向 `buf` 的指针，失败返回 NULL。

**示例用法：**
```c
chdir("/home/user");
char cwd[1024];
if (getcwd(cwd, sizeof(cwd)) != NULL) printf("%s\n", cwd);
```

**实现原理：**
`chdir` 修改进程控制块中的当前工作目录指针。`getcwd` 从内核中读取路径。

**线程安全提示：**
这些函数是线程安全的，但改变工作目录会影响整个进程的所有线程，多线程程序中使用需谨慎。`getcwd` 使用用户提供的缓冲区，安全。

---

### 15. unlink / rmdir 函数

**函数原型：**
```c
int unlink(const char *pathname);
int rmdir(const char *pathname);
```

**作用：**
- `unlink`：删除文件（减少链接计数，若计数为 0 则真正删除）。
- `rmdir`：删除空目录。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
unlink("temp.txt");
rmdir("emptydir");
```

**实现原理：**
从目录中移除文件名，减少 inode 的链接数。若链接数变为 0 且没有进程打开该文件，则释放磁盘空间。

**线程安全提示：**
安全，但多线程同时操作同一文件可能导致竞争。

---

### 16. isatty / ttyname 函数

**函数原型：**
```c
int isatty(int fd);
char *ttyname(int fd);
```

**作用：**
- `isatty`：判断文件描述符是否指向终端。
- `ttyname`：返回终端设备名。

**返回值：**
- `isatty`：是终端返回 1，否则返回 0。
- `ttyname`：成功返回设备名字符串（静态缓冲区），失败返回 NULL。

**示例用法：**
```c
if (isatty(STDIN_FILENO)) printf("%s\n", ttyname(STDIN_FILENO));
```

**实现原理：**
查询文件描述符对应的设备类型，检查主设备号是否为终端设备。

**线程安全提示：**
`isatty` 线程安全。`ttyname` 使用静态缓冲区，非线程安全，多线程应使用 `ttyname_r`。

---

### 17. gethostname / getdomainname 函数

**函数原型：**
```c
int gethostname(char *name, size_t len);
int getdomainname(char *name, size_t len);
```

**作用：**
- `gethostname`：获取主机名。
- `getdomainname`：获取域名。

**返回值：**
- 成功：返回 0。
- 失败：返回 -1。

**示例用法：**
```c
char hostname[256];
if (gethostname(hostname, sizeof(hostname)) == 0) printf("%s\n", hostname);
```

**实现原理：**
从内核的 utsname 结构中读取相应字段。

**线程安全提示：**
线程安全（写入用户提供的缓冲区）。

---

## 二、宏定义详解

### 1. 标准文件描述符宏

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `STDIN_FILENO` | 0 | 标准输入的文件描述符 |
| `STDOUT_FILENO` | 1 | 标准输出的文件描述符 |
| `STDERR_FILENO` | 2 | 标准错误的文件描述符 |

---

### 2. 文件访问模式宏（用于 `access`）

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `F_OK` | 0 | 测试文件是否存在 |
| `R_OK` | 4 | 测试读权限 |
| `W_OK` | 2 | 测试写权限 |
| `X_OK` | 1 | 测试执行权限 |

---

### 3. 文件位置基准宏（用于 `lseek`）

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `SEEK_SET` | 0 | 从文件开头计算偏移 |
| `SEEK_CUR` | 1 | 从当前位置计算偏移 |
| `SEEK_END` | 2 | 从文件末尾计算偏移 |

---

### 4. POSIX 版本和选项宏

这些宏用于条件编译检测系统特性，通常定义在 `<unistd.h>` 中。

| 宏名称 | 典型值 | 作用 |
|--------|--------|------|
| `_POSIX_VERSION` | 200809L | 支持的 POSIX 标准版本（如 200809L 表示 POSIX.1-2008） |
| `_POSIX2_VERSION` | 200809L | POSIX.2（Shell 和实用工具）版本 |
| `_POSIX_JOB_CONTROL` | 1 或 -1 | 是否支持作业控制 |
| `_POSIX_SAVED_IDS` | 1 或 -1 | 是否支持保存用户/组 ID |
| `_POSIX_CHOWN_RESTRICTED` | 1 或 -1 | `chown` 是否受限 |
| `_POSIX_NO_TRUNC` | 1 或 -1 | 长文件名是否截断 |
| `_POSIX_VDISABLE` | 1 或 -1 | 是否支持终端特殊字符禁用 |
| `_POSIX_THREADS` | 200809L 或 -1 | 是否支持线程 |
| `_POSIX_THREAD_SAFE_FUNCTIONS` | 200809L 或 -1 | 是否提供线程安全函数 |
| `_POSIX_FSYNC` | 200809L 或 -1 | 是否支持 `fsync` |
| `_POSIX_MAPPED_FILES` | 200809L 或 -1 | 是否支持内存映射文件 |
| `_POSIX_TIMERS` | 200809L 或 -1 | 是否支持定时器 |
| `_POSIX_SPIN_LOCKS` | 200809L 或 -1 | 是否支持自旋锁 |
| `_POSIX_READER_WRITER_LOCKS` | 200809L 或 -1 | 是否支持读写锁 |
| `_POSIX_MONOTONIC_CLOCK` | 200809L 或 -1 | 是否支持单调时钟 |

**使用示例：**
```c
#if _POSIX_VERSION >= 200809L
    // 使用 POSIX.1-2008 特性
#endif
```

---

### 5. 系统限制宏（用于 `sysconf`）

这些宏作为参数传递给 `sysconf` 查询系统配置值。

| 宏名称 | 作用 |
|--------|------|
| `_SC_ARG_MAX` | 参数列表最大长度 |
| `_SC_CHILD_MAX` | 每个用户最大进程数 |
| `_SC_CLK_TCK` | 每秒时钟滴答数 |
| `_SC_NPROCESSORS_ONLN` | 在线处理器数 |
| `_SC_OPEN_MAX` | 单个进程最多打开文件数 |
| `_SC_PAGE_SIZE` | 系统页大小（字节） |
| `_SC_STREAM_MAX` | 标准 I/O 流最大数 |
| `_SC_TZNAME_MAX` | 时区名最大长度 |
| `_SC_THREADS` | 是否支持线程 |
| `_SC_MAPPED_FILES` | 是否支持内存映射文件 |

**示例：**
```c
long page_size = sysconf(_SC_PAGE_SIZE);
```

---

### 6. 路径名限制宏（用于 `pathconf` / `fpathconf`）

| 宏名称 | 作用 |
|--------|------|
| `_PC_LINK_MAX` | 文件的最大硬链接数 |
| `_PC_MAX_CANON` | 终端规范输入行的最大长度 |
| `_PC_MAX_INPUT` | 终端输入缓冲区的最大长度 |
| `_PC_NAME_MAX` | 文件名的最大长度（不包括路径） |
| `_PC_PATH_MAX` | 路径名的最大长度 |
| `_PC_PIPE_BUF` | 管道的原子写入最大长度 |
| `_PC_CHOWN_RESTRICTED` | 是否对 `chown` 受限 |
| `_PC_NO_TRUNC` | 长文件名是否截断 |
| `_PC_VDISABLE` | 终端特殊字符禁用支持 |

**示例：**
```c
long max_name = pathconf("/tmp", _PC_NAME_MAX);
```

---

### 7. 其他宏

- `NULL`：空指针常量。
- `_POSIX_C_SOURCE`：编译时指定的 POSIX 版本（如 `-D_POSIX_C_SOURCE=200809L`）。
- `_XOPEN_SOURCE`：指定 X/Open 标准版本。
- `_GNU_SOURCE`：Linux 扩展，启用大量非标准函数。

---

## 三、类型定义

`<unistd.h>` 通过包含 `<sys/types.h>` 定义了以下常用类型（部分列举）：

- `ssize_t`：有符号整数，用于字节数或错误。
- `pid_t`：进程 ID。
- `uid_t`、`gid_t`：用户/组 ID。
- `off_t`：文件偏移量。
- `size_t`：无符号整数，用于对象大小。

---

## 四、模板声明

`<unistd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
