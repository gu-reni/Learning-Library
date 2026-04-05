## `<queue>` 头文件（`std::priority_queue`）详解

`<queue>` 是 C++ 标准模板库（STL）中定义的头文件，提供了容器适配器 `std::priority_queue`。`std::priority_queue` 是一个优先队列，它保证最大（或最小）元素总是在队列的顶部（`top()`），内部基于堆（heap）实现。插入和删除操作的时间复杂度为 O(log n)，获取顶部元素为 O(1)。

---

## 一、模板参数

```cpp
template< class T,
          class Container = std::vector<T>,
          class Compare = std::less<typename Container::value_type> >
class priority_queue;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素的类型 |
| `Container` | 底层容器类型，必须满足序列容器（`vector`、`deque` 等）的要求，且支持 `push_back()`、`pop_back()`、`front()`、`size()` 等操作，默认为 `std::vector<T>` |
| `Compare` | 比较函数对象类型，用于定义元素的优先级顺序。默认为 `std::less<T>`，此时最大元素具有最高优先级（即 `top()` 返回最大元素）。如果使用 `std::greater<T>`，则最小元素具有最高优先级 |

**注意：** 比较函数必须实现**严格弱序**（Strict Weak Ordering）。`priority_queue` 内部使用 `Compare` 来维护一个最大堆（默认），即 `Compare` 返回 `true` 表示第一个参数应该排在第二个参数**之后**（即优先级更低）。

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `value_type` | `T` |
| `size_type` | `Container::size_type`（通常为 `size_t`） |
| `reference` | `Container::reference`（通常为 `T&`） |
| `const_reference` | `Container::const_reference`（通常为 `const T&`） |
| `container_type` | `Container`（底层容器类型） |
| `compare_type` | `Compare`（比较器类型） |

**注意：** `priority_queue` 不提供迭代器，因此没有 `iterator`、`const_iterator` 等成员类型。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `priority_queue()` | 默认构造一个空优先队列，使用默认的底层容器和比较器 |
| `explicit priority_queue(const Compare& comp)` | 使用给定的比较器构造空优先队列 |
| `explicit priority_queue(const Container& cont)` | 使用给定的底层容器副本构造优先队列（会执行堆化操作） |
| `priority_queue(const Compare& comp, const Container& cont)` | 使用给定的比较器和底层容器副本构造 |
| `priority_queue(const Compare& comp, Container&& cont)` | 使用给定的比较器和移动的底层容器构造（C++11） |
| `template<class InputIt> priority_queue(InputIt first, InputIt last, const Compare& comp = Compare(), const Container& cont = Container())` | 用迭代器范围 `[first, last)` 构造，可选比较器和底层容器 |
| `priority_queue(const priority_queue& other)` | 拷贝构造 |
| `priority_queue(priority_queue&& other)` | 移动构造（C++11） |
| `priority_queue(std::initializer_list<T> init, const Compare& comp = Compare(), const Container& cont = Container())` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~priority_queue()` | 销毁优先队列及其元素（调用底层容器的析构函数） |

**示例：**
```cpp
std::priority_queue<int> pq1;                          // 空优先队列，最大堆
std::priority_queue<int, std::vector<int>, std::greater<int>> pq2; // 最小堆
std::vector<int> v = {1,5,3,4,2};
std::priority_queue<int> pq3(v.begin(), v.end());      // 从范围构造，内部堆化
std::priority_queue<int> pq4(pq3);                     // 拷贝构造
```

**实现原理：**
- 默认构造时，底层容器被默认构造，不分配堆。
- 从迭代器范围构造时，先将元素插入底层容器，然后调用 `std::make_heap` 进行堆化。
- 拷贝/移动构造分别复制/移动底层容器和比较器。

**线程安全提示：**
构造和析构应在单线程环境进行，或有锁保护。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const priority_queue& other)` | 拷贝赋值 |
| `operator=(priority_queue&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<T> init)` | 初始化列表赋值（C++11） |

**示例：**
```cpp
std::priority_queue<int> pq1;
pq1 = {1,2,3};                     // 初始化列表赋值
std::priority_queue<int> pq2;
pq2 = pq1;                         // 拷贝赋值
```

**实现原理：**
- 拷贝赋值时，底层容器和比较器都被复制。
- 移动赋值时，直接接管底层容器和比较器的资源。

**线程安全提示：**
并发赋值需要同步。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `const_reference top() const` | 返回最高优先级元素的常量引用（即堆顶元素）。优先队列不能为空，否则行为未定义 |

**示例：**
```cpp
std::priority_queue<int> pq;
pq.push(10);
pq.push(20);
pq.push(5);
std::cout << pq.top();   // 输出 20（默认最大堆）
```

**实现原理：**
- `top()` 返回底层容器的第一个元素（`c.front()`），因为堆算法确保最大/最小元素在索引 0 位置。

**时间复杂度：** O(1)

**线程安全提示：**
只读操作，但若其他线程同时修改，则不安全。并发读取 `top()` 是安全的，只要没有写入。

---

### 4. 容量

| 函数 | 说明 |
|------|------|
| `bool empty() const` | 检查优先队列是否为空 |
| `size_type size() const` | 返回元素个数 |

**示例：**
```cpp
std::priority_queue<int> pq;
pq.push(1);
pq.pop();
bool e = pq.empty();   // true
size_t s = pq.size();  // 0
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
| `void push(const T& value)` | 插入元素（复制） |
| `void push(T&& value)` | 插入元素（移动）（C++11） |
| `template<class... Args> void emplace(Args&&... args)` | 原位构造元素（C++11） |
| `void pop()` | 移除最高优先级元素（堆顶）。优先队列不能为空，否则行为未定义 |
| `void swap(priority_queue& other)` | 交换内容（C++11） |

**示例：**
```cpp
std::priority_queue<int> pq;
pq.push(5);
pq.push(10);
pq.emplace(3);          // 直接构造
pq.pop();               // 移除 10
```

**实现原理：**
- `push(value)`：先将元素添加到底层容器末尾，然后调用 `std::push_heap` 重新调整堆。
- `pop()`：调用 `std::pop_heap` 将堆顶元素移到末尾，再调用底层容器的 `pop_back()` 移除该元素。
- `emplace`：在底层容器末尾原位构造元素，然后调用 `std::push_heap`。

**时间复杂度：**
- `push` / `emplace`：O(log n)
- `pop`：O(log n)
- `swap`：O(1)（如果底层容器的 `swap` 是 O(1)，如 `vector` 和 `deque`）

**线程安全提示：**
所有修改操作都需要同步。

---

### 6. 底层容器访问（C++11 起）

| 函数 | 说明 |
|------|------|
| `const Container& container() const` | 返回底层容器的常量引用（C++11） |

**示例：**
```cpp
std::priority_queue<int> pq;
pq.push(3);
pq.push(1);
pq.push(4);
const auto& c = pq.container();   // 获取底层容器（如 vector）
// 注意：c 不保证堆序，堆化仅保证 top 正确，内部顺序未指定
```

**实现原理：**
- 直接返回底层容器对象。

**线程安全提示：**
只读，但若其他线程修改优先队列，则返回值可能过时。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `swap(priority_queue& lhs, priority_queue& rhs)` | 交换两个优先队列的内容（C++11） |

**示例：**
```cpp
std::priority_queue<int> pq1, pq2;
std::swap(pq1, pq2);
```

---

## 五、宏与常量

`<queue>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::priority_queue` 是一个容器适配器，它封装了底层容器并提供堆接口。典型实现包含两个成员：

```cpp
Container c;   // 底层容器（默认 vector）
Compare comp;  // 比较器
```

**堆算法：**
- 使用 `std::make_heap`、`std::push_heap`、`std::pop_heap` 等算法来维护堆性质。
- 堆是一棵完全二叉树，通常用数组表示：索引 `i` 的左子节点为 `2*i+1`，右子节点为 `2*i+2`。
- 默认比较器 `std::less` 创建**最大堆**（`top()` 返回最大元素）。
- 使用 `std::greater` 创建**最小堆**。

**操作流程：**
- `push(x)`：`c.push_back(x)` → `std::push_heap(c.begin(), c.end(), comp)`
- `pop()`：`std::pop_heap(c.begin(), c.end(), comp)` → `c.pop_back()`
- `top()`：`return c.front()`

**性能特点：**
- 空间开销小（仅底层容器 + 比较器）。
- 插入和删除 O(log n)，堆化（构造时）O(n)。
- 元素在底层容器中并不是完全有序的，只有堆顶满足优先级最高。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 构造（空） | O(1) |
| 构造（从范围） | O(n)（make_heap） |
| `empty()`、`size()` | O(1) |
| `top()` | O(1) |
| `push()` / `emplace()` | O(log n) |
| `pop()` | O(log n) |
| `swap()` | O(1)（取决于底层容器） |

---

## 八、线程安全

**`std::priority_queue` 不是线程安全的**。规则：
- **同时读取**：多个线程同时调用 `empty()`、`size()`、`top()` 是安全的（只读）。
- **读取 + 写入**：一个线程执行 `push()`、`pop()` 时，其他线程不能进行任何操作（包括 `top()`），否则数据竞争。
- **同时写入**：多个线程同时修改优先队列会导致未定义行为。

**保证线程安全的常用方法：**
- 使用互斥锁（`std::mutex`）保护所有操作。
- 使用线程局部存储的优先队列。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace` 原位构造。
- `container()` 访问底层容器。
- 初始化列表构造函数和赋值。
- 右值引用版本的 `push`。

### C++14
- 无显著新特性。

### C++17
- 添加 `std::priority_queue` 的推导指引（deduction guides），允许从底层容器或迭代器范围自动推导模板参数。
- `swap` 被标记为 `noexcept` 若底层容器的 `swap` 也为 `noexcept`。

### C++20
- 比较操作符支持（通过底层容器的比较）。

---

## 十、迭代器失效规则

`std::priority_queue` **不提供迭代器**，因此没有直接的迭代器失效问题。但需要注意：
- 对优先队列的修改（`push`、`pop`）可能使由 `container()` 返回的底层容器的迭代器失效（依据底层容器的规则）。
- 对优先队列的 `swap` 不会使任何外部引用失效，但交换后两个队列的内容互换。

---

## 十一、相关结构体与函数

| 组件 | 说明 |
|------|------|
| `std::less<T>` | 默认比较器，产生最大堆 |
| `std::greater<T>` | 产生最小堆 |
| `std::make_heap` | 将范围构建为堆（在 `<algorithm>` 中） |
| `std::push_heap` | 向堆中添加元素 |
| `std::pop_heap` | 从堆中移除最大/最小元素 |

---
