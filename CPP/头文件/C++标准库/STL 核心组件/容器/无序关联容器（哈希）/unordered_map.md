## `<unordered_map>` 头文件详解

`<unordered_map>` 是 C++11 标准引入的头文件，提供了两个无序关联容器：`std::unordered_map` 和 `std::unordered_multimap`。它们基于哈希表实现，元素不保持特定顺序，插入、删除和查找操作的平均时间复杂度为 O(1)，最坏情况 O(n)。`std::unordered_map` 要求键唯一，`std::unordered_multimap` 允许键重复。

---

## 一、模板参数

### 1. std::unordered_map
```cpp
template< class Key,
          class T,
          class Hash = std::hash<Key>,
          class KeyEqual = std::equal_to<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class unordered_map;
```

| 参数 | 说明 |
|------|------|
| `Key` | 键的类型 |
| `T` | 值的类型 |
| `Hash` | 哈希函数对象类型，默认为 `std::hash<Key>` |
| `KeyEqual` | 键相等比较函数对象类型，默认为 `std::equal_to<Key>` |
| `Allocator` | 分配器类型，默认为 `std::allocator<std::pair<const Key, T>>` |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class Key, class T, class Hash = std::hash<Key>, class KeyEqual = std::equal_to<Key> >
    using unordered_map = std::unordered_map<Key, T, Hash, KeyEqual, std::pmr::polymorphic_allocator<std::pair<const Key, T>>>;
}
```

### 2. std::unordered_multimap
```cpp
template< class Key,
          class T,
          class Hash = std::hash<Key>,
          class KeyEqual = std::equal_to<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class unordered_multimap;
```
模板参数与 `std::unordered_map` 相同，区别在于允许键重复。

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `key_type` | `Key` |
| `mapped_type` | `T` |
| `value_type` | `std::pair<const Key, T>` |
| `hasher` | `Hash` |
| `key_equal` | `KeyEqual` |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*`（或 `Allocator::pointer`） |
| `const_pointer` | `const T*` |
| `iterator` | 前向迭代器，指向 `value_type` |
| `const_iterator` | 常量前向迭代器 |
| `local_iterator` | 用于桶内遍历的迭代器（前向） |
| `const_local_iterator` | 常量桶内迭代器 |

---

## 三、成员函数详解

以下以 `std::unordered_map` 为例，`std::unordered_multimap` 的接口类似，但部分行为不同（如 `insert` 返回类型、`operator[]` 不存在等）。

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `unordered_map()` | 默认构造空容器，桶数量由实现定义 |
| `explicit unordered_map(size_type bucket_count, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 指定初始桶数、哈希函数、相等比较器和分配器 |
| `template<class InputIt> unordered_map(InputIt first, InputIt last, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 范围构造 |
| `unordered_map(const unordered_map& other)` | 拷贝构造 |
| `unordered_map(unordered_map&& other)` | 移动构造（C++11） |
| `unordered_map(std::initializer_list<value_type> init, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~unordered_map()` | 销毁所有元素并释放内存 |

**示例：**
```cpp
std::unordered_map<int, std::string> m1;                                    // 空
std::unordered_map<int, std::string> m2(100);                              // 初始桶数 100
std::unordered_map<int, std::string> m3 = {{1,"one"}, {2,"two"}};          // 初始化列表
std::unordered_map<int, std::string> m4(m3);                               // 拷贝构造
```

**实现原理：**
- 默认构造时，创建一个空的哈希表，桶数组通常为空或很小。
- 拷贝构造时，复制所有元素到新桶数组，并保持相同的哈希策略。
- 移动构造时，直接接管原哈希表的内部数组和状态。

**线程安全提示：**
构造和析构应在单线程环境进行，或有锁保护。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const unordered_map& other)` | 拷贝赋值 |
| `operator=(unordered_map&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

**示例：**
```cpp
std::unordered_map<int, std::string> m1, m2;
m1 = m2;                       // 拷贝赋值
m1 = {{1,"one"}, {2,"two"}};   // 列表赋值
```

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `mapped_type& operator[](const Key& key)` | 返回键 `key` 对应的值的引用；若键不存在，则插入一个值初始化的元素并返回其引用 |
| `mapped_type& operator[](Key&& key)` | 右值版本（C++11） |
| `mapped_type& at(const Key& key)` | 返回键 `key` 对应的值的引用，若键不存在则抛出 `std::out_of_range` |
| `const mapped_type& at(const Key& key) const` | 常量版本 |

**示例：**
```cpp
std::unordered_map<int, std::string> m;
m[1] = "one";                  // 插入键 1
std::string s = m[2];          // 插入键 2（值初始化为空字符串）
std::string t = m.at(3);       // 抛出异常
```

**实现原理：**
- `operator[]`：调用 `try_emplace(key)` 或类似操作，若键不存在则插入默认值。
- `at()`：查找键，若存在则返回引用，否则抛出异常。

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
`operator[]` 可能修改容器，并发使用需要同步。

---

### 4. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（顺序未指定） |
| `end()` / `cend()` | 返回尾后迭代器 |
| `local_iterator begin(size_type n)` | 返回第 `n` 个桶内第一个元素的局部迭代器 |
| `local_iterator end(size_type n)` | 返回第 `n` 个桶的尾后局部迭代器 |

**示例：**
```cpp
std::unordered_map<int, std::string> m = {{1,"one"}, {2,"two"}, {3,"three"}};
for (auto& p : m) {
    std::cout << p.first << ":" << p.second << "\n";
}
// 遍历桶
for (size_t i = 0; i < m.bucket_count(); ++i) {
    for (auto it = m.begin(i); it != m.end(i); ++it) {
        // 处理桶内元素
    }
}
```

**实现原理：**
- 普通迭代器遍历整个哈希表的所有桶和桶内链表。
- 局部迭代器只遍历单个桶内的元素。

**线程安全提示：**
遍历时若其他线程修改容器，迭代器可能失效，需要加锁。

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查是否为空 |
| `size()` | 返回元素个数 |
| `max_size()` | 返回最大可能元素个数 |

**时间复杂度：** O(1)

---

### 6. 修改器

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert(const value_type& value)` | 插入元素，返回 `std::pair<iterator, bool>` |
| `insert(value_type&& value)` | 插入右值（C++11） |
| `insert(iterator hint, const value_type& value)` | 带提示插入（提示不影响复杂度） |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素，返回 `std::pair<iterator, bool>`（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示原位构造（C++11） |
| `try_emplace(const Key& key, Args&&... args)` | 若键不存在则原位构造值（C++17） |
| `insert_or_assign(const Key& key, T&& obj)` | 若键存在则赋值，否则插入（C++17） |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除范围 |
| `erase(const key_type& key)` | 移除指定键的元素，返回删除个数 |
| `swap(unordered_map& other)` | 交换内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(unordered_map& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::unordered_map<int, std::string> m;
auto p = m.insert({1, "one"});      // p.second == true
m.insert({1, "uno"});               // 失败，键已存在
m.emplace(2, "two");
m.try_emplace(3, "three");
m.insert_or_assign(1, "ONE");       // 修改键 1 的值为 "ONE"
m.erase(2);                         // 删除键 2
```

**实现原理：**
- 插入：计算哈希值，定位桶，遍历链表查找键，若不存在则在链表头部或尾部插入。
- 删除：定位后移除节点，调整链表指针。

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
所有修改操作都需要同步。

---

### 7. 查找

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 返回指向键为 `key` 的元素的迭代器，若不存在则返回 `end()` |
| `count(const Key& key)` | 返回键为 `key` 的元素个数（对于 map 为 0 或 1） |
| `contains(const Key& key)` | 检查是否包含键为 `key` 的元素（C++20） |
| `equal_range(const Key& key)` | 返回 `std::pair<iterator, iterator>`，表示键等于 `key` 的范围 |

**示例：**
```cpp
auto it = m.find(1);
if (it != m.end()) std::cout << it->second;
bool has = m.contains(2);   // C++20
```

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
查找是只读的，多个线程可以并发查找（但不能有修改）。

---

### 8. 桶接口

| 函数 | 说明 |
|------|------|
| `size_type bucket_count() const` | 返回桶的数量 |
| `size_type max_bucket_count() const` | 返回最大桶数 |
| `size_type bucket_size(size_type n) const` | 返回第 `n` 个桶中的元素个数 |
| `size_type bucket(const Key& key) const` | 返回键 `key` 所属的桶索引 |
| `local_iterator begin(size_type n)` | 返回第 `n` 个桶的起始局部迭代器 |
| `local_iterator end(size_type n)` | 返回第 `n` 个桶的尾后局部迭代器 |

**示例：**
```cpp
size_t buckets = m.bucket_count();
for (size_t i = 0; i < buckets; ++i) {
    std::cout << "Bucket " << i << ": " << m.bucket_size(i) << " elements\n";
}
```

---

### 9. 哈希策略

| 函数 | 说明 |
|------|------|
| `float load_factor() const` | 返回当前负载因子（`size() / bucket_count()`） |
| `float max_load_factor() const` | 返回最大负载因子 |
| `void max_load_factor(float ml)` | 设置最大负载因子 |
| `void rehash(size_type n)` | 设置桶数至少为 `n`，并重新哈希 |
| `void reserve(size_type n)` | 预留空间，使 `bucket_count() >= n / max_load_factor()` |

**示例：**
```cpp
m.reserve(100);      // 提前预留空间，减少 rehash
m.rehash(200);       // 强制重新哈希
float lf = m.load_factor();
m.max_load_factor(0.8f);
```

**实现原理：**
- 当负载因子超过 `max_load_factor()` 时，容器自动进行 rehash（增加桶数，重新分配元素）。
- `rehash` 和 `reserve` 允许用户控制哈希表的大小。

---

### 10. 观察器

| 函数 | 说明 |
|------|------|
| `hasher hash_function() const` | 返回哈希函数对象 |
| `key_equal key_eq() const` | 返回键相等比较函数对象 |
| `allocator_type get_allocator() const` | 返回分配器副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=` | 比较两个 unordered_map 是否相等（不考虑顺序） |
| `swap(unordered_map& lhs, unordered_map& rhs)` | 交换内容 |
| `erase_if(unordered_map& c, Predicate pred)` | 移除所有满足谓词的元素（C++20） |

**注意：** 由于无序，比较操作需要逐元素比较，时间复杂度 O(n)。

---

## 五、宏与常量

`<unordered_map>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::unordered_map` 通常基于**开链法（separate chaining）**哈希表实现：
- 内部维护一个桶数组（bucket array），每个桶是一个链表（或类似结构）。
- 插入时，计算键的哈希值 `h = hash(key)`，桶索引 `idx = h % bucket_count()`，然后将元素插入到桶的链表中。
- 查找时，计算桶索引，然后遍历桶内链表，使用 `KeyEqual` 比较键。
- 当负载因子超过阈值时，进行 rehash：分配更大的桶数组，重新计算所有元素的桶索引并移动。

**节点结构（简化）：**
```cpp
struct Node {
    value_type value;
    Node* next;
};
```

**性能特点：**
- 平均 O(1) 得益于均匀哈希和合适的负载因子。
- 最坏 O(n) 当所有元素哈希到同一个桶（如哈希函数质量差或攻击）。
- 元素不排序，但插入顺序不影响查找效率。

**unordered_multimap 的区别：**
- 允许重复键，插入时总是成功，返回指向新元素的迭代器。
- `equal_range` 可以返回一个范围。
- 没有 `operator[]` 和 `at()`。

---

## 七、时间复杂度

| 操作 | 平均 | 最坏 |
|------|------|------|
| `operator[]`、`at` | O(1) | O(n) |
| `insert`、`emplace` | O(1) | O(n) |
| `erase`（键或迭代器） | O(1) | O(n) |
| `find`、`count`、`contains` | O(1) | O(n) |
| `rehash`、`reserve` | O(n) | O(n) |
| `bucket_count`、`size`、`empty` | O(1) | O(1) |

---

## 八、线程安全

**`std::unordered_map` 不是线程安全的**。规则与其它容器相同：
- **同时读取**：多个线程同时读取（`find`、`size` 等）是安全的。
- **读取 + 写入**：一个线程写入（`insert`、`erase`、`operator[]`）时，其他线程不能进行任何操作，否则数据竞争。
- **同时写入**：多个线程同时写入会导致未定义行为。
- **rehash**：写入操作可能触发 rehash，使所有迭代器失效，并发更加危险。

**保证线程安全的常用方法：**
- 使用互斥锁（`std::mutex`）保护所有访问。
- 使用并发哈希表（如 `tbb::concurrent_unordered_map`）。

---

## 九、各标准版本新增特性

### C++11
- 首次引入 `std::unordered_map` 和 `std::unordered_multimap`。

### C++14
- 无显著新特性。

### C++17
- `try_emplace`、`insert_or_assign` 成员函数。
- `extract` 和 `merge` 操作。
- `std::pmr::unordered_map` 多态分配器别名。

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
| `erase(iterator pos)` | 仅被删除元素的迭代器失效，其他迭代器有效 |
| `erase(const key_type& key)` | 仅被删除元素的迭代器失效 |
| `clear` | 所有迭代器失效 |
| `rehash` / `reserve` | 所有迭代器失效 |
| `swap` | 所有迭代器失效（指向其他容器） |
| `extract` | 被提取节点的迭代器失效，但节点指针有效 |
| `merge` | 被移动节点的迭代器在原容器中失效，在目标容器中重新有效 |

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::hash<Key>` | 默认哈希函数 |
| `std::equal_to<Key>` | 默认键相等比较器 |
| `std::allocator<std::pair<const Key, T>>` | 默认分配器 |
| `std::pmr::polymorphic_allocator` | 多态分配器（C++17） |

---

## 十二、补充：std::unordered_multimap 简要说明

`std::unordered_multimap` 与 `std::unordered_map` 的区别：
- 允许重复键。
- `insert` 总是成功，返回 `iterator`（而不是 `pair<iterator,bool>`）。
- 没有 `operator[]` 和 `at()`。
- `count`、`equal_range` 对于重复键有意义。
- `erase(const key_type& key)` 删除所有等于该键的元素，返回删除个数。

其他方面与 `std::unordered_map` 基本一致。