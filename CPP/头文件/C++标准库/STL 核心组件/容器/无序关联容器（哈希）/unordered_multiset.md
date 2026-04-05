## `<unordered_set>` 头文件（`std::unordered_multiset`）详解

`<unordered_set>` 是 C++11 标准引入的头文件，提供了两个无序关联容器：`std::unordered_set` 和 `std::unordered_multiset`。`std::unordered_multiset` 是一个基于哈希表的无序集合，**允许元素重复**。它不保持元素的特定顺序，插入、删除和查找操作的平均时间复杂度为 O(1)，最坏情况 O(n)。

---

## 一、模板参数

```cpp
template< class Key,
          class Hash = std::hash<Key>,
          class KeyEqual = std::equal_to<Key>,
          class Allocator = std::allocator<Key> >
class unordered_multiset;
```

| 参数 | 说明 |
|------|------|
| `Key` | 元素的类型（键即值） |
| `Hash` | 哈希函数对象类型，默认为 `std::hash<Key>` |
| `KeyEqual` | 键相等比较函数对象类型，默认为 `std::equal_to<Key>` |
| `Allocator` | 分配器类型，默认为 `std::allocator<Key>` |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class Key, class Hash = std::hash<Key>, class KeyEqual = std::equal_to<Key> >
    using unordered_multiset = std::unordered_multiset<Key, Hash, KeyEqual, std::pmr::polymorphic_allocator<Key>>;
}
```

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `key_type` | `Key` |
| `value_type` | `Key`（与 `key_type` 相同） |
| `hasher` | `Hash` |
| `key_equal` | `KeyEqual` |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*`（或 `Allocator::pointer`） |
| `const_pointer` | `const T*` |
| `iterator` | 前向迭代器，指向 `const value_type`（不可修改元素） |
| `const_iterator` | 常量前向迭代器 |
| `local_iterator` | 用于桶内遍历的迭代器 |
| `const_local_iterator` | 常量桶内迭代器 |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `unordered_multiset()` | 默认构造空容器 |
| `explicit unordered_multiset(size_type bucket_count, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 指定初始桶数、哈希函数、相等比较器和分配器 |
| `template<class InputIt> unordered_multiset(InputIt first, InputIt last, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 范围构造 |
| `unordered_multiset(const unordered_multiset& other)` | 拷贝构造 |
| `unordered_multiset(unordered_multiset&& other)` | 移动构造（C++11） |
| `unordered_multiset(std::initializer_list<value_type> init, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~unordered_multiset()` | 销毁所有元素并释放内存 |

**示例：**
```cpp
std::unordered_multiset<int> ms1;                                    // 空
std::unordered_multiset<int> ms2 = {1, 2, 2, 3, 3, 3};              // 包含重复元素
std::unordered_multiset<int> ms3(ms2);                              // 拷贝构造
```

**实现原理：** 基于哈希表（开链法），桶数组管理链表。

**线程安全提示：** 构造和析构应在单线程环境进行，或有锁保护。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const unordered_multiset& other)` | 拷贝赋值 |
| `operator=(unordered_multiset&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

---

### 3. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（顺序未指定） |
| `end()` / `cend()` | 返回尾后迭代器 |
| `local_iterator begin(size_type n)` | 返回第 `n` 个桶的起始局部迭代器 |
| `local_iterator end(size_type n)` | 返回第 `n` 个桶的尾后局部迭代器 |

**示例：**
```cpp
std::unordered_multiset<int> ms = {1,2,2,3};
for (int x : ms) std::cout << x << " ";   // 输出顺序不确定
```

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
| `insert(const value_type& value)` | 插入元素，**返回指向新元素的迭代器**（因为插入总是成功） |
| `insert(value_type&& value)` | 插入右值（C++11） |
| `insert(iterator hint, const value_type& value)` | 带提示插入 |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素，返回指向新元素的迭代器（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示原位构造 |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除范围 |
| `erase(const key_type& key)` | 移除所有等于 `key` 的元素，**返回被删除的元素个数** |
| `swap(unordered_multiset& other)` | 交换内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(unordered_multiset& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::unordered_multiset<int> ms;
auto it = ms.insert(42);           // 插入成功，返回 iterator
ms.insert(42);                     // 再次插入成功
size_t n = ms.erase(42);           // 删除所有 42，n == 2
```

**实现原理：**
- 插入时计算哈希值，定位桶，将新节点插入到桶的链表中（不检查重复）。
- 删除键时遍历桶内链表，删除所有匹配的节点。

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
所有修改操作需要同步。

---

### 6. 查找

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 返回指向第一个等于 `key` 的元素的迭代器，若不存在返回 `end()` |
| `count(const Key& key)` | 返回等于 `key` 的元素个数 |
| `contains(const Key& key)` | 检查是否存在等于 `key` 的元素（C++20） |
| `equal_range(const Key& key)` | 返回 `std::pair<iterator, iterator>`，表示等于 `key` 的范围 |

**示例：**
```cpp
std::unordered_multiset<int> ms = {1,2,2,3};
size_t c = ms.count(2);                      // 2
auto range = ms.equal_range(2);
for (auto it = range.first; it != range.second; ++it) {
    std::cout << *it << " ";                 // 2 2
}
```

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
查找是只读的，多个线程可并发查找（但不能有修改）。

---

### 7. 桶接口

| 函数 | 说明 |
|------|------|
| `size_type bucket_count() const` | 返回桶的数量 |
| `size_type max_bucket_count() const` | 返回最大桶数 |
| `size_type bucket_size(size_type n) const` | 返回第 `n` 个桶中的元素个数 |
| `size_type bucket(const Key& key) const` | 返回键 `key` 所属的桶索引 |
| `local_iterator begin(size_type n)` | 返回第 `n` 个桶的起始局部迭代器 |
| `local_iterator end(size_type n)` | 返回第 `n` 个桶的尾后局部迭代器 |

---

### 8. 哈希策略

| 函数 | 说明 |
|------|------|
| `float load_factor() const` | 返回当前负载因子（`size() / bucket_count()`） |
| `float max_load_factor() const` | 返回最大负载因子 |
| `void max_load_factor(float ml)` | 设置最大负载因子 |
| `void rehash(size_type n)` | 设置桶数至少为 `n`，并重新哈希 |
| `void reserve(size_type n)` | 预留空间，使 `bucket_count() >= n / max_load_factor()` |

**示例：**
```cpp
ms.reserve(100);
ms.max_load_factor(0.8f);
```

---

### 9. 观察器

| 函数 | 说明 |
|------|------|
| `hasher hash_function() const` | 返回哈希函数对象 |
| `key_equal key_eq() const` | 返回键相等比较函数对象 |
| `allocator_type get_allocator() const` | 返回分配器副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=` | 比较两个 unordered_multiset 是否相等（不考虑顺序） |
| `swap(unordered_multiset& lhs, unordered_multiset& rhs)` | 交换内容 |
| `erase_if(unordered_multiset& c, Predicate pred)` | 移除所有满足谓词的元素（C++20） |

**注意：** 比较操作需要逐元素比较，时间复杂度 O(n)。

---

## 五、宏与常量

`<unordered_set>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::unordered_multiset` 基于**开链法（separate chaining）**哈希表：
- 内部维护一个桶数组，每个桶是一个链表。
- 插入时，计算哈希值 `h = hash(key)`，桶索引 `idx = h % bucket_count()`，将新节点插入到链表（通常头部）。不检查重复，插入总是成功。
- 查找时，计算桶索引，遍历链表，使用 `KeyEqual` 比较。
- 当负载因子超过阈值时自动 rehash（增大桶数组，重新分配元素）。

**与 `unordered_set` 的区别：**
- `insert` 返回 `iterator`（而不是 `pair<iterator,bool>`）。
- `count` 可以返回大于 1 的值。
- `equal_range` 可以返回一个范围。
- 没有 `operator[]`（因为键即值，且不唯一）。

---

## 七、时间复杂度

| 操作 | 平均 | 最坏 |
|------|------|------|
| `insert`、`emplace` | O(1) | O(n) |
| `erase`（迭代器） | O(1) | O(1) |
| `erase`（键） | O(1 + count) | O(n) |
| `find`、`count`、`contains` | O(1) | O(n) |
| `equal_range` | O(1 + count) | O(n) |
| `rehash`、`reserve` | O(n) | O(n) |
| `bucket_count`、`size`、`empty` | O(1) | O(1) |

---

## 八、线程安全

**`std::unordered_multiset` 不是线程安全的**。规则：
- **同时读取**：多个线程同时读取是安全的。
- **读取 + 写入**：一个线程写入时，其他线程不能进行任何操作。
- **同时写入**：多个线程同时写入会导致未定义行为。
- **rehash**：写入操作可能触发 rehash，使所有迭代器失效。

**保证线程安全的常用方法：**
- 使用互斥锁保护。
- 使用线程局部存储。
- 使用并发容器。

---

## 九、各标准版本新增特性

### C++11
- 首次引入 `std::unordered_multiset`。

### C++14
- 无显著新特性。

### C++17
- `extract` 和 `merge` 操作。
- `std::pmr::unordered_multiset` 多态分配器别名。

### C++20
- `contains` 成员函数。
- `erase_if` 非成员函数。
- 支持 `constexpr` 部分成员函数（有限）。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `insert` | 若未触发 rehash，所有迭代器有效；若触发 rehash，所有迭代器失效 |
| `emplace` | 同上 |
| `erase(iterator pos)` | 仅被删除元素的迭代器失效 |
| `erase(const key_type& key)` | 所有指向被删除元素的迭代器失效 |
| `clear` | 所有迭代器失效 |
| `rehash` / `reserve` | 所有迭代器失效 |
| `swap` | 所有迭代器失效（指向其他容器） |
| `extract` | 被提取节点的迭代器失效 |
| `merge` | 被移动节点的迭代器在原容器中失效，在目标容器中重新有效 |

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::hash<Key>` | 默认哈希函数 |
| `std::equal_to<Key>` | 默认键相等比较器 |
| `std::allocator<Key>` | 默认分配器 |
| `std::pmr::polymorphic_allocator<Key>` | 多态分配器（C++17） |

---