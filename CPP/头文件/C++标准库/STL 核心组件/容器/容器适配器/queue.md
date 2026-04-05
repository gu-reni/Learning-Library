## `<queue>` 头文件（`std::queue`）详解

`<queue>` 是 C++ 标准模板库（STL）中定义的头文件，提供了容器适配器 `std::queue`。`std::queue` 是一个**先进先出**（FIFO）队列，只允许在队尾插入元素（`push`），从队首移除元素（`pop`），并支持访问队首和队尾元素。它默认基于 `std::deque` 实现，也可指定其他底层容器（如 `std::list`）。

---

## 一、模板参数

```cpp
template< class T, class Container = std::deque<T> >
class queue;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素的类型 |
| `Container` | 底层容器类型，必须满足序列容器（`deque`、`list` 等）的要求，且支持 `push_back()`、`pop_front()`、`front()`、`back()`、`size()`、`empty()` 等操作，默认为 `std::deque<T>` |

**注意：** `std::queue` 不提供迭代器，也不支持随机访问，仅支持队列特定的操作。

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `value_type` | `T` |
| `size_type` | `Container::size_type`（通常为 `size_t`） |
| `reference` | `Container::reference`（通常为 `T&`） |
| `const_reference` | `Container::const_reference`（通常为 `const T&`） |
| `container_type` | `Container`（底层容器类型） |

**注意：** `queue` 没有迭代器类型（`iterator`、`const_iterator` 等）。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `queue()` | 默认构造一个空队列，使用默认构造的底层容器 |
| `explicit queue(const Container& cont)` | 使用给定的底层容器副本构造队列 |
| `explicit queue(Container&& cont)` | 使用移动的底层容器构造队列（C++11） |
| `queue(const queue& other)` | 拷贝构造 |
| `queue(queue&& other)` | 移动构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~queue()` | 销毁队列及其元素（调用底层容器的析构函数） |

**示例：**
```cpp
std::queue<int> q1;                         // 空队列
std::deque<int> d = {1,2,3};
std::queue<int> q2(d);                      // 从 deque 拷贝构造
std::queue<int> q3(std::move(d));           // 从 deque 移动构造
std::queue<int> q4(q2);                     // 拷贝构造
```

**实现原理：**
- 默认构造时，底层容器被默认构造。
- 拷贝/移动构造分别复制/移动底层容器。

**线程安全提示：**
构造和析构应在单线程环境进行，或有锁保护。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const queue& other)` | 拷贝赋值 |
| `operator=(queue&& other)` | 移动赋值（C++11） |

**示例：**
```cpp
std::queue<int> q1, q2;
q1 = q2;   // 拷贝赋值
```

**实现原理：**
- 拷贝赋值时，底层容器被复制。
- 移动赋值时，直接接管底层容器的资源。

**线程安全提示：**
并发赋值需要同步。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `reference front()` | 返回队首元素的引用（队列不能为空） |
| `const_reference front() const` | 常量版本 |
| `reference back()` | 返回队尾元素的引用（队列不能为空） |
| `const_reference back() const` | 常量版本 |

**示例：**
```cpp
std::queue<int> q;
q.push(10);
q.push(20);
q.push(30);
int a = q.front();   // 10
int b = q.back();    // 30
```

**实现原理：**
- 直接调用底层容器的 `front()` 和 `back()`。

**时间复杂度：** O(1)

**线程安全提示：**
只读操作，若其他线程同时修改队列则不安全。并发读取是安全的（仅当没有写入）。

---

### 4. 容量

| 函数 | 说明 |
|------|------|
| `bool empty() const` | 检查队列是否为空 |
| `size_type size() const` | 返回元素个数 |

**示例：**
```cpp
std::queue<int> q;
bool e = q.empty();   // true
size_t s = q.size();  // 0
```

**实现原理：**
- 直接调用底层容器的 `empty()` 和 `size()`。

**时间复杂度：** O(1)

**线程安全提示：**
只读，安全。

---

### 5. 修改器

| 函数 | 说明 |
|------|------|
| `void push(const T& value)` | 在队尾插入元素（复制） |
| `void push(T&& value)` | 在队尾插入元素（移动）（C++11） |
| `template<class... Args> void emplace(Args&&... args)` | 在队尾原位构造元素（C++11） |
| `void pop()` | 移除队首元素（队列不能为空） |
| `void swap(queue& other)` | 交换内容（C++11） |

**示例：**
```cpp
std::queue<int> q;
q.push(10);          // 队尾插入 10
q.emplace(20);       // 队尾构造 20
q.pop();             // 移除 10，现在队首是 20
```

**实现原理：**
- `push(value)`：调用底层容器的 `push_back(value)`。
- `emplace`：调用底层容器的 `emplace_back(args...)`。
- `pop()`：调用底层容器的 `pop_front()`。
- `swap`：调用底层容器的 `swap`。

**时间复杂度：**
- `push` / `emplace` / `pop`：取决于底层容器，通常为 O(1)。
- `swap`：O(1)（如果底层容器的 `swap` 是 O(1)，如 `deque` 和 `list`）。

**线程安全提示：**
所有修改操作都需要同步。

---

### 6. 底层容器访问（C++11 起）

| 函数 | 说明 |
|------|------|
| `const Container& container() const` | 返回底层容器的常量引用（C++11） |

**示例：**
```cpp
std::queue<int> q;
q.push(1);
const auto& c = q.container();   // 获取底层容器（如 deque）
```

**实现原理：**
- 直接返回底层容器对象。

**线程安全提示：**
只读，但若其他线程修改队列，则返回值可能过时。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `swap(queue& lhs, queue& rhs)` | 交换两个队列的内容（C++11） |
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个队列的内容（C++20 起使用三路比较，之前版本可能不支持） |

**示例：**
```cpp
std::queue<int> q1, q2;
std::swap(q1, q2);
```

---

## 五、宏与常量

`<queue>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::queue` 是一个容器适配器，它封装了底层容器并提供队列接口。典型实现包含一个成员：

```cpp
Container c;   // 底层容器（默认 deque）
```

所有操作都转发到底层容器：
- `push` → `c.push_back()`
- `pop` → `c.pop_front()`
- `front` → `c.front()`
- `back` → `c.back()`
- `empty` → `c.empty()`
- `size` → `c.size()`

**底层容器的要求：**
- 支持 `push_back()`、`pop_front()`、`front()`、`back()`、`size()`、`empty()`。
- 随机访问和迭代器不是必需的。
- 常见可选底层容器：`std::deque`（默认）、`std::list`。

**性能特点：**
- 无额外开销（仅底层容器）。
- 所有操作时间复杂度与底层容器相同。

---

## 七、时间复杂度

| 操作 | 时间复杂度（默认 deque） |
|------|--------------------------|
| 构造（空） | O(1) |
| `empty()`、`size()` | O(1) |
| `front()`、`back()` | O(1) |
| `push()` / `emplace()` | O(1) |
| `pop()` | O(1) |
| `swap()` | O(1) |

---

## 八、线程安全

**`std::queue` 不是线程安全的**。规则与 `priority_queue` 类似：
- **同时读取**：多个线程同时调用 `empty()`、`size()`、`front()`、`back()` 是安全的（只读）。
- **读取 + 写入**：一个线程执行 `push()`、`pop()` 时，其他线程不能进行任何操作（包括 `front()`），否则数据竞争。
- **同时写入**：多个线程同时修改队列会导致未定义行为。

**保证线程安全的常用方法：**
- 使用互斥锁（`std::mutex`）保护所有操作。
- 使用线程局部存储的队列。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace` 原位构造。
- `container()` 访问底层容器。
- 右值引用版本的 `push`。
- `swap` 成员函数和非成员函数。

### C++14
- 无显著新特性。

### C++17
- 添加推导指引（deduction guides），允许从底层容器自动推导模板参数。
- `swap` 被标记为 `noexcept` 若底层容器的 `swap` 也为 `noexcept`。

### C++20
- 比较操作符支持（通过底层容器的比较）。

---

## 十、迭代器失效规则

`std::queue` **不提供迭代器**，因此没有直接的迭代器失效问题。但需要注意：
- 对队列的修改（`push`、`pop`）可能使由 `container()` 返回的底层容器的迭代器失效（依据底层容器的规则）。
- 对队列的 `swap` 不会使任何外部引用失效，但交换后两个队列的内容互换。

---

## 十一、相关结构体与函数

| 组件 | 说明 |
|------|------|
| `std::deque<T>` | 默认底层容器 |
| `std::list<T>` | 可作为底层容器的备选 |
| `std::stack` | 另一容器适配器（LIFO） |

---
