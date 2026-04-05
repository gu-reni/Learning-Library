## `<vector>` 头文件详解

`<vector>` 是 C++ 标准模板库（STL）中定义的头文件，提供了 `std::vector` 容器。`std::vector` 是一个动态数组容器，能够根据需要自动调整大小，元素在内存中连续存储，支持高效的随机访问和尾部插入/删除。

---

## 一、模板参数

```cpp
template< class T, class Allocator = std::allocator<T> >
class vector;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型。C++11 前要求 `T` 满足 `CopyAssignable` 和 `CopyConstructible`；C++11 起要求取决于实际执行的操作，通常要求 `T` 是完整类型并满足 `Erasable` 要求 |
| `Allocator` | 分配器类型，默认为 `std::allocator<T>`。用于管理 vector 的内存分配和释放 |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class T >
    using vector = std::vector<T, std::pmr::polymorphic_allocator<T>>;
}
```

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `value_type` | `T` |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*`（或 `Allocator::pointer`） |
| `const_pointer` | `const T*` |
| `iterator` | 随机访问迭代器 |
| `const_iterator` | 常量随机访问迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `vector()` | 默认构造一个空 vector |
| `explicit vector(const Allocator& alloc)` | 使用指定分配器构造空 vector |
| `explicit vector(size_type count)` | 构造包含 `count` 个默认插入元素的 vector |
| `vector(size_type count, const T& value)` | 构造包含 `count` 个 `value` 副本的 vector |
| `template<class InputIt> vector(InputIt first, InputIt last)` | 用迭代器范围 `[first, last)` 构造 vector |
| `vector(const vector& other)` | 拷贝构造函数 |
| `vector(vector&& other)` | 移动构造函数（C++11） |
| `vector(std::initializer_list<T> init)` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~vector()` | 销毁 vector 中的所有元素并释放内存 |

**示例：**
```cpp
std::vector<int> v1;                         // 空 vector
std::vector<int> v2(5);                      // 5 个默认值（0）
std::vector<int> v3(5, 42);                  // 5 个 42
std::vector<int> v4(v3);                     // 拷贝构造
std::vector<int> v5 = {1, 2, 3, 4, 5};       // 初始化列表
std::vector<int> v6(std::move(v5));          // 移动构造
```

**实现原理：**
- 默认构造时，`size` 和 `capacity` 均为 0，不分配内存。
- 拷贝构造时，分配足够内存，将原 vector 的所有元素复制到新内存中。
- 移动构造（C++11）时，直接接管原 vector 的内存指针，将原 vector 置为空，避免内存复制。

**线程安全提示：**
`vector` 本身不是线程安全的。构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const vector& other)` | 拷贝赋值 |
| `operator=(vector&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<T> init)` | 初始化列表赋值（C++11） |
| `assign(size_type count, const T& value)` | 用 `count` 个 `value` 副本替换内容 |
| `template<class InputIt> assign(InputIt first, InputIt last)` | 用迭代器范围替换内容 |
| `assign(std::initializer_list<T> init)` | 用初始化列表替换内容（C++11） |

**示例：**
```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2;
v2 = v1;                           // 拷贝赋值
v2.assign(5, 42);                  // 变成 5 个 42
v2.assign({10, 20, 30});           // 变成 10, 20, 30
```

**实现原理：**
- 拷贝赋值时，如果目标 vector 的 `capacity` 足够大，会复用现有内存；否则会先释放再重新分配。
- 移动赋值时，直接接管源 vector 的内存，源 vector 变为空。

**线程安全提示：**
多线程中并发对同一个 vector 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `at(size_type pos)` | 返回指定位置的元素（带边界检查），越界抛出 `std::out_of_range` |
| `operator[](size_type pos)` | 返回指定位置的元素（不带边界检查），越界行为未定义 |
| `front()` | 返回第一个元素的引用 |
| `back()` | 返回最后一个元素的引用 |
| `data()` | 返回指向底层数组的指针（C++11） |

**示例：**
```cpp
std::vector<int> v = {10, 20, 30, 40, 50};
std::cout << v[0];          // 输出 10
std::cout << v.at(1);       // 输出 20
std::cout << v.front();     // 输出 10
std::cout << v.back();      // 输出 50
int* p = v.data();          // 指向底层数组的指针
```

**实现原理：**
- `operator[]` 和 `at()` 都通过 `begin() + pos` 计算指针，然后解引用返回。
- `at()` 会额外调用 `size()` 检查边界，越界时抛出异常。
- `data()` 返回指向底层数组的指针，可以直接传递给期望 C 风格数组的 C 函数。

**时间复杂度：**
所有元素访问操作都是 O(1)。

**线程安全提示：**
多个线程同时读取 vector 的不同元素是安全的，但一个线程写入而另一个线程读取同一元素会导致数据竞争。对同一个 vector 进行并发读写需要加锁。

---

### 4. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器 |
| `end()` / `cend()` | 返回指向尾后位置的迭代器 |
| `rbegin()` / `crbegin()` | 返回指向最后一个元素的反向迭代器 |
| `rend()` / `crend()` | 返回指向第一个元素前一个位置的反向迭代器 |

**示例：**
```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto it = v.begin(); it != v.end(); ++it) {
    std::cout << *it << " ";
}
for (auto rit = v.rbegin(); rit != v.rend(); ++rit) {
    std::cout << *rit << " ";
}
```

**实现原理：**
- `vector` 的迭代器通常实现为原生指针（或封装的指针），因此非常轻量。
- 元素连续存储保证了迭代器的随机访问性能。
- `begin()` 返回指向 `_M_start` 的指针，`end()` 返回指向 `_M_finish` 的指针。

**线程安全提示：**
在遍历过程中，如果另一个线程修改了 vector（如插入、删除），当前迭代器可能失效，导致未定义行为。多线程环境下遍历时应对 vector 加锁。

---

### 5. 容量管理

| 函数 | 说明 |
|------|------|
| `empty()` | 检查 vector 是否为空 |
| `size()` | 返回当前元素个数 |
| `max_size()` | 返回理论最大元素个数 |
| `capacity()` | 返回当前已分配内存可容纳的元素个数 |
| `reserve(size_type new_cap)` | 预分配内存，预留至少 `new_cap` 个元素的存储空间 |
| `shrink_to_fit()` | 请求移除未使用的内存（C++11） |

**示例：**
```cpp
std::vector<int> v;
v.reserve(100);                 // 预分配 100 个元素的空间
std::cout << v.capacity();      // 输出 >= 100
std::cout << v.size();          // 输出 0
v.shrink_to_fit();              // 释放未使用的内存
```

**实现原理：**
- `size` 和 `capacity` 通常由两个指针（`_M_start`、`_M_finish` 和 `_M_end_of_storage`）维护。
- `reserve()` 会在 `new_cap > capacity()` 时重新分配更大的内存，将旧元素移动/复制到新内存中。
- `shrink_to_fit()` 会尝试重新分配内存以减小 `capacity` 到 `size`，但标准不要求一定会执行。

**时间复杂度：**
- `empty()`、`size()`、`capacity()`：O(1)。
- `reserve(n)`：O(n)（当 `n > capacity()` 时）。

**线程安全提示：**
`reserve()` 会改变 vector 的内存布局，在并发环境中使用需要同步。

---

### 6. 修改器

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert(iterator pos, const T& value)` | 在 `pos` 位置插入元素 |
| `insert(iterator pos, T&& value)` | 插入右值（C++11） |
| `insert(iterator pos, size_type count, const T& value)` | 插入多个相同元素 |
| `template<class InputIt> insert(iterator pos, InputIt first, InputIt last)` | 插入迭代器范围 |
| `insert(iterator pos, std::initializer_list<T> init)` | 插入初始化列表（C++11） |
| `emplace(iterator pos, Args&&... args)` | 在 `pos` 位置原位构造元素（C++11） |
| `emplace_back(Args&&... args)` | 在末尾原位构造元素（C++11） |
| `push_back(const T& value)` | 在末尾添加元素 |
| `push_back(T&& value)` | 在末尾添加右值元素（C++11） |
| `pop_back()` | 移除最后一个元素 |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除迭代器范围内的元素 |
| `resize(size_type count)` | 调整容器大小 |
| `resize(size_type count, const value_type& value)` | 调整容器大小，新元素填充 `value` |
| `swap(vector& other)` | 交换两个 vector 的内容 |

**示例：**
```cpp
std::vector<int> v = {1, 2, 3};
v.push_back(4);                  // 1,2,3,4
v.pop_back();                    // 1,2,3
v.insert(v.begin(), 0);          // 0,1,2,3
v.erase(v.begin() + 1);          // 0,2,3
v.emplace_back(5);               // 0,2,3,5
v.emplace(v.begin(), 6);         // 6,0,2,3,5
v.resize(3);                     // 6,0,2
v.swap(v2);                      // 与 v2 交换内容
```

**实现原理：**
- `push_back` 会在 `size == capacity` 时触发扩容，分配新内存（通常为旧容量的 1.5 倍或 2 倍），将旧元素移动到新内存，然后添加新元素。
- `emplace_back` 相比 `push_back` 可以减少一次拷贝/移动，直接在内存中构造对象。
- `insert` 和 `erase` 涉及元素的移动：插入时，从插入点开始的元素会后移；删除时，后面的元素会前移。
- `swap` 只交换三个指针（`_M_start`、`_M_finish`、`_M_end_of_storage`）和分配器，不涉及元素复制。

**时间复杂度：**
- `push_back` / `emplace_back` / `pop_back`：均摊 O(1)。
- `insert` / `erase`：O(n)（n 为从插入/删除点到末尾的元素个数）。
- `swap`：O(1)。

**线程安全提示：**
多个线程对同一个 vector 进行修改（如 `push_back`、`insert`、`erase`）会导致数据竞争。如果多个线程需要修改，必须使用互斥锁（如 `std::mutex`）或其他同步机制保护。

---

### 7. 比较操作符

| 操作符 | 说明 |
|--------|------|
| `operator==` / `operator!=` | 比较两个 vector 是否相等 |
| `operator<` / `operator<=` / `operator>` / `operator>=` | 按字典序比较 |

这些操作符通常定义在 `<vector>` 头文件中，作为非成员函数。

---

### 8. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器的副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 vector |
| `swap(std::vector& lhs, std::vector& rhs)` | 交换两个 vector 的内容 |
| `erase(std::vector& c, const U& value)` | 移除所有等于 `value` 的元素（C++20） |
| `erase_if(std::vector& c, Predicate pred)` | 移除所有满足谓词 `pred` 的元素（C++20） |

---

## 五、宏与常量

`<vector>` 头文件中**没有定义任何宏**。所有容量和限制相关信息通过成员函数（如 `max_size()`）获取。

---

## 六、实现原理

`std::vector` 的典型实现使用三个指针来管理内存：
- `_M_start`：指向底层数组的起始位置。
- `_M_finish`：指向最后一个有效元素之后的位置（`size = _M_finish - _M_start`）。
- `_M_end_of_storage`：指向已分配内存的末尾（`capacity = _M_end_of_storage - _M_start`）。

**扩容机制：**
当 `size == capacity` 且需要插入新元素时，vector 会重新分配一块更大的连续内存，将旧元素移动/复制到新内存中，然后释放旧内存。新的容量通常是旧容量的 1.5 倍或 2 倍，具体取决于标准库实现。这种指数级扩容策略保证了 `push_back` 的均摊 O(1) 时间复杂度。

**内存布局：**
元素在内存中连续存储，因此：
- 可以通过 `&v[0]` 获得指向底层数组的指针，并传递给期望 C 风格数组的 C 函数。
- 迭代器本质上是指针，访问效率高。
- 随机访问 O(1) 的时间复杂度得益于连续存储。

**`std::vector<bool>` 特化：**
`std::vector<bool>` 是 `std::vector` 对 `bool` 类型的特化版本，使用位压缩存储，每个 bool 只占用 1 位。因此它不满足 `ContiguousContainer` 的要求，也不提供 `data()` 成员函数。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 随机访问（`operator[]`、`at()`） | O(1) |
| 在末尾插入/删除（`push_back`、`emplace_back`、`pop_back`） | 均摊 O(1) |
| 在任意位置插入/删除（`insert`、`emplace`、`erase`） | O(n) |
| 查找（按值） | O(n) |
| 容量操作（`size`、`capacity`、`empty`） | O(1) |
| 预分配内存（`reserve`） | O(n)（当 `n > capacity()` 时） |
| 交换（`swap`） | O(1) |

---

## 八、线程安全

**`std::vector` 本身不是线程安全的**。标准库容器不提供内置的线程安全机制。

**并发读写规则：**
- **同时读取**：多个线程同时读取同一个 vector 的不同元素是安全的。
- **读取 + 写入**：一个线程写入时，其他线程不能进行任何操作（包括读取），否则会导致数据竞争。
- **同时写入**：多个线程同时写入（包括修改不同位置的元素）会导致未定义行为。
- **修改导致扩容**：扩容会使所有迭代器、指针和引用失效，并发访问更加危险。

**保证线程安全的常用方法：**
- 使用 `std::mutex` 或 `std::shared_mutex` 保护所有对 vector 的访问。
- 使用线程局部存储，每个线程使用独立的 vector 实例。
- 使用第三方并发容器（如 `tbb::concurrent_vector`）。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符
- `emplace()` 和 `emplace_back()` 原位构造
- `shrink_to_fit()` 请求释放未使用内存
- `cbegin()` / `cend()` / `crbegin()` / `crend()` 常量迭代器
- 支持初始化列表构造和赋值
- 支持右值引用版本的 `push_back()` 和 `insert()`
- 满足 `AllocatorAwareContainer` 要求

### C++14
- 无明显针对 vector 的新特性

### C++17
- `std::pmr::vector` 多态分配器别名模板
- 满足 `ContiguousContainer` 要求
- `insert` 等成员函数支持节点提取

### C++20
- 所有成员函数都变为 `constexpr`（可以在常量表达式中创建和使用）
- `erase()` 和 `erase_if()` 非成员函数

### C++23
- `resize_and_overwrite` 等新成员函数

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `reserve(n)`（`n > capacity()`） | 所有迭代器、指针和引用失效（内存重新分配） |
| `reserve(n)`（`n <= capacity()`） | 无失效 |
| `shrink_to_fit()` | 所有迭代器、指针和引用失效 |
| `push_back()` / `emplace_back()` | 如果触发扩容，所有迭代器失效；否则仅 `end()` 失效 |
| `pop_back()` | 指向被删除元素的迭代器和引用失效，`end()` 失效 |
| `insert(pos, ...)` | 如果触发扩容，所有迭代器失效；否则，从 `pos` 开始的所有迭代器失效 |
| `emplace(pos, ...)` | 同 `insert` |
| `erase(pos)` | 从 `pos` 开始的所有迭代器失效 |
| `erase(first, last)` | 从 `first` 开始的所有迭代器失效 |
| `clear()` | 所有迭代器失效 |
| `swap()` | 所有迭代器失效（但指向不同容器的元素） |
| `resize()` | 如果新 `size` 大于旧 `size`，且触发扩容，所有迭代器失效；否则，可能部分失效 |

**特别注意：**
- 指向 `vector` 元素的指针和引用的失效规则与迭代器相同。
- 使用 `reserve()` 预分配足够的内存可以避免插入操作引起的迭代器失效。

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::allocator<T>` | 默认分配器，位于 `<memory>` 头文件 |
| `std::pmr::polymorphic_allocator<T>` | 多态分配器（C++17），位于 `<memory_resource>` 头文件 |
| `std::vector<bool>` | `bool` 类型的特化，使用位压缩存储 |
| `std::reverse_iterator` | 反向迭代器适配器 |

---