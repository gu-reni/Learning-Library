## `<sys/mount.h>` 头文件详解（Linux / POSIX）

`<sys/mount.h>` 是 Linux 系统提供的头文件，用于挂载和卸载文件系统。它定义了 `mount()` 和 `umount()`/`umount2()` 系统调用的参数结构体、标志位以及相关常量。该头文件是进行文件系统管理（如挂载磁盘、绑定挂载、共享子树等）的核心接口。

---

## 一、函数详解

### 1. `mount`

**函数原型：**
```c
int mount(const char *source, const char *target,
          const char *filesystemtype, unsigned long mountflags,
          const void *data);
```

**作用：** 将一个文件系统（块设备、目录或伪文件系统）挂载到指定的挂载点。

**参数：**
- `source`：源文件系统的标识（例如块设备路径 `/dev/sda1`、目录路径用于绑定挂载，或伪文件系统类型如 `"none"`）。
- `target`：挂载点目录路径（必须已存在）。
- `filesystemtype`：文件系统类型字符串（如 `"ext4"`, `"xfs"`, `"proc"`, `"tmpfs"` 等）。
- `mountflags`：挂载标志位掩码（见下文宏定义）。
- `data`：文件系统特定的参数，通常为 `NULL`，但某些文件系统（如 `tmpfs`）可接受以逗号分隔的选项字符串。

**返回值：** 成功返回 `0`，失败返回 `-1` 并设置 `errno`。

**示例用法：**
```c
#include <sys/mount.h>
#include <stdio.h>

int main() {
    if (mount("/dev/sdb1", "/mnt", "ext4", 0, NULL) == -1) {
        perror("mount");
        return 1;
    }
    return 0;
}
```

**实现原理：** 系统调用进入内核，内核查找对应的文件系统驱动，调用其 `mount` 方法，将源与目标关联到 VFS（虚拟文件系统）的全局挂载树中。

**线程安全提示：** `mount()` 系统调用是线程安全的，但挂载点路径的并发操作（如同时挂载/卸载）可能导致竞争，需由应用层同步。

---

### 2. `umount` / `umount2`

**函数原型：**
```c
int umount(const char *target);
int umount2(const char *target, int flags);
```

**作用：** 卸载挂载点 `target` 上的文件系统。`umount2` 支持额外的标志控制卸载行为。

**参数：**
- `target`：挂载点路径。
- `flags`：卸载标志，可取 `0` 或 `MNT_FORCE`、`MNT_DETACH`、`MNT_EXPIRE` 等（见宏定义）。

**返回值：** 成功返回 `0`，失败返回 `-1` 并设置 `errno`。

**示例用法：**
```c
if (umount2("/mnt", MNT_DETACH) == -1) perror("umount2");
```

**实现原理：** 系统调用减少 VFS 挂载项的引用计数，若不再被使用则从挂载树中移除，并通知文件系统驱动执行卸载清理。

**线程安全提示：** 同 `mount`。

---

## 二、结构体详解

### `struct mount_attr`（Linux 5.12+）

**定义：**
```c
struct mount_attr {
    __u64 attr_set;      // 要设置的属性掩码
    __u64 attr_clr;      // 要清除的属性掩码
    __u64 propagation;   // 传播类型
    __u64 userns_fd;     // 用户命名空间文件描述符
};
```

**作用：** 用于 `mount_setattr()` 系统调用（Linux 5.12+），提供更精细的挂载属性控制（如只读绑定、传播类型、用户命名空间等）。该结构体属于较新的 API，传统 `<sys/mount.h>` 可能未包含，但部分系统已支持。

**成员详解：**
- `attr_set` / `attr_clr`：指定要设置或清除的挂载属性（如 `MOUNT_ATTR_RDONLY`）。
- `propagation`：挂载传播类型（如 `MS_SHARED`, `MS_PRIVATE` 等）。
- `userns_fd`：用户命名空间文件描述符。

---

## 三、宏定义详解

### 1. 挂载标志（用于 `mountflags`）

| 宏名称 | 作用 |
|--------|------|
| `MS_RDONLY` | 以只读方式挂载 |
| `MS_NOSUID` | 禁止 set-user-ID 和 set-group-ID 位生效 |
| `MS_NODEV` | 禁止访问设备文件 |
| `MS_NOEXEC` | 禁止执行程序 |
| `MS_SYNCHRONOUS` | 同步写操作（立即刷盘） |
| `MS_REMOUNT` | 重新挂载（修改现有挂载的标志） |
| `MS_MANDLOCK` | 允许强制锁（已废弃） |
| `MS_DIRSYNC` | 目录同步更新 |
| `MS_NOATIME` | 不更新访问时间 |
| `MS_NODIRATIME` | 不更新目录访问时间 |
| `MS_BIND` | 创建绑定挂载（使目录树在另一位置可见） |
| `MS_MOVE` | 将子挂载树移动到新位置 |
| `MS_REC` | 递归挂载（配合 `MS_BIND` 或传播类型使用） |
| `MS_SILENT` | 抑制内核日志 |
| `MS_POSIXACL` | 启用 POSIX ACL |
| `MS_UNBINDABLE` | 标记挂载点为不可绑定 |
| `MS_PRIVATE` | 设置为私有传播类型 |
| `MS_SLAVE` | 设置为从属传播类型 |
| `MS_SHARED` | 设置为共享传播类型 |
| `MS_RELATIME` | 仅在 mtime/ctime 发生变化或 atime 早于它们时更新 atime |
| `MS_KERNMOUNT` | 内核内部挂载（用户态不可用） |
| `MS_I_VERSION` | 更新 inode 版本号 |
| `MS_STRICTATIME` | 总是更新 atime |
| `MS_LAZYTIME` | 延迟更新时间戳（减少磁盘写入） |
| `MS_NOSEC` | 不检查安全标记 |

---

### 2. 卸载标志（用于 `umount2`）

| 宏名称 | 作用 |
|--------|------|
| `MNT_FORCE` | 强制卸载（即使有文件在使用，可能导致数据丢失） |
| `MNT_DETACH` | 延迟卸载（lazy unmount），从挂载树中隐藏，待无引用时彻底清理 |
| `MNT_EXPIRE` | 标记挂载点为“过期”，若未被使用则卸载 |
| `UMOUNT_NOFOLLOW` | 若 `target` 是符号链接，则不跟随（Linux 5.14+） |

---

### 3. 传播类型常量

| 宏名称 | 作用 |
|--------|------|
| `MS_SHARED` | 挂载点的挂载/卸载事件会传播到对等组 |
| `MS_PRIVATE` | 不传播事件 |
| `MS_SLAVE` | 接收来自主机的传播，但自身不向外传播 |
| `MS_UNBINDABLE` | 类似私有，且不能用于绑定挂载 |

---

### 4. 挂载属性常量（用于 `struct mount_attr`）

| 宏名称 | 作用 |
|--------|------|
| `MOUNT_ATTR_RDONLY` | 只读 |
| `MOUNT_ATTR_NOSUID` | 禁止 setuid |
| `MOUNT_ATTR_NODEV` | 禁止设备访问 |
| `MOUNT_ATTR_NOEXEC` | 禁止执行 |
| `MOUNT_ATTR_RELATIME` | 相对时间更新 |
| `MOUNT_ATTR_NOATIME` | 不更新访问时间 |
| `MOUNT_ATTR_STRICTATIME` | 严格更新时间 |
| `MOUNT_ATTR_NODIRATIME` | 不更新目录访问时间 |
| `MOUNT_ATTR_IDMAP` | 使用用户命名空间映射 ID |

---

### 5. 其他常量

- `MS_RMT_MASK`：用于内核内部掩码，用户态无需关心。
- `MS_VERBOSE`：旧版本标志，现已废弃。

---

## 四、类型定义

- `struct mount_attr`（Linux 5.12+）如上。
- 无其他专有类型。

---

## 五、模板声明

`<sys/mount.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
