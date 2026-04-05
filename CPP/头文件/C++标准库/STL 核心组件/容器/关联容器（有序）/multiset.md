## `<set>` 头文件（含 `std::multiset`）详解

`<set>` 头文件提供了两个有序关联容器：`std::set`（键唯一）和 `std::multiset`（键允许重复）。本章节重点说明 `std::multiset`，它与 `std::set` 的区别在于**允许键重复**，因此接口行为有所不同。

---

## 一、模板参数

```cpp
template< class Key, class Compare = std::less<Key>,
          class Allocator = std::allocator<Key> >
class multiset;
```

| 参数 | 说明 |
|------|------|
| `Key` | 元素的类型（键即值） |
| `Compare` | 比较函数对象类型，用于对元素排序，默认为 `std::less<Key>`（升序） |
| `Allocator` | 分配器类型，用于管理节点内存，默认为 `std::allocator<Key>` |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class Key, class Compare = std::less<Key> >
    using multiset = std::multiset<Key, Compare, std::pmr::polymorphic_allocator<Key>>;
}
```

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `key_type` | `Key` |
| `value_type` | `Key`（与 `key_type` 相同） |
| `key_compare` | `Compare` |
| `value_compare` | `Compare`（因为键值相同） |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*`（或 `Allocator::pointer`） |
| `const_pointer` | `const T*` |
| `iterator` | 双向迭代器，指向 `const value_type`（**注意：`multiset` 的迭代器是常量迭代器**，因为修改元素会破坏排序） |
| `const_iterator` | 常量双向迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

**注意：** `multiset` 的 `iterator` 实际上指向常量元素，因此不能通过迭代器修改元素值。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `multiset()` | 默认构造一个空 multiset |
| `explicit multiset(const Compare& comp, const Allocator& alloc = Allocator())` | 指定比较器和分配器 |
| `template<class InputIt> multiset(InputIt first, InputIt last, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 范围构造 |
| `multiset(const multiset& other)` | 拷贝构造 |
| `multiset(multiset&& other)` | 移动构造（C++11） |
| `multiset(std::initializer_list<value_type> init, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 初始化列表构造（C++11） |

**示例：**
```cpp
std::multiset<int> ms1;
std::multiset<int> ms2 = {1, 2, 2, 3, 3, 3};  // 包含重复元素
```

**实现原理：** 基于红黑树，允许重复键的插入通常将新节点插入到相等键的右侧。

**线程安全提示：** 构造和析构应在单线程环境进行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const multiset& other)` | 拷贝赋值 |
| `operator=(multiset&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

---

### 3. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（按排序规则的最小元素） |
| `end()` / `cend()` | 返回尾后迭代器 |
| `rbegin()` / `crbegin()` | 反向迭代器 |
| `rend()` / `crend()` | 反向尾后迭代器 |

**示例：**
```cpp
std::multiset<int> ms = {3,1,4,1,5};
for (int x : ms) std::cout << x << " ";  // 输出: 1 1 3 4 5 （升序）
```

**实现原理：** 迭代器按照红黑树的中序遍历顺序访问元素。

---

### 4. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查是否为空 |
| `size()` | 返回元素个数 |
| `max_size()` | 返回最大可能元素个数 |

**时间复杂度：** O(1)

---

### 5. 修改器

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
| `erase(const key_type& key)` | 移除所有等于 `key` 的元素，**返回被移除的元素个数** |
| `swap(multiset& other)` | 交换内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(multiset& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::multiset<int> ms = {1,2,2,3};
ms.insert(2);                         // 插入成功，返回迭代器
ms.insert({2,4});
size_t n = ms.erase(2);               // 删除所有 2，n == 3
```

**实现原理：** 红黑树插入时允许重复，通常插入到相等键的右侧。

**时间复杂度：**
- `insert` / `emplace`：O(log n)
- `erase(iterator)`：均摊 O(1)
- `erase(key)`：O(log n + count)

---

### 6. 查找

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
std::multiset<int> ms = {1,2,2,3,3,3};
auto range = ms.equal_range(2);
for (auto it = range.first; it != range.second; ++it) {
    std::cout << *it << " ";   // 输出: 2 2
}
```

**时间复杂度：** O(log n)

---

### 7. 观察器

| 函数 | 说明 |
|------|------|
| `key_comp()` | 返回键比较函数 |
| `value_comp()` | 返回值比较函数（同 `key_comp`） |

---

### 8. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 multiset |
| `swap(std::multiset& lhs, std::multiset& rhs)` | 交换两个 multiset |
| `erase_if(std::multiset& c, Predicate pred)` | 移除所有满足谓词的元素（C++20） |

---

## 五、宏与常量

`<set>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::multiset` 基于**红黑树**实现，与 `std::set` 共享底层数据结构，唯一区别是插入时允许键相等。通常实现中，比较器使用 `std::less<Key>`，当键相等时认为“不小于”且“不大于”，因此插入位置在相等键的右侧。

**节点结构（简化）：**
```cpp
struct Node {
    Key value;
    Node* left;
    Node* right;
    Node* parent;
    bool color;   // 红/黑
};
```

**插入重复键：** 红黑树查找过程中，当遇到相等键时，继续向右走，最终插入到所有相等键的最右侧。

**`equal_range` 实现：** `lower_bound` 返回第一个不小于 key 的位置（即第一个等于 key 或第一个大于 key），`upper_bound` 返回第一个大于 key 的位置，两者之间的范围即为所有等于 key 的元素。

**与 `std::set` 的主要区别：**
- `insert` 返回 `iterator`（而不是 `pair<iterator,bool>`），因为插入总是成功。
- `count` 可以返回大于 1 的值。
- `extract` 和 `merge` 的行为类似，但允许重复键。
- 没有 `operator[]`（因为键即值，且允许重复）。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 查找（`find`、`lower_bound`、`upper_bound`、`equal_range`） | O(log n) |
| 插入（`insert`、`emplace`） | O(log n) |
| 带提示的插入（提示正确） | 均摊 O(1) |
| 删除单个元素（迭代器） | 均摊 O(1) |
| 删除所有等于键的元素 | O(log n + k)，k 为删除个数 |
| `size`、`empty` | O(1) |
| `swap` | O(1) |

---

## 八、线程安全

**`std::multiset` 不是线程安全的**。规则：
- **同时读取**：多个线程同时读取是安全的。
- **读取 + 写入**：一个线程写入（插入、删除）时，其他线程不能进行任何操作，否则数据竞争。
- **同时写入**：多个线程同时写入会导致未定义行为。

**保证线程安全的常用方法：**
- 使用 `std::mutex` 或 `std::shared_mutex` 保护整个 multiset。
- 使用线程局部存储。
- 使用并发容器（如 `tbb::concurrent_unordered_multiset`）。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace`、`emplace_hint` 原位构造。
- 初始化列表构造和赋值。
- `cbegin()` / `cend()` / `crbegin()` / `crend()`。

### C++14
- 无显著新特性。

### C++17
- `extract` 节点提取操作。
- `merge` 合并操作。
- `std::pmr::multiset` 多态分配器别名。

### C++20
- `contains` 成员函数。
- `operator<=>` 三路比较支持。
- `erase_if` 非成员函数。
- `constexpr` 修饰部分成员函数（有限支持）。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `insert` | 不影响已有迭代器 |
| `emplace` | 不影响已有迭代器 |
| `erase(iterator pos)` | 仅被删除的迭代器失效 |
| `erase(const key_type& key)` | 所有指向被删除元素的迭代器失效 |
| `clear` | 所有迭代器失效 |
| `extract` | 被提取节点的迭代器失效，但节点指针仍有效 |
| `merge` | 被移动节点的迭代器在原容器中失效，在目标容器中重新有效 |
| `swap` | 所有迭代器失效（指向其他容器） |

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::less<Key>` | 默认比较器 |
| `std::allocator<Key>` | 默认分配器 |
| `std::pmr::polymorphic_allocator` | 多态分配器（C++17） |

---

（与 `std::set` 同属有序关联容器，支持重复键。）