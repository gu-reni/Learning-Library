## `<map>` 头文件（含 `std::multimap`）详解

`<map>` 头文件提供了两个关联容器：`std::map` 和 `std::multimap`。本章节重点说明 `std::multimap`，它与 `std::map` 的区别在于**允许键重复**，因此某些接口行为不同。以下内容基于 C++11 及以上标准。

---

## 一、模板参数

```cpp
template< class Key, class T, class Compare = std::less<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class multimap;
```

| 参数 | 说明 |
|------|------|
| `Key` | 键的类型 |
| `T` | 值的类型 |
| `Compare` | 比较函数对象类型，用于对键进行排序，默认为 `std::less<Key>`（升序） |
| `Allocator` | 分配器类型，用于管理节点内存，默认为 `std::allocator<std::pair<const Key, T>>` |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class Key, class T, class Compare = std::less<Key> >
    using multimap = std::multimap<Key, T, Compare, std::pmr::polymorphic_allocator<std::pair<const Key, T>>>;
}
```

---

## 二、成员类型

与 `std::map` 基本相同，但 `std::multimap` 没有 `mapped_type` 的单独别名？实际上有，同 map。列出：

| 类型别名 | 说明 |
|----------|------|
| `key_type` | `Key` |
| `mapped_type` | `T` |
| `value_type` | `std::pair<const Key, T>` |
| `key_compare` | `Compare` |
| `value_compare` | 嵌套比较器，通过键比较 `value_type` |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*` |
| `const_pointer` | `const T*` |
| `iterator` | 双向迭代器，指向 `value_type` |
| `const_iterator` | 常量双向迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

与 `std::map` 完全一致：

| 构造函数 | 说明 |
|----------|------|
| `multimap()` | 默认构造一个空 multimap |
| `explicit multimap(const Compare& comp, const Allocator& alloc = Allocator())` | 指定比较器和分配器 |
| `template<class InputIt> multimap(InputIt first, InputIt last, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 范围构造 |
| `multimap(const multimap& other)` | 拷贝构造 |
| `multimap(multimap&& other)` | 移动构造（C++11） |
| `multimap(std::initializer_list<value_type> init, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 初始化列表构造（C++11） |

**示例：**
```cpp
std::multimap<int, std::string> mm1;
std::multimap<int, std::string> mm2 = {{1,"one"}, {1,"uno"}, {2,"two"}};
```

**实现原理：** 基于红黑树，允许重复键的插入通常将新节点插入到相等键的右侧。

**线程安全提示：** 构造和析构应在单线程环境进行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const multimap& other)` | 拷贝赋值 |
| `operator=(multimap&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

---

### 3. 元素访问

**注意：** `std::multimap` **没有** `operator[]` 和 `at()` 成员函数，因为键不唯一，无法确定返回哪个值。

---

### 4. 迭代器

与 `std::map` 相同，迭代器按**键升序**遍历（相同键的元素按插入顺序？标准未规定，但通常按插入顺序或实现定义）。

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器 |
| `end()` / `cend()` | 返回尾后迭代器 |
| `rbegin()` / `crbegin()` | 反向迭代器 |
| `rend()` / `crend()` | 反向尾后迭代器 |

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查是否为空 |
| `size()` | 返回元素个数 |
| `max_size()` | 返回最大可能元素个数 |

---

### 6. 修改器

与 `std::map` 类似，但 **`insert` 返回类型不同**，且没有 `operator[]` 和 `at`。

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert(const value_type& value)` | 插入元素，**返回指向新元素的迭代器**（而不是 `pair<iterator,bool>`） |
| `insert(value_type&& value)` | 插入右值（C++11） |
| `insert(iterator hint, const value_type& value)` | 带提示的插入，返回迭代器 |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素，返回指向新元素的迭代器（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示原位构造（C++11） |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除范围 |
| `erase(const key_type& key)` | 移除所有键等于 `key` 的元素，**返回被移除的元素个数** |
| `swap(multimap& other)` | 交换内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(multimap& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::multimap<int, std::string> mm;
auto it = mm.insert({1, "one"});   // 返回 iterator
mm.insert({1, "uno"});
mm.insert({2, "two"});
size_t n = mm.erase(1);            // 删除所有键为 1 的元素，n == 2
```

**实现原理：** 红黑树插入时允许键重复，通常将新节点插入到相同键的右侧。

**时间复杂度：**
- `insert` / `emplace`：O(log n)
- `erase(iterator)`：均摊 O(1)
- `erase(key)`：O(log n + count)

---

### 7. 查找

与 `std::map` 类似，但 `count` 和 `equal_range` 对重复键更有意义。

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 返回指向第一个键等于 `key` 的元素的迭代器，若不存在返回 `end()` |
| `count(const Key& key)` | 返回键等于 `key` 的元素个数 |
| `contains(const Key& key)` | 检查是否存在键等于 `key` 的元素（C++20） |
| `lower_bound(const Key& key)` | 返回第一个键**不小于** `key` 的迭代器 |
| `upper_bound(const Key& key)` | 返回第一个键**大于** `key` 的迭代器 |
| `equal_range(const Key& key)` | 返回 `pair<iterator, iterator>`，表示键等于 `key` 的范围 |

**示例：**
```cpp
std::multimap<int, std::string> mm = {{1,"one"}, {1,"uno"}, {2,"two"}};
auto range = mm.equal_range(1);
for (auto it = range.first; it != range.second; ++it) {
    std::cout << it->second << " ";   // "one uno"
}
```

**时间复杂度：** O(log n)

---

### 8. 观察器

| 函数 | 说明 |
|------|------|
| `key_comp()` | 返回键比较函数 |
| `value_comp()` | 返回值比较函数（通过键） |

---

### 9. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 multimap |
| `swap(std::multimap& lhs, std::multimap& rhs)` | 交换两个 multimap |

---

## 五、宏与常量

`<map>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::multimap` 同样基于**红黑树**实现，与 `std::map` 共享相同的底层数据结构，唯一区别是插入时允许键相等。通常实现中，比较器使用 `std::less<Key>`，当键相等时认为“不小于”且“不大于”，因此插入位置在相等键的右侧。

**节点结构：** 与 `std::map` 相同，存储 `std::pair<const Key, T>`。

**插入重复键：** 红黑树查找过程中，当遇到相等键时，继续向右走，最终插入到所有相等键的最右侧。

**`equal_range` 实现：** `lower_bound` 返回第一个不小于 key 的位置（即第一个等于 key 或第一个大于 key），`upper_bound` 返回第一个大于 key 的位置，两者之间的范围即为所有等于 key 的元素。

---

## 七、时间复杂度

与 `std::map` 相同：

| 操作 | 时间复杂度 |
|------|------------|
| 查找（`find`、`lower_bound` 等） | O(log n) |
| 插入（`insert`、`emplace`） | O(log n) |
| 带提示的插入（提示正确） | 均摊 O(1) |
| 删除单个元素（迭代器） | 均摊 O(1) |
| 删除所有等于键的元素 | O(log n + k)，k 为删除个数 |
| `size`、`empty` | O(1) |
| `swap` | O(1) |

---

## 八、线程安全

与 `std::map` 相同，**不是线程安全的**。并发读写需要外部同步（如互斥锁）。

---

## 九、各标准版本新增特性

与 `std::map` 同步：

### C++11
- 移动语义、原位构造、初始化列表。

### C++17
- `extract`、`merge`、`pmr` 别名。

### C++20
- `contains`、`erase_if`、三路比较。

---

## 十、迭代器失效规则

与 `std::map` 相同：
- `insert` 不影响已有迭代器。
- `erase` 仅使被删除的迭代器失效。
- `clear` 使所有迭代器失效。
- `swap` 使所有迭代器失效（指向其他容器）。

---

## 十一、相关结构体

- `std::pair<const Key, T>`
- `std::less<Key>`
- `std::allocator`
- `std::pmr::polymorphic_allocator`

---

（`std::multimap` 与 `std::map` 同属有序关联容器，支持重复键。）