## `<span>` 头文件详解

`<span>` 是 C++20 标准库中定义的头文件，提供了 `std::span` 类模板。`std::span` 是一个轻量级的、非拥有的连续内存视图，可以引用数组、`std::vector`、`std::array` 等连续存储的对象序列，而不复制数据。它类似于指针和长度的组合，但提供了更安全的接口和更丰富的功能，是函数接口设计的现代替代方案。

---

## 一、模板参数

```cpp
template< class T, std::size_t Extent = std::dynamic_extent >
class span;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型；必须是完整的非抽象类对象类型 |
| `Extent` | 序列中的元素数量，若为动态则为 `std::dynamic_extent` |

**说明：** 
- `std::dynamic_extent` 是一个常量（通常为 `static_cast<std::size_t>(-1)`），表示 span 的大小在运行时确定。
- 如果 `Extent` 是一个具体的非负整数，则 span 具有**静态范围**（编译时大小），类型中编码了大小信息。
- 如果 `Extent` 为 `std::dynamic_extent`，则 span 具有**动态范围**（运行时大小），大小信息存储在对象中。

**多态分配器别名（C++17 起，注意：`std::pmr::span` 是 C++20 引入的）：**
```cpp
namespace pmr {
    template< class T >
    using span = std::span<T, std::dynamic_extent>;
}
```

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `element_type` | `T` |
| `value_type` | `std::remove_cv_t<T>` |
| `size_type` | `std::size_t` |
| `difference_type` | `std::ptrdiff_t` |
| `pointer` | `T*` |
| `const_pointer` | `const T*` |
| `reference` | `T&` |
| `const_reference` | `const T&` |
| `iterator` | 实现定义的随机访问迭代器（LegacyRandomAccessIterator），满足 `ConstexprIterator` 和 `contiguous_iterator` 要求，其 `value_type` 为 `value_type` |
| `const_iterator` | `std::const_iterator<iterator>`（C++23 起） |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::const_iterator<reverse_iterator>`（C++23 起） |

**注意：** `iterator` 是一个可变迭代器，如果 `T` 不是 const 限定类型。所有对容器迭代器类型的要求同样适用于 `span` 的迭代器类型。

---

## 三、成员常量

| 名称 | 值 |
|------|-----|
| `static constexpr std::size_t extent = Extent;` | `Extent` 模板参数的值（公开静态成员常量） |

---

## 四、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `span()` | 默认构造一个空 span，其 `data() == nullptr` 且 `size() == 0`。此重载仅在 `Extent == 0 \|\| Extent == std::dynamic_extent` 时参与重载决议 |
| `span(const span& other)` | 拷贝构造函数（默认） |
| `span(span&& other)` | 移动构造函数（默认） |
| `span(T* ptr, size_type count)` | 从指针和大小构造 |
| `span(T* first, T* last)` | 从指针范围 `[first, last)` 构造 |
| `span(T (&arr)[N])` | 从 C 风格数组构造（自动推导大小） |
| `span(std::array<T, N>& arr)` | 从 `std::array` 构造 |
| `span(const std::array<T, N>& arr)` | 从 const `std::array` 构造 |
| `span(Container& cont)` | 从支持 `data()` 和 `size()` 的容器（如 `std::vector`）构造 |
| `span(Container const& cont)` | 从 const 容器构造 |
| `span(std::initializer_list<T> il)` | 从初始化列表构造（需要 `Extent == std::dynamic_extent` 或 `il.size() == Extent`） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~span()` | 析构一个 span（隐式声明） |

**示例：**
```cpp
#include <span>
#include <vector>
#include <array>

int arr[] = {1, 2, 3, 4, 5};
std::vector<int> vec = {6, 7, 8, 9, 10};
std::array<int, 5> arr2 = {11, 12, 13, 14, 15};

std::span<int> s1(arr);                     // 从数组构造（动态大小）
std::span<int> s2(vec);                     // 从 vector 构造
std::span<int, 5> s3(arr2);                 // 静态大小 span
std::span<int> s4(arr, 3);                  // 只取前 3 个元素
std::span<int> s5(arr + 1, arr + 4);        // 从指针范围构造（索引 1-3）
```

**实现原理：**
- `std::span` 是一个轻量级的非拥有式容器，仅存储指向底层序列的指针和大小（当范围为动态时）。
- 在 64 位系统下，每个 `span` 实例通常仅占用 16 字节内存（8 字节指针 + 8 字节大小）。
- `std::span` 的每个特化都是可平凡复制类型（C++23 起），这意味着其复制构造函数、移动构造函数等都是平凡的。

**线程安全提示：**
构造和析构操作是平凡的，可以在多线程环境中安全地进行（只要不共享同一个 `span` 对象）。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const span& other)` | 拷贝赋值（默认） |
| `operator=(span&& other)` | 移动赋值（默认） |

**实现原理：**
- 赋值操作是平凡的，只是复制指针和大小（对于动态范围）或仅复制指针（对于静态范围）。

**线程安全提示：**
并发对同一个 `span` 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `front()` | 访问第一个元素（C++23 起） |
| `back()` | 访问最后一个元素（C++26 起） |
| `at(size_type pos)` | 带边界检查访问指定元素，越界抛出 `std::out_of_range`（C++26 起） |
| `operator[](size_type pos)` | 访问指定元素（不带边界检查） |
| `data()` | 返回指向底层连续存储的指针 |

**示例：**
```cpp
std::span<int> s = {1, 2, 3, 4, 5};
int a = s[0];          // 1
int b = s.front();     // 1 (C++23)
int c = s.back();      // 5 (C++26)
int* p = s.data();     // 指向第一个元素的指针
int d = s.at(2);       // 3 (C++26)
```

**实现原理：**
- `operator[]` 通过 `_ptr + pos * sizeof(T)` 计算地址，O(1) 复杂度。
- `at()` 在访问前检查边界，越界时抛出异常。
- `data()` 返回存储的指针。

**时间复杂度：**
所有元素访问操作都是 O(1)。

**线程安全提示：**
多个线程同时读取不同元素是安全的；一个线程写入而另一个线程读取同一元素会导致数据竞争。

---

### 4. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向起始的迭代器（C++23 起） |
| `end()` / `cend()` | 返回指向末尾的迭代器（C++23 起） |
| `rbegin()` / `crbegin()` | 返回指向起始的逆向迭代器 |
| `rend()` / `crend()` | 返回指向末尾的逆向迭代器 |

**示例：**
```cpp
std::span<int> s = {1, 2, 3, 4, 5};
for (auto it = s.begin(); it != s.end(); ++it) {
    std::cout << *it << " ";
}
for (auto rit = s.rbegin(); rit != s.rend(); ++rit) {
    std::cout << *rit << " ";
}
```

**实现原理：**
- 迭代器通常实现为指针或简单的封装，支持随机访问。
- `begin()` 返回指向 `_ptr` 的迭代器，`end()` 返回指向 `_ptr + _size` 的迭代器。

**线程安全提示：**
遍历过程中，如果另一个线程修改了底层数据，迭代器不会失效（因为 `span` 不拥有数据），但可能读取到修改中的值，产生数据竞争。

---

### 5. 观察器

| 函数 | 说明 |
|------|------|
| `size()` | 返回元素数量 |
| `size_bytes()` | 返回序列的字节大小（`size() * sizeof(T)`） |
| `empty()` | 检查序列是否为空 |

**示例：**
```cpp
std::span<int> s = {1, 2, 3, 4, 5};
std::cout << s.size();         // 5
std::cout << s.size_bytes();   // 20 (5 * 4)
std::cout << s.empty();        // false
```

**实现原理：**
- `size()` 返回存储的大小（对于动态范围）或模板参数 `Extent`（对于静态范围）。
- `size_bytes()` 返回 `size() * sizeof(T)`。
- `empty()` 返回 `size() == 0`。

**时间复杂度：**
所有观察器操作都是 O(1)。

**线程安全提示：**
只读操作，线程安全。

---

### 6. 子视图操作

| 函数 | 说明 |
|------|------|
| `first(size_type Count)` | 返回包含前 `Count` 个元素的子 span |
| `last(size_type Count)` | 返回包含后 `Count` 个元素的子 span |
| `subspan(size_type Offset, size_type Count = dynamic_extent)` | 返回从指定偏移量开始的子 span |

**示例：**
```cpp
std::span<int> s = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
auto first_3 = s.first(3);           // {1, 2, 3}
auto last_2 = s.last(2);             // {9, 10}
auto mid = s.subspan(2, 4);          // {3, 4, 5, 6}
auto rest = s.subspan(5);            // {6, 7, 8, 9, 10} (从索引5到末尾)
```

**实现原理：**
- `first(N)`：返回 `span(data(), N)`。
- `last(N)`：返回 `span(data() + size() - N, N)`。
- `subspan(Offset, Count)`：返回 `span(data() + Offset, Count)`（如果 `Count` 为 `dynamic_extent`，则大小为 `size() - Offset`）。

**时间复杂度：**
所有子视图操作都是 O(1)，不复制数据。

**线程安全提示：**
子视图共享底层数据，对底层数据的修改会影响所有视图。

---

## 五、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==(const span& lhs, const span& rhs)` | 比较两个 span 是否相等（逐个元素比较） |
| `operator!=(const span& lhs, const span& rhs)` | 比较两个 span 是否不相等 |
| `operator<(const span& lhs, const span& rhs)` | 按字典序比较（C++20 起） |
| `operator<=(const span& lhs, const span& rhs)` | 按字典序比较（C++20 起） |
| `operator>(const span& lhs, const span& rhs)` | 按字典序比较（C++20 起） |
| `operator>=(const span& lhs, const span& rhs)` | 按字典序比较（C++20 起） |
| `as_bytes(span<T> s)` | 将 `span<T>` 转换为 `span<const std::byte>`，用于字节级操作 |
| `as_writable_bytes(span<T> s)` | 将 `span<T>` 转换为 `span<std::byte>`，用于字节级操作（要求 T 不是 const 限定） |

**示例（字节级操作）：**
```cpp
#include <span>
#include <cstddef>

std::vector<int> vec = {1, 2, 3};
std::span<int> s(vec);
auto bytes = std::as_bytes(s);              // 只读字节视图
auto writable_bytes = std::as_writable_bytes(s);  // 可写字节视图
```

**注意：** 比较运算符和字节转换函数通常定义在 `<span>` 头文件中，作为非成员函数。

---

## 六、宏与常量

`<span>` 头文件中**没有定义任何宏**。主要常量包括：

| 常量 | 说明 |
|------|------|
| `std::dynamic_extent` | 表示动态范围大小的常量，通常为 `static_cast<std::size_t>(-1)` |
| `std::span<T>::extent` | 静态成员常量，表示 span 的大小（动态时为 `dynamic_extent`） |

---

## 七、实现原理

`std::span` 的典型实现仅包含两个核心成员：

```cpp
template <typename T, size_t Extent = dynamic_extent>
class span {
    T* _ptr;      // 指向连续内存首地址的指针
    size_t _size; // 当前视图包含的元素数量（仅当范围为动态时存在）
};
```

- 当 `Extent != dynamic_extent` 时，`_size` 成员不存在（通过模板特化或条件编译实现），大小在编译时已知。
- 当 `Extent == dynamic_extent` 时，`_size` 成员存储运行时大小。
- 在 64 位系统下，每个 `span` 实例通常仅占用 16 字节内存（8 字节指针 + 8 字节大小）。
- 无虚函数/继承，避免虚函数表带来的内存开销和运行时损耗。
- 底层数据必须满足连续内存布局，因此 `span` 不能用于 `std::list` 或 `std::deque` 等非连续容器。
- `span` 的每个特化都是可平凡复制类型（C++23 起）。

**内存模型：**
```
内存地址: | 0x1000 | 0x1004 | 0x1008 | 0x100C | ...
元素:     | T[0]   | T[1]   | T[2]   | T[3]   | ...
```
- `span[i]` 的访问通过 `_ptr + i * sizeof(T)` 实现，时间复杂度 O(1)。
- `span` 本身不分配内存，不拥有数据，不释放数据。

---

## 八、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 随机访问（`operator[]`、`at()`、`front()`、`back()`） | O(1) |
| 迭代器操作（`begin`、`end` 等） | O(1) |
| 子视图操作（`first`、`last`、`subspan`） | O(1) |
| `size()`、`size_bytes()`、`empty()` | O(1) |
| 比较操作（`==`、`!=`、`<` 等） | O(n)（n 为元素个数） |

---

## 九、各标准版本新增特性

### C++20
- 首次引入 `std::span`。

### C++23
- `std::span` 的每个特化都是可平凡复制类型。
- 添加 `const_iterator` 和 `const_reverse_iterator` 成员类型。
- 添加 `begin()`、`end()`、`cbegin()`、`cend()`、`rbegin()`、`rend()`、`crbegin()`、`crend()` 成员函数。
- 添加 `front()` 成员函数。
- 添加 `size_bytes()` 成员函数。

### C++26
- 添加 `back()` 成员函数。
- 添加 `at()` 成员函数（带边界检查）。

---

## 十、迭代器失效规则

`std::span` 本身不拥有数据，因此迭代器失效规则与底层数据相关：

- **底层数据被重新分配**：如果底层容器（如 `std::vector`）重新分配内存，所有指向该数据的 `span` 和迭代器都会失效（因为 `data()` 返回的指针已改变）。
- **底层数据被修改**：如果底层数据被修改（如通过 `span` 或原始指针），迭代器不会失效（仍然指向同一位置），但读取到的值可能已改变。
- **底层数据被销毁**：如果底层数据被销毁，所有指向该数据的 `span` 和迭代器都会失效（变为悬空指针）。

**最佳实践：** 确保 `span` 的生命周期不超过底层数据的生命周期，避免悬空引用。

---

## 十一、相关结构体与类型特征

| 组件 | 说明 |
|------|------|
| `std::dynamic_extent` | 表示动态范围大小的常量 |
| `std::as_bytes()` | 将 `span<T>` 转换为 `span<const std::byte>` |
| `std::as_writable_bytes()` | 将 `span<T>` 转换为 `span<std::byte>` |
| `std::span<T>::extent` | 静态成员常量，表示 span 的大小（动态时为 `dynamic_extent`） |

---


## 十二、补充说明

### 与 `std::string_view` 的对比
- `std::string_view` 专用于字符序列，而 `std::span` 是更通用的连续内存视图。
- `std::span` 支持静态范围（编译时大小），可以进行更激进的优化。
- `std::span` 支持字节级操作（`as_bytes` 和 `as_writable_bytes`）。

### 最佳实践
- 函数参数中使用 `std::span<T>` 替代 `const std::vector<T>&` 或 `T* + size_t`，以获得更好的通用性和安全性。
- 使用 `std::span<const T>` 表示只读视图，明确表达非拥有语义。
- 使用 `std::as_bytes` 进行序列化或网络传输等字节级操作。
- 注意确保 `span` 的生命周期不超过底层数据的生命周期。

### 性能特点
- 零成本抽象：不存储数据，仅包含指针和长度信息（通常为 2 个机器字）。
- 边界安全：自动携带长度信息，避免 C 风格数组的越界风险。
- 统一接口：适配 `std::vector`、原生数组、`std::array` 等多种容器。