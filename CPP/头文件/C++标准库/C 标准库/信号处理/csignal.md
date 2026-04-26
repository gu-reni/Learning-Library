## `<csignal>` 头文件详解

`<csignal>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**信号处理**功能。它允许程序注册信号处理函数、发送信号以及执行信号相关的操作。信号是操作系统发送给进程的异步通知，用于通知异常事件（如段错误、中断）或用户自定义事件。该头文件为 C++ 程序提供了与 C 相同的信号处理接口。

---

## 一、函数详解

### 1. `std::signal`

**函数原型：**
```cpp
void (*signal(int sig, void (*handler)(int)))(int);
```

**作用：** 设置指定信号 `sig` 的处理方式。成功时返回之前的处理函数指针，失败时返回 `SIG_ERR`。

**参数：**
- `sig`：信号编号（如 `SIGINT`、`SIGTERM`）。
- `handler`：处理方式，可以是：
  - `SIG_DFL`：恢复默认处理。
  - `SIG_IGN`：忽略该信号。
  - 用户自定义函数：`void handler(int)`。

**返回值：** 成功返回之前设置的信号处理函数指针（或 `SIG_DFL` / `SIG_IGN`），失败返回 `SIG_ERR`。

**示例用法：**
```cpp
#include <csignal>
#include <cstdio>

void sigint_handler(int sig) {
    std::printf("Caught signal %d\n", sig);
}

int main() {
    std::signal(SIGINT, sigint_handler);
    // ... 后续代码
    return 0;
}
```

**实现原理：** 系统调用（POSIX `sigaction` 的简化包装）。内核修改进程的 `sigaction` 表项，记录新的处理方式。

**线程安全提示：** `signal()` 函数本身是线程安全的，但它的行为在不同系统上可能有差异（例如是否自动重置处理函数）。多线程环境中推荐使用更细粒度的 `sigaction()`（POSIX）。

---

### 2. `std::raise`

**函数原型：**
```cpp
int raise(int sig);
```

**作用：** 向当前进程发送信号 `sig`。

**参数：** 信号编号。

**返回值：** 成功返回 `0`，失败返回非零。

**示例用法：**
```cpp
std::raise(SIGUSR1);
```

**实现原理：** 等价于 `kill(getpid(), sig)`。内核将该信号递送给当前进程。

**线程安全提示：** 线程安全。信号可能被当前进程的任何线程处理。

---

## 二、宏定义详解

### 1. 信号编号常量（常见）

这些宏的值由实现定义，但含义如下：

| 宏名 | 作用 |
|------|------|
| `SIGABRT` | 异常终止（`abort` 调用） |
| `SIGFPE` | 浮点异常（除零、溢出等） |
| `SIGILL` | 非法指令 |
| `SIGINT` | 中断（通常为 Ctrl+C） |
| `SIGSEGV` | 段错误（无效内存访问） |
| `SIGTERM` | 终止请求（默认终止） |
| `SIG_DFL` | 默认信号处理方式 |
| `SIG_IGN` | 忽略信号 |
| `SIG_ERR` | `signal()` 返回的错误值 |

### 2. 其他可能的信号（取决于平台）

POSIX 还定义了 `SIGHUP`、`SIGQUIT`、`SIGKILL` 等，但标准 C++ 仅保证上述几个存在。

---

## 三、类型定义

| 类型 | 说明 |
|------|------|
| `sig_atomic_t` | 可以原子访问的整数类型，用于信号处理函数中与主程序通信的共享变量。 |

---

## 四、结构体详解

`<csignal>` 不定义任何结构体。

---

## 五、模板声明

`<csignal>` 是纯 C 风格头文件，不包含 C++ 模板。

---
