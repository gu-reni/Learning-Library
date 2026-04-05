## `<stack>` 头文件详解

`<stack>` 是 C++ 标准模板库（STL）中定义的头文件，提供了容器适配器 `std::stack`。`std::stack` 是一个**后进先出**（LIFO）栈，只允许在栈顶进行插入（`push`）和删除（`pop`），并支持访问栈顶元素（`top`）。它默认基于 `std::deque` 实现，也可指定其他底层容器（如 `std::vector`、`std::list`）。

---

## 一、模板参数

```cpp
template< class T, class Container = std::deque<T> >
class stack;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素的类型 |
| `Container` | 底层容器类型，必须满足序列容器（`deque`、`vector`、`list` 等）的要求，且支持 `push_back()`、`pop_back()`、`back()`、`size()`、`empty()` 等操作，默认为 `std::deque<T>` |

**注意：** `std::stack` 不提供迭代器，也不支持随机访问，仅支持栈特定的操作。

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `value_type` | `T` |
| `size_type` | `Container::size_type`（通常为 `size_t`） |
| `reference` | `Container::reference`（通常为 `T&`） |
| `const_reference` | `Container::const_reference`（通常为 `const T&`） |
| `container_type` | `Container`（底层容器类型） |

**注意：** `stack` 没有迭代器类型（`iterator`、`const_iterator` 等）。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `stack()` | 默认构造一个空栈，使用默认构造的底层容器 |
| `explicit stack(const Container& cont)` | 使用给定的底层容器副本构造栈 |
| `explicit stack(Container&& cont)` | 使用移动的底层容器构造栈（C++11） |
| `stack(const stack& other)` | 拷贝构造 |
| `stack(stack&& other)` | 移动构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~stack()` | 销毁栈及其元素（调用底层容器的析构函数） |

**示例：**
```cpp
std::stack<int> s1;                         // 空栈
std::vector<int> v = {1,2,3};
std::stack<int, std::vector<int>> s2(v);    // 从 vector 拷贝构造
std::stack<int, std::vector<int>> s3(std::move(v)); // 从 vector 移动构造
std::stack<int> s4(s1);                     // 拷贝构造
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
| `operator=(const stack& other)` | 拷贝赋值 |
| `operator=(stack&& other)` | 移动赋值（C++11） |

**示例：**
```cpp
std::stack<int> s1, s2;
s1 = s2;   // 拷贝赋值
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
| `reference top()` | 返回栈顶元素的引用（栈不能为空） |
| `const_reference top() const` | 常量版本 |

**示例：**
```cpp
std::stack<int> s;
s.push(10);
s.push(20);
int a = s.top();   // 20（最后压入的）
```

**实现原理：**
- 直接调用底层容器的 `back()`。

**时间复杂度：** O(1)

**线程安全提示：**
只读操作，若其他线程同时修改栈则不安全。并发读取是安全的（仅当没有写入）。

---

### 4. 容量

| 函数 | 说明 |
|------|------|
| `bool empty() const` | 检查栈是否为空 |
| `size_type size() const` | 返回元素个数 |

**示例：**
```cpp
std::stack<int> s;
bool e = s.empty();   // true
size_t sz = s.size(); // 0
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
| `void push(const T& value)` | 在栈顶插入元素（复制） |
| `void push(T&& value)` | 在栈顶插入元素（移动）（C++11） |
| `template<class... Args> void emplace(Args&&... args)` | 在栈顶原位构造元素（C++11） |
| `void pop()` | 移除栈顶元素（栈不能为空） |
| `void swap(stack& other)` | 交换内容（C++11） |

**示例：**
```cpp
std::stack<int> s;
s.push(10);          // 栈顶插入 10
s.emplace(20);       // 栈顶构造 20
s.pop();             // 移除 20，现在栈顶是 10
```

**实现原理：**
- `push(value)`：调用底层容器的 `push_back(value)`。
- `emplace`：调用底层容器的 `emplace_back(args...)`。
- `pop()`：调用底层容器的 `pop_back()`。
- `swap`：调用底层容器的 `swap`。

**时间复杂度：**
- `push` / `emplace` / `pop`：取决于底层容器，通常为 O(1)。
- `swap`：O(1)（如果底层容器的 `swap` 是 O(1)，如 `deque`、`vector`、`list`）。

**线程安全提示：**
所有修改操作都需要同步。

---

### 6. 底层容器访问（C++11 起）

| 函数 | 说明 |
|------|------|
| `const Container& container() const` | 返回底层容器的常量引用（C++11） |

**示例：**
```cpp
std::stack<int> s;
s.push(1);
const auto& c = s.container();   // 获取底层容器（如 deque）
```

**实现原理：**
- 直接返回底层容器对象。

**线程安全提示：**
只读，但若其他线程修改栈，则返回值可能过时。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `swap(stack& lhs, stack& rhs)` | 交换两个栈的内容（C++11） |
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个栈的内容（C++20 起使用三路比较，之前版本可能不支持） |

**示例：**
```cpp
std::stack<int> s1, s2;
std::swap(s1, s2);
```

---

## 五、宏与常量

`<stack>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::stack` 是一个容器适配器，它封装了底层容器并提供栈接口。典型实现包含一个成员：

```cpp
Container c;   // 底层容器（默认 deque）
```

所有操作都转发到底层容器：
- `push` → `c.push_back()`
- `pop` → `c.pop_back()`
- `top` → `c.back()`
- `empty` → `c.empty()`
- `size` → `c.size()`

**底层容器的要求：**
- 支持 `push_back()`、`pop_back()`、`back()`、`size()`、`empty()`。
- 随机访问和迭代器不是必需的。
- 常见可选底层容器：`std::deque`（默认）、`std::vector`、`std::list`。

**性能特点：**
- 无额外开销（仅底层容器）。
- 所有操作时间复杂度与底层容器相同。

---

## 七、时间复杂度

| 操作 | 时间复杂度（默认 deque） |
|------|--------------------------|
| 构造（空） | O(1) |
| `empty()`、`size()` | O(1) |
| `top()` | O(1) |
| `push()` / `emplace()` | O(1) |
| `pop()` | O(1) |
| `swap()` | O(1) |

---

## 八、线程安全

**`std::stack` 不是线程安全的**。规则：
- **同时读取**：多个线程同时调用 `empty()`、`size()`、`top()` 是安全的（只读）。
- **读取 + 写入**：一个线程执行 `push()`、`pop()` 时，其他线程不能进行任何操作（包括 `top()`），否则数据竞争。
- **同时写入**：多个线程同时修改栈会导致未定义行为。

**保证线程安全的常用方法：**
- 使用互斥锁（`std::mutex`）保护所有操作。
- 使用线程局部存储的栈。

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

`std::stack` **不提供迭代器**，因此没有直接的迭代器失效问题。但需要注意：
- 对栈的修改（`push`、`pop`）可能使由 `container()` 返回的底层容器的迭代器失效（依据底层容器的规则）。
- 对栈的 `swap` 不会使任何外部引用失效，但交换后两个栈的内容互换。

---

## 十一、相关结构体与函数

| 组件 | 说明 |
|------|------|
| `std::deque<T>` | 默认底层容器 |
| `std::vector<T>` | 可作为底层容器（但需要注意 `vector` 没有 `pop_front`，这不影响栈，因为栈只需 `pop_back`） |
| `std::list<T>` | 可作为底层容器 |
| `std::queue` | 另一容器适配器（FIFO） |

---
