## `<linux/fs.h>` 头文件详解（Linux 内核接口）

`<linux/fs.h>` 是 Linux 内核文件系统子系统的核心头文件，定义了内核中文件、索引节点、超级块、目录项等关键数据结构及其操作函数，还包含了设备号管理、文件系统控制等相关的宏定义。该头文件是编写 Linux 设备驱动程序所必须包含的，许多重要的函数和数据结构均在其中定义[reference:0]。以下将介绍其中的重要函数、结构体、宏及类型。

---

## 一、函数详解

### 1. `filp_open()`

**函数原型：**
```c
struct file *filp_open(const char *filename, int open_mode, int mode);
```
**作用：** 在内核空间中打开一个文件，返回一个指向 `struct file` 的指针，供后续操作使用[reference:1]。

**参数：**
- `filename`：文件的路径（包括文件名）。
- `open_mode`：文件的打开方式，如 `O_RDWR`、`O_CREAT` 等，与用户空间的 `open()` 系统调用的 `flags` 参数类似。
- `mode`：创建文件时使用的权限（例如 `0644`），其它情况下可设为 `0`。

**返回值：** 成功返回 `struct file*` 指针，失败时需要使用 `IS_ERR()` 宏来检验其有效性，而不是与 `NULL` 进行比较[reference:2]。

**示例用法：**
```c
#include <linux/fs.h>

struct file *fp = filp_open("/home/1.txt", O_RDWR | O_CREAT, 0644);
if (IS_ERR(fp)) {
    printk("create file error\n");
    return -1;
}
// 后续操作...
filp_close(fp, NULL);
```
**实现原理：** `filp_open()` 在内核中通过调用 `do_filp_open()` 和 `path_openat()` 等函数，最终执行 `do_dentry_open()`。它依据传入的路径名，查找并解析目录项和挂载点，最后分配一个新的 `struct file` 对象并返回给调用者[reference:3]。

**线程安全提示：** 在内核中，`struct file` 结构体本身可能被多个线程共享，但对该结构体的使用（如调用 `vfs_write()`）应当确保不会引起数据竞争。

---

### 2. `filp_close()`

**函数原型：**
```c
int filp_close(struct file *filp, fl_owner_t id);
```
**作用：** 关闭一个在内核中已打开的文件。

**参数：**
- `filp`：由 `filp_open()` 返回的文件指针。
- `id`：通常传入 `NULL`，也可以传入 `current->files` 作为实参[reference:4]。

**返回值：** 成功时返回 0，失败时返回负的错误码。

**示例用法：**
```c
filp_close(fp, NULL);
```
**实现原理：** `filp_close()` 调用 `fput()` 减少文件的引用计数，并在引用计数降到零时调用 `__fput()` 释放资源。如果文件描述符被标记为 `O_CLOEXEC`，在 `exec` 时也会自动关闭。

**线程安全提示：** `filp` 可能被多个线程共享，调用 `filp_close()` 时需要确保该 `struct file` 不再被使用，以免出现 `use-after-free` 的问题。

---

### 3. `vfs_read()`

**函数原型：**
```c
ssize_t vfs_read(struct file *filp, char __user *buffer, size_t len, loff_t *pos);
```
**作用：** 从打开的文件中读取数据，基于文件当前的偏移量。

**参数：**
- `filp`：指向打开的文件对象的指针。
- `buffer`：指向用户空间缓冲区的指针（带有 `__user` 标记）。
- `len`：要读取的字节数。
- `pos`：指定读取的起始位置，该值会被更新为读取后的新位置。

**返回值：** 成功时返回实际读取的字节数，失败时返回负的错误码。

**示例用法：**
```c
loff_t pos = 0;
char buf[256];
ssize_t ret = vfs_read(fp, buf, sizeof(buf), &pos);
```
**实现原理：** 该函数是系统调用 `read()` 在内核中的具体实现。它首先检查 `filp` 是否允许读操作，然后调用 `file_operations` 中的 `read` 方法（如 `filp->f_op->read`）。如果该方法未提供，则尝试调用 `read_iter` 接口，最后由具体的文件系统或设备驱动完成实际的数据传输。

**线程安全提示：** 对同一个 `struct file` 对象并发执行 `vfs_read` 和 `vfs_write` 会导致文件偏移量(`pos`)和数据竞争，通常需要使用文件锁或确保单线程访问。

---

### 4. `vfs_write()`

**函数原型：**
```c
ssize_t vfs_write(struct file *filp, const char __user *buffer, size_t len, loff_t *pos);
```
**作用：** 向打开的文件中写入数据，基于文件当前的偏移量。

**参数：**
- `filp`：指向打开的文件对象的指针。
- `buffer`：指向用户空间缓冲区的指针。
- `len`：要写入的字节数。
- `pos`：指定写入的起始位置，该值会被更新为写入后的新位置。

**返回值：** 成功时返回实际写入的字节数，失败时返回负的错误码。

**示例用法：**
```c
loff_t pos = 0;
char *data = "hello";
ssize_t ret = vfs_write(fp, data, strlen(data), &pos);
```
**实现原理：** 与 `vfs_read` 类似，调用 `file_operations` 中的 `write` 方法。如果未提供 `write` 方法，则回退到 `write_iter` 接口。

**线程安全提示：** 对同一个 `struct file` 对象并发执行 `vfs_read` 和 `vfs_write` 会导致文件偏移量和数据竞争，通常需要使用文件锁或确保单线程访问。

---

### 5. `set_fs()` 与 `get_fs()`（仅适用于旧版内核，内核 5.x 后已移除）

**函数原型：**
```c
mm_segment_t get_fs(void);
void set_fs(mm_segment_t fs);
```
**作用：** `get_fs()` 返回当前内核线程的地址空间限制，`set_fs()` 用于临时修改该限制，例如改为 `KERNEL_DS`（内核数据段）以允许内核空间地址作为系统调用的参数。在较新的内核（5.x 及更高版本）中，这些函数已经被移除，不再推荐使用。

**参数：**
- `fs`：地址空间限制，通常是 `USER_DS` 或 `KERNEL_DS`。

**示例用法（旧内核）：**
```c
mm_segment_t old_fs = get_fs();
set_fs(KERNEL_DS);
// 在此区域调用 vfs_read/vfs_write 等函数，传递内核空间的指针
set_fs(old_fs);
```
**实现原理：** 在 x86 架构中，`set_fs()` 会修改当前内核线程的 `addr_limit`。当系统调用检查参数是否位于用户空间时，会参考这个限制。若修改为 `KERNEL_DS`，则允许内核地址作为参数传递。

**线程安全提示：** `set_fs()` 修改的是当前内核线程的地址限制，不会影响其他线程，但恢复原限制非常重要，否则可能导致安全漏洞。

---

### 6. `register_chrdev_region()` / `alloc_chrdev_region()` / `unregister_chrdev_region()`

**函数原型：**
```c
int register_chrdev_region(dev_t first, unsigned int count, const char *name);
int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, const char *name);
void unregister_chrdev_region(dev_t first, unsigned int count);
```
**作用：** 用于字符设备驱动的设备号注册和释放[reference:5]。

**参数：**
- `first`：要注册的设备号起始值。
- `count`：要注册的连续设备编号个数。
- `name`：设备名称，会出现在 `/proc/devices` 中。
- `dev`：`alloc_chrdev_region()` 动态分配的设备号存储位置。
- `firstminor`：动态分配时请求的起始次设备号。

**返回值：** 注册成功返回 0，失败返回负的错误码。

**实现原理：** 内核维护一个全局的设备号链表，这些函数通过 `__register_chrdev_region` 在链表中查找空闲区域并分配。

**线程安全提示：** 内核的 `chrdevs` 数组在注册和注销时使用互斥锁保护，因此这些函数是线程安全的。

---

### 7. `register_chrdev()` / `unregister_chrdev()`（已过时）

**函数原型：**
```c
int register_chrdev(unsigned int major, const char *name, struct file_operations *fops);
int unregister_chrdev(unsigned int major, const char *name);
```
**作用：** 这是 Linux 2.6 之前的旧版字符设备注册函数，在 2.6 内核中已被模拟但仍不建议新代码使用[reference:6]。

**参数：**
- `major`：主设备号（若为 0，则动态分配）。
- `name`：设备名称。
- `fops`：指向 `file_operations` 结构体的指针。

**返回值：** 成功时返回 0，失败时返回负的错误码。

**实现原理：** 该函数在内部调用 `__register_chrdev_region` 分配设备号，并将 `fops` 注册到 `chrdevs` 数组中，一次注册 0-255 共 256 个次设备号。

**线程安全提示：** 与上述注册函数类似，是线程安全的。

---

## 二、结构体详解

### 1. `struct file`

**定义：**
```c
struct file {
    union {
        struct llist_node fu_llist;
        struct rcu_head fu_rcuhead;
    } f_u;
    struct path f_path;
    struct inode *f_inode;
    const struct file_operations *f_op;
    spinlock_t f_lock;
    atomic_long_t f_count;
    unsigned int f_flags;
    fmode_t f_mode;
    struct mutex f_pos_lock;
    loff_t f_pos;
    struct fown_struct f_owner;
    const struct cred *f_cred;
    struct file_ra_state f_ra;
    u64 f_version;
#ifdef CONFIG_SECURITY
    void *f_security;
#endif
    void *private_data;
#ifdef CONFIG_EPOLL
    struct list_head f_ep_links;
    struct list_head f_tfile_llink;
#endif
    struct address_space *f_mapping;
} __attribute__((aligned(4)));
```
**作用：** 代表一个进程打开的文件。系统中每个打开的文件在内核空间都有一个关联的 `struct file`[reference:7][reference:8]。该结构体在 `open()` 系统调用时被创建，在 `close()` 时被销毁。

**成员详解：**
- `f_u`：联合体，用于管理链表的节点或 RCU 资源。
- `f_path`：包含 `dentry` 和 `mnt` 成员，用于确定文件路径。
- `f_inode`：指向该文件对应的 inode 结构体。
- `f_op`：指向该文件的操作函数表，通常为 `file_operations` 结构体。
- `f_lock`：保护 `f_flags` 和 `f_ep_links` 的自旋锁。
- `f_count`：文件的引用计数，表示有多少个进程打开了该文件。
- `f_flags`：打开文件时指定的标志，如 `O_RDONLY`、`O_NONBLOCK` 等。
- `f_mode`：文件的读写模式，如 `FMODE_READ`、`FMODE_WRITE`。
- `f_pos_lock`：保护 `f_pos` 的互斥锁。
- `f_pos`：当前文件偏移量（读写位置）。
- `f_owner`：用于通过信号进行 I/O 事件通知的数据。
- `f_cred`：打开该文件时的凭证信息。
- `f_ra`：文件预读状态。
- `f_version`：文件的版本号，每次使用后递增。
- `f_security`：安全相关数据，在启用安全模块时使用。
- `private_data`：驱动可用的私有数据指针，常用于设备驱动存储上下文信息。
- `f_ep_links`：用于 epoll 的链表头。
- `f_mapping`：指向该文件的地址空间对象，用于管理文件页缓存。

---

### 2. `struct file_operations`

**定义：**
```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
    ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
    int (*open) (struct inode *, struct file *);
    int (*release) (struct inode *, struct file *);
    int (*flush) (struct file *, fl_owner_t id);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
    int (*mmap) (struct file *, struct vm_area_struct *);
    unsigned long mmap_supported_flags;
    // ... 许多其他方法
} __randomize_layout;
```
**作用：** 包含一组函数指针，用于操作一个已打开的文件。这个结构体是设备驱动程序必须实现的核心接口之一，也是 VFS 与文件系统或设备驱动之间的纽带[reference:9][reference:10]。

**成员详解：**
- `owner`：指向拥有此结构体的模块，通常初始化为 `THIS_MODULE`，防止模块在使用时被卸载。
- `llseek`：修改文件读写位置的函数，对应 `lseek()` 系统调用。
- `read`：从文件读取数据的函数，对应 `read()` 系统调用。
- `write`：向文件写入数据的函数，对应 `write()` 系统调用。
- `read_iter`：异步读取数据的函数，用于高性能 I/O。
- `write_iter`：异步写入数据的函数。
- `open`：打开文件时调用的函数，对应 `open()` 系统调用。
- `release`：关闭文件时调用的函数，对应 `close()` 系统调用。
- `flush`：刷新文件缓冲区的函数。
- `unlocked_ioctl`：设备控制函数，对应 `ioctl()` 系统调用。
- `compat_ioctl`：用于 32 位程序在 64 位内核上运行的 `ioctl`。
- `mmap`：内存映射函数，对应 `mmap()` 系统调用。

---

### 3. `struct inode`

**定义：**（结构体很大，仅列出关键成员）
```c
struct inode {
    umode_t i_mode;
    unsigned short i_opflags;
    kuid_t i_uid;
    kgid_t i_gid;
    unsigned int i_flags;
    const struct inode_operations *i_op;
    struct super_block *i_sb;
    struct address_space *i_mapping;
    struct list_head i_list;
    struct list_head i_sb_list;
    union {
        struct hlist_head i_dentry;
        struct rcu_head i_rcu;
    };
    u64 i_version;
    atomic_t i_count;
    atomic_t i_dio_count;
    atomic_t i_writecount;
    const struct file_operations *i_fop;
    struct file_lock_context *i_flctx;
    struct address_space i_data;
    struct list_head i_devices;
    union {
        struct pipe_inode_info *i_pipe;
        struct block_device *i_bdev;
        struct cdev *i_cdev;
        char *i_link;
        unsigned i_dir_seq;
    };
    __u32 i_generation;
    loff_t i_size;
    struct timespec64 i_atime;
    struct timespec64 i_mtime;
    struct timespec64 i_ctime;
    // ... 其他成员
};
```
**作用：** 表示文件系统中的文件或目录的元数据（metadata），每个文件/目录都有一个唯一的 inode[reference:11]。当多个进程打开同一个文件时，它们共享同一个 `inode` 对象，但拥有各自独立的 `struct file` 对象[reference:12]。

**成员详解：**
- `i_mode`：文件的类型和访问权限。
- `i_uid`、`i_gid`：文件的所有者和所属组。
- `i_flags`：文件标志，如 `S_IMMUTABLE`。
- `i_op`：指向 inode 操作函数表（`inode_operations`）的指针。
- `i_sb`：指向所属超级块的指针。
- `i_mapping`：指向地址空间的指针，用于管理文件页缓存。
- `i_count`：inode 的引用计数。
- `i_writecount`：文件的写入者计数，用于实现 `O_EXCL` 等语义。
- `i_fop`：默认的文件操作函数表。
- `i_data`：文件数据在内存中的地址空间结构。
- `i_pipe`、`i_bdev`、`i_cdev`：联合体，用于不同文件类型（管道、块设备、字符设备）的特定信息。
- `i_size`：文件大小（字节）。
- `i_atime`、`i_mtime`、`i_ctime`：访问时间、修改时间和状态变更时间。

---

### 4. `struct super_block`

**定义：**（结构体很大，仅列出关键成员）
```c
struct super_block {
    struct list_head s_list;
    dev_t s_dev;
    unsigned char s_blocksize_bits;
    unsigned long s_blocksize;
    loff_t s_maxbytes;
    struct file_system_type *s_type;
    const struct super_operations *s_op;
    const struct dquot_operations *dq_op;
    const struct quotactl_ops *s_qcop;
    unsigned long s_flags;
    unsigned long s_magic;
    struct dentry *s_root;
    struct rw_semaphore s_umount;
    struct mutex s_lock;
    int s_count;
    atomic_t s_active;
    struct list_head s_inodes;
    struct list_head s_files;
    struct list_head s_dentry_lru;
    int s_nr_dentry_unused;
    struct block_device *s_bdev;
    struct backing_dev_info *s_bdi;
    struct list_head s_instances;
    struct quota_info s_dquot;
    char s_id[32];
    void *s_fs_info;
    fmode_t s_mode;
    u32 s_time_gran;
    // ... 其他成员
} __randomize_layout;
```
**作用：** 表示一个已挂载的文件系统实例，每个挂载的文件系统都有一个超级块对象[reference:13]。在文件系统被挂载时创建，卸载时销毁[reference:14]。

**成员详解：**
- `s_list`：用于将所有超级块链接成链表。
- `s_dev`：该文件系统所在的设备标识符。
- `s_blocksize`：文件系统的块大小（字节）。
- `s_maxbytes`：该文件系统支持的最大文件大小。
- `s_type`：指向该文件系统类型（`struct file_system_type`）的指针。
- `s_op`：指向超级块操作函数表（`super_operations`）的指针。
- `s_magic`：文件系统的魔数，用于识别文件系统类型（例如 `EXT4_SUPER_MAGIC`）。
- `s_root`：文件系统的根目录项。
- `s_umount`：保护卸载操作的读写信号量。
- `s_count`：超级块的引用计数。
- `s_active`：活跃引用计数，用于延迟释放。
- `s_inodes`：该文件系统中所有 inode 的链表。
- `s_files`：该文件系统中所有打开文件的链表。
- `s_bdev`：指向该文件系统所在的块设备对象。
- `s_id`：文件系统的名称（如 `sda1`）。
- `s_fs_info`：文件系统私有数据指针，具体文件系统（如 ext4、xfs）将自己的私有数据存储于此。
- `s_time_gran`：时间戳精度（纳秒）。

---

### 5. `struct dentry`

**定义：**（在 `<linux/dcache.h>` 中定义，此处仅列出框架）
```c
struct dentry {
    struct dentry *d_parent;
    struct qstr d_name;
    struct inode *d_inode;
    unsigned char d_iname[DNAME_INLINE_LEN];
    struct list_head d_child;
    struct list_head d_subdirs;
    struct hlist_node d_hash;
    struct list_head d_lru;
    // ... 其他成员
};
```
**作用：** 表示一个目录项，将文件名与其 inode 关联起来。目录项对象没有对应的磁盘结构，VFS 会根据路径名在内存中现场创建，用于路径查找和缓存[reference:15]。

**成员详解：**
- `d_parent`：指向父目录项的指针。
- `d_name`：目录项的名称。
- `d_inode`：与该目录项关联的 inode 对象。
- `d_hash`：用于目录项缓存的哈希链表节点。
- `d_lru`：用于 LRU 缓存的链表节点。

---

### 6. `struct super_operations`

**定义：**（包含多个函数指针）
```c
struct super_operations {
    struct inode *(*alloc_inode)(struct super_block *sb);
    void (*destroy_inode)(struct inode *);
    void (*free_inode)(struct inode *);
    void (*dirty_inode)(struct inode *, int flags);
    int (*write_inode)(struct inode *, struct writeback_control *wbc);
    int (*drop_inode)(struct inode *);
    void (*evict_inode)(struct inode *);
    void (*put_super)(struct super_block *);
    int (*sync_fs)(struct super_block *sb, int wait);
    int (*statfs)(struct dentry *, struct kstatfs *);
    int (*remount_fs)(struct super_block *, int *, char *);
    void (*umount_begin)(struct super_block *);
    int (*show_options)(struct seq_file *, struct dentry *);
    // ... 其他成员
};
```
**作用：** 超级块的操作函数表，包含了对文件系统整体进行管理的方法，如 inode 分配、同步文件系统、卸载文件系统等[reference:16]。

---

### 7. `struct inode_operations`

**定义：**（包含多个函数指针）
```c
struct inode_operations {
    struct dentry * (*lookup) (struct inode *,struct dentry *, unsigned int);
    const char * (*get_link) (struct dentry *, struct inode *, struct delayed_call *);
    int (*permission) (struct inode *, int);
    int (*create) (struct inode *,struct dentry *, umode_t, bool);
    int (*link) (struct dentry *,struct inode *,struct dentry *);
    int (*unlink) (struct inode *,struct dentry *);
    int (*symlink) (struct inode *,struct dentry *,const char *);
    int (*mkdir) (struct inode *,struct dentry *, umode_t);
    int (*rmdir) (struct inode *,struct dentry *);
    int (*rename) (struct inode *, struct dentry *, struct inode *, struct dentry *, unsigned int);
    // ... 其他成员
};
```
**作用：** inode 的操作函数表，包含了针对文件或目录进行操作的方法，如创建、删除、重命名文件等[reference:17]。

---

### 8. `struct address_space`

**定义：**（部分成员）
```c
struct address_space {
    struct inode *host;
    struct xarray i_pages;
    gfp_t gfp_mask;
    atomic_t i_mmap_writable;
    struct rb_root i_mmap;
    unsigned long nrpages;
    unsigned long nrexceptional;
    unsigned long writeback_index;
    const struct address_space_operations *a_ops;
    unsigned long flags;
    spinlock_t private_lock;
    void *private_data;
    // ... 其他成员
};
```
**作用：** 用于管理文件的页缓存（page cache），负责将文件数据缓存在内存中，并提供读写操作接口[reference:18]。

---

## 三、宏定义详解

### 1. 设备号操作宏（定义于 `<linux/kdev_t.h>`）

| 宏名称 | 作用 |
|--------|------|
| `MAJOR(dev_t dev)` | 从设备号中提取主设备号[reference:19] |
| `MINOR(dev_t dev)` | 从设备号中提取次设备号[reference:20] |
| `MKDEV(major, minor)` | 根据主次设备号组合成 `dev_t` 类型[reference:21] |

**示例用法：**
```c
dev_t dev = MKDEV(250, 0);
int major = MAJOR(dev);  // 250
int minor = MINOR(dev);  // 0
```

---

### 2. 文件操作标志

| 宏名称 | 作用 |
|--------|------|
| `O_RDONLY` | 只读打开 |
| `O_WRONLY` | 只写打开 |
| `O_RDWR` | 读写打开 |
| `O_CREAT` | 如果文件不存在，则创建它 |
| `O_EXCL` | 与 `O_CREAT` 一起使用，如果文件已存在则返回错误 |
| `O_TRUNC` | 如果文件存在，且以写方式打开，则将其长度截断为 0 |
| `O_APPEND` | 每次写入操作都追加到文件末尾 |
| `O_NONBLOCK` | 非阻塞模式 |
| `O_SYNC` | 写操作同步到磁盘 |
| `O_CLOEXEC` | 在执行 `exec()` 时关闭文件描述符 |
| `O_DIRECT` | 直接 I/O，绕过页缓存 |
| `O_LARGEFILE` | 支持大文件（64 位偏移量） |

---

### 3. 文件模式掩码（用于 `struct inode` 的 `i_mode`）

| 宏名称 | 作用 |
|--------|------|
| `S_IFMT` | 文件类型位掩码 |
| `S_IFREG` | 常规文件 |
| `S_IFDIR` | 目录 |
| `S_IFCHR` | 字符设备 |
| `S_IFBLK` | 块设备 |
| `S_IFIFO` | FIFO（命名管道） |
| `S_IFLNK` | 符号链接 |
| `S_IFSOCK` | 套接字 |

**示例用法：**
```c
if ((inode->i_mode & S_IFMT) == S_IFCHR) {
    // 这是一个字符设备文件
}
```

---

### 4. 文件权限宏

| 宏名称 | 作用 |
|--------|------|
| `S_IRUSR` | 文件所有者读权限 |
| `S_IWUSR` | 文件所有者写权限 |
| `S_IXUSR` | 文件所有者执行权限 |
| `S_IRGRP` | 组读权限 |
| `S_IWGRP` | 组写权限 |
| `S_IXGRP` | 组执行权限 |
| `S_IROTH` | 其他用户读权限 |
| `S_IWOTH` | 其他用户写权限 |
| `S_IXOTH` | 其他用户执行权限 |
| `S_ISUID` | Set-User-ID 位 |
| `S_ISGID` | Set-Group-ID 位 |
| `S_ISVTX` | 粘滞位 |

---

### 5. 文件系统相关的 ioctl 命令（定义于 `<linux/fs.h>`）

| 宏名称 | 作用 |
|--------|------|
| `FS_IOC_GETFLAGS` | 获取文件的属性标志 |
| `FS_IOC_SETFLAGS` | 设置文件的属性标志 |
| `FS_IOC_GETVERSION` | 获取文件的版本号 |
| `FS_IOC_SETVERSION` | 设置文件的版本号 |
| `FS_IOC_FSGETXATTR` | 获取文件的扩展属性 |
| `FS_IOC_FSSETXATTR` | 设置文件的扩展属性 |

---

### 6. 文件系统类型相关宏

| 宏名称 | 作用 |
|--------|------|
| `FS_REQUIRES_DEV` | 文件系统需要块设备支持 |
| `FS_BINARY_MOUNTDATA` | 挂载数据是二进制格式 |
| `FS_HAS_SUBTYPE` | 文件系统有子类型 |
| `FS_USERNS_MOUNT` | 允许在用户命名空间中挂载 |
| `FS_DISALLOW_NOTIFY_PERM` | 不允许在权限不足时进行通知 |
| `FS_ALLOW_IDMAP` | 允许使用 ID 映射 |
| `FS_THP_SUPPORT` | 支持透明大页 |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `dev_t` | 设备号类型，用于表示主设备号和次设备号的组合[reference:22] |
| `loff_t` | 文件偏移量类型，用于支持大文件（64 位） |
| `size_t` | 无符号整数，用于表示对象大小 |
| `ssize_t` | 有符号整数，用于表示字节数或错误 |
| `uid_t` / `gid_t` | 用户 ID 和组 ID 类型 |
| `mode_t` | 文件模式类型，用于表示权限和文件类型 |
| `fmode_t` | 文件模式标志类型（如 `FMODE_READ`、`FMODE_WRITE`） |
| `fl_owner_t` | 文件锁所有者类型 |

---

## 五、模板声明

`<linux/fs.h>` 是 Linux 内核专用的 C 头文件，不包含 C++ 模板，且该头文件仅适用于内核空间编程，不能直接在用户空间应用程序中使用。

---
