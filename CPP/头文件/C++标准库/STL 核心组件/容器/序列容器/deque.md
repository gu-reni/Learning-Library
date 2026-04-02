## `<deque>` 头文件详解

`<deque>` 是 C++ 标准模板库（STL）中定义的头文件，提供了 `std::deque` 容器（双端队列）。`std::deque`（double-ended queue）是一个支持在两端进行高效插入和删除的序列容器，允许随机访问。与 `vector` 不同，`deque` 的元素不是连续存储在单个内存块中，而是分段存储，因此不需要在头部插入时移动大量元素。

---

## 一、模板参数

```cpp
template< class T, class Allocator = std::allocator<T> >
class deque;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型。必须满足 `Erasable` 要求，具体取决于实际执行的操作 |
| `Allocator` | 分配器类型，默认为 `std::allocator<T>`。用于管理 `deque` 的内存分配和释放 |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class T >
    using deque = std::deque<T, std::pmr::polymorphic_allocator<T>>;
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
| `iterator` | 随机访问迭代器（实现定义，通常为支持随机访问的迭代器） |
| `const_iterator` | 常量随机访问迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `deque()` | 默认构造一个空 `deque` |
| `explicit deque(const Allocator& alloc)` | 使用指定分配器构造空 `deque` |
| `explicit deque(size_type count)` | 构造包含 `count` 个默认插入元素的 `deque` |
| `deque(size_type count, const T& value)` | 构造包含 `count` 个 `value` 副本的 `deque` |
| `template<class InputIt> deque(InputIt first, InputIt last)` | 用迭代器范围 `[first, last)` 构造 `deque` |
| `deque(const deque& other)` | 拷贝构造函数 |
| `deque(deque&& other)` | 移动构造函数（C++11） |
| `deque(std::initializer_list<T> init)` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~deque()` | 销毁 `deque` 中的所有元素并释放内存 |

**示例：**
```cpp
std::deque<int> d1;                     // 空 deque
std::deque<int> d2(5);                  // 5 个默认值（0）
std::deque<int> d3(5, 42);              // 5 个 42
std::deque<int> d4(d3);                 // 拷贝构造
std::deque<int> d5 = {1,2,3,4,5};       // 初始化列表
std::deque<int> d6(std::move(d5));      // 移动构造
```

**实现原理：**
- `deque` 通常由一块“中央控制数组”（map）和多个固定大小的块（block）组成。
- 默认构造时，可能不分配任何块或只分配一个空块。
- 拷贝构造时，复制所有元素到新的块中。
- 移动构造时，直接接管原 `deque` 的控制数组和块指针，原 `deque` 变为空。

**线程安全提示：**
构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const deque& other)` | 拷贝赋值 |
| `operator=(deque&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<T> init)` | 初始化列表赋值（C++11） |
| `assign(size_type count, const T& value)` | 用 `count` 个 `value` 副本替换内容 |
| `template<class InputIt> assign(InputIt first, InputIt last)` | 用迭代器范围替换内容 |
| `assign(std::initializer_list<T> init)` | 用初始化列表替换内容（C++11） |

**示例：**
```cpp
std::deque<int> d1 = {1,2,3};
std::deque<int> d2;
d2 = d1;                           // 拷贝赋值
d2.assign(5, 42);                  // 变成 5 个 42
d2.assign({10, 20, 30});           // 变成 10, 20, 30
```

**实现原理：**
- 拷贝赋值时，如果目标 `deque` 的容量足够，会复用现有块；否则会重新分配。
- 移动赋值时，直接接管源 `deque` 的内存，源 `deque` 变为空。

**线程安全提示：**
多线程中并发对同一个 `deque` 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `at(size_type pos)` | 返回指定位置的元素（带边界检查），越界抛出 `std::out_of_range` |
| `operator[](size_type pos)` | 返回指定位置的元素（不带边界检查），越界行为未定义 |
| `front()` | 返回第一个元素的引用 |
| `back()` | 返回最后一个元素的引用 |

**示例：**
```cpp
std::deque<int> d = {10, 20, 30, 40, 50};
std::cout << d[0];          // 输出 10
std::cout << d.at(1);       // 输出 20
std::cout << d.front();     // 输出 10
std::cout << d.back();      // 输出 50
```

**实现原理：**
- `deque` 的随机访问通过控制数组和块内偏移实现。
- 典型实现中，`operator[]` 先通过控制数组找到对应的块，然后在块内计算偏移。
- 时间复杂度 O(1)。

**线程安全提示：**
多个线程同时读取不同元素是安全的；一个线程写入而另一个线程读取同一元素会导致数据竞争。

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
std::deque<int> d = {1, 2, 3, 4, 5};
for (auto it = d.begin(); it != d.end(); ++it) {
    std::cout << *it << " ";
}
for (auto rit = d.rbegin(); rit != d.rend(); ++rit) {
    std::cout << *rit << " ";
}
```

**实现原理：**
- `deque` 的迭代器是一个类，内部通常包含指向当前块的指针、块内偏移和指向控制数组的指针。
- 迭代器支持随机访问（`+`、`-`、`[]` 等），通过计算跨块偏移实现。

**线程安全提示：**
遍历过程中，如果另一个线程修改 `deque`（如插入、删除），迭代器可能失效。多线程遍历时应加锁。

---

### 5. 容量管理

| 函数 | 说明 |
|------|------|
| `empty()` | 检查 `deque` 是否为空 |
| `size()` | 返回当前元素个数 |
| `max_size()` | 返回理论最大元素个数 |
| `shrink_to_fit()` | 请求移除未使用的内存（C++11，非强制性） |

**示例：**
```cpp
std::deque<int> d;
std::cout << d.empty();    // true
d.push_back(42);
std::cout << d.size();     // 1
d.shrink_to_fit();         // 可能释放未使用的块
```

**实现原理：**
- `size` 通常通过记录总元素个数维护。
- `deque` 没有 `capacity()` 成员，因为内存是分段分配的，没有单一容量概念。
- `shrink_to_fit()` 尝试重新分配控制数组和块，使其刚好容纳当前元素。

**时间复杂度：**
- `empty()`、`size()`：O(1)。
- `shrink_to_fit()`：O(size())，但标准不保证一定执行。

**线程安全提示：**
`shrink_to_fit()` 会重新分配内存，并发使用需要同步。

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
| `emplace_front(Args&&... args)` | 在头部原位构造元素（C++11） |
| `emplace_back(Args&&... args)` | 在尾部原位构造元素（C++11） |
| `push_front(const T& value)` | 在头部添加元素 |
| `push_front(T&& value)` | 在头部添加右值元素（C++11） |
| `push_back(const T& value)` | 在尾部添加元素 |
| `push_back(T&& value)` | 在尾部添加右值元素（C++11） |
| `pop_front()` | 移除第一个元素 |
| `pop_back()` | 移除最后一个元素 |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除迭代器范围内的元素 |
| `resize(size_type count)` | 调整容器大小 |
| `resize(size_type count, const value_type& value)` | 调整容器大小，新元素填充 `value` |
| `swap(deque& other)` | 交换两个 `deque` 的内容 |

**示例：**
```cpp
std::deque<int> d = {2,3,4};
d.push_front(1);                // 1,2,3,4
d.push_back(5);                 // 1,2,3,4,5
d.pop_front();                  // 2,3,4,5
d.pop_back();                   // 2,3,4
d.insert(d.begin() + 1, 99);    // 2,99,3,4
d.erase(d.begin());             // 99,3,4
d.emplace_front(0);             // 0,99,3,4
d.emplace_back(5);              // 0,99,3,4,5
```

**实现原理：**
- `push_front` / `pop_front`：通过分配新块或在现有块头部预留空间实现，无需移动其他元素（O(1)）。
- `push_back` / `pop_back`：类似，在尾部块操作。
- `insert` / `erase` 在中间位置：需要移动元素，可能比 `vector` 更高效（因为只需要移动较近一端）。
- 当需要新块时，`deque` 通过控制数组管理块，可能重新分配控制数组。

**时间复杂度：**
- `push_front` / `pop_front` / `push_back` / `pop_back`：O(1)。
- `insert` / `erase`：O(n)（n 为到最近端点的距离）。
- `swap`：O(1)（只交换控制数组指针和大小）。

**线程安全提示：**
多个线程对同一个 `deque` 进行修改（如 `push_front`、`insert`、`erase`）会导致数据竞争，必须使用互斥锁保护。

---

### 7. 比较操作符

| 操作符 | 说明 |
|--------|------|
| `operator==` / `operator!=` | 比较两个 `deque` 是否相等 |
| `operator<` / `operator<=` / `operator>` / `operator>=` | 按字典序比较 |

这些操作符通常定义在 `<deque>` 头文件中，作为非成员函数。

---

### 8. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器的副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 `deque` |
| `swap(std::deque& lhs, std::deque& rhs)` | 交换两个 `deque` 的内容 |
| `erase(std::deque& c, const U& value)` | 移除所有等于 `value` 的元素（C++20） |
| `erase_if(std::deque& c, Predicate pred)` | 移除所有满足谓词 `pred` 的元素（C++20） |

---

## 五、宏与常量

`<deque>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::deque` 的典型实现使用**中央控制数组（map）**和**固定大小的块（block）**：

```
map (指针数组)     块0       块1       块2       块3
   [0]  ----> [元素][元素][元素]...
   [1]  ----> [元素][元素][元素]...
   [2]  ----> [元素][元素][元素]...
   ...
```

- **块大小**：通常为 512 字节或 4096 字节，与元素大小相关，使得每个块可容纳固定数量的元素。
- **控制数组**：存储指向各个块的指针。当在两端插入导致需要新块时，如果控制数组两端有空位，则直接分配新块并在控制数组中添加指针；否则重新分配控制数组（扩容）。
- **索引计算**：通过 `(index / block_size)` 定位块，通过 `(index % block_size)` 定位块内偏移。
- **两端高效插入**：因为可以在控制数组的两端添加块，所以 `push_front` 和 `push_back` 都是 O(1)，不需要移动已有元素。
- **中间插入**：需要移动从插入点到一端的元素，通常选择移动较近的一端，以减少移动次数。

**与 `vector` 的对比：**
- `deque` 支持 O(1) 的头部插入/删除，`vector` 不支持。
- `deque` 的内存不是连续的，因此不能保证与 C 风格数组的兼容性（无 `data()` 成员函数）。
- `deque` 的随机访问稍慢（需要两次间接访问），但仍然是 O(1)。
- `deque` 的迭代器比 `vector` 的迭代器更复杂（不是原生指针）。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 随机访问（`operator[]`、`at()`） | O(1) |
| 在头部插入/删除（`push_front`、`pop_front`） | O(1) |
| 在尾部插入/删除（`push_back`、`pop_back`） | O(1) |
| 在中间位置插入/删除（`insert`、`erase`） | O(n)（到最近端点的距离） |
| 查找（按值） | O(n) |
| 容量操作（`size`、`empty`） | O(1) |
| 交换（`swap`） | O(1) |

---

## 八、线程安全

**`std::deque` 本身不是线程安全的**。规则与 `vector` 类似：
- **同时读取**：多个线程同时读取不同元素是安全的。
- **读取 + 写入**：一个线程写入时，其他线程不能进行任何操作（包括读取），否则数据竞争。
- **同时写入**：多个线程同时写入（包括在两端插入）会导致未定义行为。
- **重新分配**：当控制数组需要扩容时，所有迭代器、指针和引用可能失效，并发访问更危险。

**保证线程安全的常用方法：**
- 使用 `std::mutex` 或 `std::shared_mutex` 保护所有对 `deque` 的访问。
- 使用线程局部存储。
- 使用第三方并发容器。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace`、`emplace_front`、`emplace_back` 原位构造。
- `shrink_to_fit()`。
- `cbegin()` / `cend()` / `crbegin()` / `crend()`。
- 初始化列表构造和赋值。
- 右值引用版本的 `push_back`、`push_front`、`insert`。

### C++14
- 无明显针对 `deque` 的新特性。

### C++17
- `std::pmr::deque` 多态分配器别名模板。
- `insert` 等成员函数支持节点提取（未合并）。

### C++20
- `erase()` 和 `erase_if()` 非成员函数。
- `constexpr` 修饰部分成员函数（但 `deque` 不能完全 `constexpr`，因为涉及动态内存分配）。

### C++23
- `deque` 的范围构造函数和范围赋值（`from_range_t`）。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `push_front` / `push_back` | 所有迭代器失效（因为可能重新分配控制数组），但元素引用不失效（除 C++11 前） |
| `pop_front` / `pop_back` | 仅指向被删除元素的迭代器失效 |
| `insert(pos, ...)` | 所有迭代器失效（C++11 前）；C++11 起，如果插入点靠近一端且没有重新分配控制数组，则可能只影响部分迭代器 |
| `erase(pos)` | 被删除元素及之后的所有迭代器失效 |
| `shrink_to_fit()` | 所有迭代器失效 |
| `swap()` | 所有迭代器失效（但指向其他容器） |
| `resize()` | 如果缩小，被删除元素迭代器失效；如果扩大，所有迭代器可能失效 |
| `clear()` | 所有迭代器失效 |

**注意：** 由于 `deque` 的实现差异，迭代器失效规则较为复杂。通常建议：任何可能改变容器大小或重新分配控制数组的操作都可能导致所有迭代器失效。安全做法是在修改后重新获取迭代器。

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::allocator<T>` | 默认分配器 |
| `std::pmr::polymorphic_allocator<T>` | 多态分配器（C++17） |
| `std::reverse_iterator` | 反向迭代器适配器 |

---