# 并发库（thread、atomic、future 等）（C++11，C++14/17/20 增强）

**一句话定义**：C++ 并发库提供了一套跨平台的多线程编程组件，包括线程管理（`std::thread`）、同步原语（`std::mutex`、`std::atomic`）、异步任务（`std::future`/`std::promise`）等，使开发者能够以类型安全、可移植的方式编写并发程序。

---

## 详细语法与用法

### 1. `std::thread` – 线程管理

```cpp
#include <thread>
#include <iostream>

// C++11 基本线程
void func(int x) {
    std::cout << "Value: " << x << std::endl;
}

std::thread t(func, 42);
t.join();  // 等待线程结束
// t.detach();  // 分离线程（慎用）

// 使用 lambda
std::thread t2([](int a, int b) {
    std::cout << a + b << std::endl;
}, 3, 4);
t2.join();

// 获取硬件并发数
unsigned int n = std::thread::hardware_concurrency();
```

### 2. `std::mutex` 与锁管理

```cpp
#include <mutex>

std::mutex mtx;
int shared_data = 0;

void increment() {
    mtx.lock();
    ++shared_data;
    mtx.unlock();  // 必须确保解锁，否则死锁
}

// 使用 std::lock_guard（RAII，C++11）
void safe_increment() {
    std::lock_guard<std::mutex> lock(mtx);
    ++shared_data;
}   // 自动解锁

// std::unique_lock（更灵活，可延迟加锁、条件变量等）
void flexible_lock() {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    // 做一些非临界区操作
    lock.lock();   // 手动加锁
    ++shared_data;
    lock.unlock(); // 手动解锁
    // ...
}

// 死锁避免：同时锁多个 mutex
std::mutex m1, m2;
void transfer() {
    std::lock(m1, m2);                    // 一次性锁住两个
    std::lock_guard<std::mutex> lock1(m1, std::adopt_lock);
    std::lock_guard<std::mutex> lock2(m2, std::adopt_lock);
    // 临界区操作
}
```

### 3. `std::atomic` – 原子操作

```cpp
#include <atomic>

// C++11 原子类型
std::atomic<int> counter(0);

void increment_atomic() {
    counter++;   // 原子自增（fetch_add 的简化）
}

// 原子操作 API
int expected = 3;
int desired = 5;
counter.compare_exchange_strong(expected, desired);  // 若等于 expected 则替换

// 内存顺序（memory order）控制
std::atomic<bool> flag(false);
flag.store(true, std::memory_order_release);
bool val = flag.load(std::memory_order_acquire);

// 原子智能指针（C++20）
std::atomic<std::shared_ptr<int>> atomic_sp;
```

### 4. `std::future` / `std::promise` – 异步任务

```cpp
#include <future>
#include <chrono>

// 使用 std::async 创建异步任务（C++11）
int compute(int x) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

auto future = std::async(std::launch::async, compute, 5);
int result = future.get();  // 阻塞直到结果就绪

// 使用 std::promise 与 std::future
std::promise<int> prom;
std::future<int> fut = prom.get_future();

std::thread t([&prom] {
    prom.set_value(42);   // 设置值
});
int value = fut.get();    // 获取值
t.join();

// 共享 future（C++11 的 std::shared_future）
std::shared_future<int> shared_fut = fut.share();
// 多个线程可以等待同一个结果
```

### 5. 条件变量 `std::condition_variable`

```cpp
#include <condition_variable>

std::mutex cv_mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(cv_mtx);
    cv.wait(lock, []{ return ready; });  // 等待条件满足
    // 条件满足后继续
}

void notifier() {
    {
        std::lock_guard<std::mutex> lock(cv_mtx);
        ready = true;
    }
    cv.notify_one();   // 唤醒一个等待线程
}
```

### 6. 信号量（C++20）

```cpp
#include <semaphore>

std::counting_semaphore<10> sem(5);  // 最多 10 个计数，初始 5
sem.acquire();   // P 操作，计数减 1，若为 0 则阻塞
sem.release();   // V 操作，计数加 1
```

### 7. `std::jthread`（C++20）

```cpp
#include <thread>

// 自动 join 的线程，支持停止令牌
std::jthread jt([](std::stop_token stoken) {
    while (!stoken.stop_requested()) {
        // 工作循环
    }
});
// 析构时自动调用 join()
// 可请求停止
jt.request_stop();
```

---

## 底层原理与实现机制

### 1. `std::thread` 的实现

`std::thread` 是对底层操作系统线程句柄（如 POSIX `pthread_t` 或 Windows `HANDLE`）的 RAII 封装。构造时调用 `pthread_create` 或 `CreateThread`，将可调用对象和参数打包并传递给线程入口函数。`join()` 调用 `pthread_join` 或 `WaitForSingleObject` 等待线程结束。`detach()` 将线程分离，不再关联。

### 2. 内存模型与原子操作

C++11 引入了**内存模型**，定义了多线程环境下内存访问的可见性和顺序约束。原子操作通过硬件提供的原子指令（如 x86 的 `LOCK` 前缀、ARM 的 `LDREX/STREX`）或编译器屏障实现。

**内存顺序（Memory Order）**：
- `memory_order_relaxed`：仅保证原子性，无顺序保证。
- `memory_order_acquire`：后续读写不可重排到该操作之前。
- `memory_order_release`：之前读写不可重排到该操作之后。
- `memory_order_acq_rel`：同时满足 acquire 和 release。
- `memory_order_seq_cst`：顺序一致性（默认，最强）。

### 3. `std::future` 与 `std::promise` 的共享状态

`future`/`promise` 通过一个**共享状态（shared state）** 通信。该状态是一个堆上分配的对象，包含：
- 结果值或异常
- 就绪标志
- 等待的线程列表（条件变量）

`promise::set_value` 将值写入共享状态并通知所有等待的 `future`。`future::get` 在状态未就绪时阻塞（或等待超时）。

`std::async` 的策略：
- `std::launch::async`：强制创建新线程执行。
- `std::launch::deferred`：延迟到 `get()` 或 `wait()` 时在当前线程执行。
- 默认策略（`async|deferred`）：由实现选择，通常使用异步执行。

### 4. `std::mutex` 的底层

标准库的 `mutex` 通常封装操作系统提供的互斥体（如 pthread_mutex_t 或 CRITICAL_SECTION）。`lock` 调用系统调用（如 `futex`）可能导致线程阻塞并让出 CPU；`try_lock` 尝试原子测试并设置锁，失败则立即返回。`std::mutex` 通常是非递归的（同一个线程重复 lock 导致未定义行为），递归需求使用 `std::recursive_mutex`。

### 5. 条件变量的等待与唤醒

`condition_variable::wait` 原子地释放锁并将线程加入等待队列，然后阻塞。被唤醒后重新获取锁并检查条件。`notify_one` 唤醒一个等待线程，`notify_all` 唤醒全部。为了避免虚假唤醒，必须循环检查条件（通常使用带谓词的 `wait` 重载）。

---

## 与相关特性的对比

### 1. `std::thread` vs `std::jthread`（C++20）

| 特性 | `std::thread` | `std::jthread` |
|------|---------------|----------------|
| 析构行为 | 若未 join 或 detach，调用 `std::terminate` | 自动 `join()` |
| 停止机制 | 无 | 内置 `std::stop_token` / `request_stop()` |
| 异常安全 | 需手动管理 join | RAII 自动管理 |
| 适用场景 | 明确知道何时 join | 简化异常安全和协作取消 |

### 2. `std::atomic` vs `std::mutex` + 普通变量

| 方面 | `std::atomic` | `std::mutex` + 普通变量 |
|------|---------------|------------------------|
| 粒度 | 单个操作（通常一条指令） | 代码块（多条语句） |
| 开销 | 极低（无锁） | 较高（系统调用、上下文切换） |
| 适用场景 | 计数器、标志位、简单读写 | 复杂数据结构更新 |
| 阻塞 | 无阻塞（但可能自旋） | 阻塞线程 |
| 组合操作 | 支持 CAS、fetch_add 等原语 | 可任意组合 |

### 3. `std::async` vs 手动创建 `std::thread` + `std::promise`

| 方面 | `std::async` | 手动 `thread` + `promise` |
|------|--------------|---------------------------|
| 代码简洁性 | 高 | 低（需管理线程和 promise） |
| 异常传播 | 自动 | 手动捕获并 `set_exception` |
| 控制粒度 | 有限（仅策略选择） | 完全控制（线程池、优先级等） |
| 返回值获取 | 通过 `future::get` | 同样 |
| 适用场景 | 简单异步任务 | 需要精细线程管理或复用线程 |

---

## 常见面试问题与解答

### 问题 1：`std::thread` 没有 `join` 或 `detach` 会怎样？

**答**：若 `std::thread` 对象析构时仍处于 **joinable** 状态（即未调用 `join` 或 `detach`），标准规定调用 `std::terminate()` 终止程序。这是为了防止线程在 RAII 对象销毁后仍然运行，可能导致资源泄漏或访问已销毁对象。解决方案：始终在 `thread` 对象销毁前 `join` 或 `detach`，或使用 C++20 的 `std::jthread`。

---

### 问题 2：`std::atomic` 是否保证所有操作都是无锁的？

**答**：不保证。`std::atomic` 是否无锁取决于类型和平台。对于整型和指针类型，主流平台通常实现为无锁（lock-free）。对于自定义类型，如果其大小较小且对齐合适，也可能无锁，否则可能内部使用互斥锁。可通过 `atomic<T>::is_always_lock_free`（C++17）或 `atomic<T>::is_lock_free()` 查询。无锁意味着不使用互斥锁，但可能使用自旋等机制，仍可能因重试而耗时。

---

### 问题 3：`std::future` 的 `get()` 只能调用一次，如果想多次获取结果怎么办？

**答**：使用 `std::shared_future`。`shared_future` 允许多次 `get()`（返回常量引用或副本）。可以从 `future` 通过 `share()` 构造，也可以直接创建 `promise::get_future().share()`。`shared_future` 可拷贝，多个对象共享同一共享状态。

```cpp
std::promise<int> prom;
std::shared_future<int> sf = prom.get_future().share();
// 多个线程可以调用 sf.get()
```

---

### 问题 4：条件变量为什么需要配合 `mutex`？为什么 `wait` 需要传入锁？

**答**：条件变量本身不提供原子性。典型的使用模式：
1. 检查条件（如 `ready` 标志）必须在锁保护下进行。
2. `wait` 操作必须原子地释放锁并进入等待状态，否则可能出现：线程检查条件不满足后、进入等待前，另一线程修改了条件并发出通知，导致该线程永远阻塞。

`wait` 传入的 `unique_lock` 会在等待时原子解锁，被唤醒后重新锁定。配合谓词的重载自动处理虚假唤醒。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：忘记 `join` 或 `detach` 导致 `std::terminate`

```cpp
void bad() {
    std::thread t([]{});   // 未 join 也未 detach，析构时 terminate
}
```

**正确做法**：使用 `join` 或 `detach`，或使用 `std::jthread`。

---

#### 错误 2：数据竞争：共享变量不加锁或不用原子

```cpp
int counter = 0;
void increment() { ++counter; }   // 多线程调用导致数据竞争 UB
```

**正确做法**：使用 `std::atomic<int>` 或 `std::mutex`。

---

#### 错误 3：死锁 – 同一线程重复 lock 非递归 mutex

```cpp
std::mutex m;
m.lock();
m.lock();   // 未定义行为，通常死锁
```

**正确做法**：使用 `std::recursive_mutex` 或避免重复加锁。

---

#### 错误 4：条件变量虚假唤醒未检查条件

```cpp
cv.wait(lock);   // 可能被虚假唤醒，此时条件可能不成立
```

**正确做法**：使用带谓词的 `wait`：
```cpp
cv.wait(lock, []{ return ready; });
```

---

#### 错误 5：`std::async` 默认策略导致不创建新线程

```cpp
auto fut = std::async(compute, 5);   // 可能延迟执行（deferred）
fut.wait();  // 如果延迟，可能不创建新线程，在当前线程调用 compute
```

**正确做法**：显式指定 `std::launch::async` 强制异步。

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **优先使用 RAII 锁（`lock_guard`/`unique_lock`）** | 避免手动 lock/unlock，保证异常安全 |
| **使用 `std::atomic` 代替 volatile** | `volatile` 不保证原子性和内存顺序 |
| **使用 `std::jthread` 代替 `std::thread`** | 自动 join 和停止支持，更安全（C++20） |
| **为条件变量使用谓词重载** | 自动处理虚假唤醒 |
| **显式指定 `std::async` 策略** | 避免默认策略的不确定性 |
| **使用 `std::shared_future` 多次取值** | 或通过 `std::future::share()` |
| **尽量减少锁的粒度** | 只保护必要的数据，避免长时间持有锁 |
| **使用无锁编程仅当性能瓶颈且充分验证** | 无锁编程复杂，容易出错 |

```cpp
// 好的示例：使用 std::async 并指定策略
auto fut = std::async(std::launch::async, heavy_computation);
int result = fut.get();

// 好的示例：使用 std::jthread 自动管理生命周期
std::jthread t([](std::stop_token st) {
    while (!st.stop_requested()) {
        // 工作
    }
});
```