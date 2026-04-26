## `<iterator>` 头文件详解

`<iterator>` 是 C++ 标准模板库（STL）中定义的头文件，提供了**迭代器**的概念、类别标签、辅助函数、迭代器适配器以及预定义的迭代器类型（如 `back_insert_iterator`、`istream_iterator` 等）。迭代器是 STL 的核心组件，它抽象了容器元素的访问方式，使得算法可以独立于特定容器而工作。该头文件定义了迭代器必需的类型别名（`iterator_traits`）、常用操作函数（`advance`、`distance`、`next`、`prev`）、以及用于简化迭代器编写的基类模板等。

---

## 一、迭代器类别标签

这些空结构体用于标记迭代器所属的类别，在重载与特化中作为标签派发。

| 标签类型 | 说明 |
|----------|------|
| `std::input_iterator_tag` | 输入迭代器（只读、单遍扫描） |
| `std::output_iterator_tag` | 输出迭代器（只写、单遍扫描） |
| `std::forward_iterator_tag` | 前向迭代器（读写、多遍扫描） |
| `std::bidirectional_iterator_tag` | 双向迭代器（支持递减） |
| `std::random_access_iterator_tag` | 随机访问迭代器（支持 `+`、`-`、`[]` 等） |
| `std::contiguous_iterator_tag`（C++20） | 连续迭代器（元素在内存中连续，如指针） |

---

## 二、特性和型别

### `std::iterator_traits`

**定义：**
```cpp
template<class Iterator>
struct iterator_traits {
    using difference_type   = typename Iterator::difference_type;
    using value_type        = typename Iterator::value_type;
    using pointer           = typename Iterator::pointer;
    using reference         = typename Iterator::reference;
    using iterator_category = typename Iterator::iterator_category;
};

// 对指针类型的偏特化
template<class T>
struct iterator_traits<T*> {
    using difference_type   = ptrdiff_t;
    using value_type        = T;
    using pointer           = T*;
    using reference         = T&;
    using iterator_category = random_access_iterator_tag;
};
```

**作用：** 提供统一的方式访问迭代器的关联类型。所有泛型算法都通过 `iterator_traits` 获取迭代器的属性，使得算法可以适用于任何迭代器类型（包括原生指针）。

---

## 三、迭代器辅助函数

这些函数用于在不直接使用迭代器算术的情况下执行操作，提高泛型代码的可移植性。

| 函数 | 作用 |
|------|------|
| `std::advance(it, n)` | 将迭代器 `it` 前进（或后退）`n` 步。对随机访问迭代器为 O(1)，否则为 O(n)。 |
| `std::distance(first, last)` | 计算从 `first` 到 `last` 的步数。对随机访问迭代器为 O(1)，否则为 O(n)。 |
| `std::next(it, n = 1)` | 返回迭代器前进 `n` 步后的新迭代器（不修改原迭代器）。 |
| `std::prev(it, n = 1)` | 返回迭代器后退 `n` 步后的新迭代器（不修改原迭代器，要求双向迭代器）。 |

**示例用法：**
```cpp
#include <iterator>
#include <vector>
#include <iostream>
int main() {
    std::vector<int> v = {1,2,3,4,5};
    auto it = v.begin();
    std::advance(it, 2);              // it 指向 3
    auto dist = std::distance(v.begin(), it); // 2
    auto nxt = std::next(it);         // 指向 4
    auto prv = std::prev(it);         // 指向 2
}
```

---

## 四、迭代器适配器

这些类模板将现有迭代器转换为具有不同行为的迭代器，例如反向迭代、插入迭代器等。

### 1. 反向迭代器 `std::reverse_iterator`

**定义：**
```cpp
template<class Iterator>
class reverse_iterator;
```

**作用：** 将底层迭代器方向反转，`rbegin()` 和 `rend()` 通常返回此类对象。

**获取方式：** 通常通过容器的 `rbegin()` / `rend()` 成员函数获得，也可以显式构造：`std::reverse_iterator<Iter>(iter)`。

**示例：**
```cpp
std::vector<int> v = {1,2,3};
auto rit = std::make_reverse_iterator(v.end()); // 指向 3
```

---

### 2. 插入迭代器

| 适配器 | 作用 |
|--------|------|
| `std::back_insert_iterator` | 通过 `push_back` 在容器末尾插入元素。需容器支持 `push_back`。 |
| `std::front_insert_iterator` | 通过 `push_front` 在容器头部插入元素。需容器支持 `push_front`。 |
| `std::insert_iterator` | 通过 `insert` 在指定位置插入元素。 |

**辅助函数：**
- `std::back_inserter(container)` → 返回 `back_insert_iterator`
- `std::front_inserter(container)` → 返回 `front_insert_iterator`
- `std::inserter(container, pos)` → 返回 `insert_iterator`

**示例：**
```cpp
#include <iterator>
#include <vector>
#include <algorithm>
std::vector<int> src = {1,2,3};
std::vector<int> dest;
std::copy(src.begin(), src.end(), std::back_inserter(dest));
```

---

### 3. 流迭代器

| 适配器 | 作用 |
|--------|------|
| `std::istream_iterator<T>` | 从输入流（如 `std::cin`）读取 `T` 类型对象。 |
| `std::ostream_iterator<T>` | 向输出流（如 `std::cout`）写入 `T` 类型对象和分隔符。 |
| `std::istreambuf_iterator<char>` | 读取原始字符流（不解析格式化）。 |
| `std::ostreambuf_iterator<char>` | 写入原始字符流。 |

**示例：**
```cpp
#include <iterator>
#include <iostream>
#include <vector>
std::vector<int> v{std::istream_iterator<int>(std::cin),
                   std::istream_iterator<int>()};
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "));
```

---

### 4. 移动迭代器 `std::move_iterator`（C++11）

**定义：**
```cpp
template<class Iterator>
class move_iterator;
```

**作用：** 将底层迭代器的解引用转换为右值引用，从而移动元素而非复制。可用于 `std::copy` 等算法中实现移动语义。

**辅助函数：** `std::make_move_iterator(it)`。

**示例：**
```cpp
std::vector<std::string> src = {"a","b"};
std::vector<std::string> dest;
std::move(src.begin(), src.end(), std::back_inserter(dest));
// src 中的字符串现在处于“被移动”状态
```

---

## 五、预定义迭代器

### `std::default_sentinel_t`（C++20）

**作用：** 与某些范围适配器一起使用，表示“无迭代器”的哨兵。

---

### `std::counted_iterator`（C++20）

**定义：**
```cpp
template<std::input_or_output_iterator I>
class counted_iterator;
```

**作用：** 包装一个迭代器和一个计数，表示最多访问 N 步，配合 `std::default_sentinel` 使用。可用于范围适配器 `std::views::counted`。

---

## 六、迭代器基类（C++17 前用于简化定义）

在 C++17 之前，常通过继承 `std::iterator` 来定义新迭代器类型，但 C++17 已废弃该模板。

```cpp
template<class Category, class T, class Distance = ptrdiff_t,
         class Pointer = T*, class Reference = T&>
struct iterator {
    using iterator_category = Category;
    using value_type        = T;
    using difference_type   = Distance;
    using pointer           = Pointer;
    using reference         = Reference;
};
```

**现在推荐自行定义所需的 typedef 或使用 `iterator_traits` 协议。**

---

## 七、宏与常量

`<iterator>` 头文件中没有定义任何宏。

---

## 八、类型定义汇总

| 类型 | 说明 |
|------|------|
| `std::input_iterator_tag` | 空标签类型 |
| `std::output_iterator_tag` | 空标签类型 |
| `std::forward_iterator_tag` | 空标签类型 |
| `std::bidirectional_iterator_tag` | 空标签类型 |
| `std::random_access_iterator_tag` | 空标签类型 |
| `std::contiguous_iterator_tag` | 空标签类型（C++20） |
| `std::iterator_traits<Iter>` | 迭代器特征类模板 |
| `std::reverse_iterator<Iter>` | 反向迭代器类模板 |
| `std::move_iterator<Iter>` | 移动迭代器类模板（C++11） |
| `std::back_insert_iterator<Container>` | 尾部插入迭代器类模板 |
| `std::front_insert_iterator<Container>` | 头部插入迭代器类模板 |
| `std::insert_iterator<Container>` | 任意位置插入迭代器类模板 |
| `std::istream_iterator<T>` | 输入流迭代器类模板 |
| `std::ostream_iterator<T>` | 输出流迭代器类模板 |
| `std::istreambuf_iterator<CharT>` | 输入流缓冲区迭代器类模板 |
| `std::ostreambuf_iterator<CharT>` | 输出流缓冲区迭代器类模板 |
| `std::counted_iterator<I>` | 计数迭代器类模板（C++20） |
| `std::default_sentinel_t` | 哨兵类型（C++20） |

---

## 九、模板声明

`<iterator>` 包含大量模板类和函数。除上述提到的模板外，还有：
- `std::begin` / `std::end`（C++11）及其 `const` 版本、`cbegin`/`cend`、`rbegin`/`rend`、`crbegin`/`crend`（用于数组和容器）。
- `std::size`、`std::empty`、`std::data`（C++17）：与 `std::begin` 类似，用于获取大小和指针。

这些函数通常与容器和内置数组一起使用。

---
