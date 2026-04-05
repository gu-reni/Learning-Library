## `<unordered_map>` 头文件（`std::unordered_multimap`）详解

`<unordered_map>` 是 C++11 标准引入的头文件，提供了两个无序关联容器：`std::unordered_map` 和 `std::unordered_multimap`。`std::unordered_multimap` 是一个基于哈希表的关联容器，存储键值对（`std::pair<const Key, T>`），**允许键重复**。它不保持元素的特定顺序，插入、删除和查找操作的平均时间复杂度为 O(1)，最坏情况 O(n)。

---

## 一、模板参数

```cpp
template< class Key,
          class T,
          class Hash = std::hash<Key>,
          class KeyEqual = std::equal_to<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class unordered_multimap;
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
    using unordered_multimap = std::unordered_multimap<Key, T, Hash, KeyEqual, std::pmr::polymorphic_allocator<std::pair<const Key, T>>>;
}
```

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

**注意：** `std::unordered_multimap` 没有 `operator[]` 和 `at()` 成员函数，因为键不唯一。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `unordered_multimap()` | 默认构造空容器，桶数量由实现定义 |
| `explicit unordered_multimap(size_type bucket_count, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 指定初始桶数、哈希函数、相等比较器和分配器 |
| `template<class InputIt> unordered_multimap(InputIt first, InputIt last, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 范围构造 |
| `unordered_multimap(const unordered_multimap& other)` | 拷贝构造 |
| `unordered_multimap(unordered_multimap&& other)` | 移动构造（C++11） |
| `unordered_multimap(std::initializer_list<value_type> init, size_type bucket_count = /* 实现定义 */, const Hash& hash = Hash(), const KeyEqual& equal = KeyEqual(), const Allocator& alloc = Allocator())` | 初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~unordered_multimap()` | 销毁所有元素并释放内存 |

**示例：**
```cpp
std::unordered_multimap<int, std::string> mm1;                                    // 空
std::unordered_multimap<int, std::string> mm2(100);                              // 初始桶数 100
std::unordered_multimap<int, std::string> mm3 = {{1,"one"}, {1,"uno"}, {2,"two"}}; // 允许重复键
std::unordered_multimap<int, std::string> mm4(mm3);                              // 拷贝构造
```

**实现原理：**
- 基于哈希表（开链法），桶数组管理链表。
- 默认构造时桶数组通常为空或很小。
- 拷贝构造时复制所有元素到新的桶数组。

**线程安全提示：**
构造和析构应在单线程环境进行，或有锁保护。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const unordered_multimap& other)` | 拷贝赋值 |
| `operator=(unordered_multimap&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

**示例：**
```cpp
std::unordered_multimap<int, std::string> mm1, mm2;
mm1 = mm2;                       // 拷贝赋值
mm1 = {{1,"one"}, {1,"uno"}};    // 列表赋值
```

---

### 3. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（顺序未指定） |
| `end()` / `cend()` | 返回尾后迭代器 |
| `local_iterator begin(size_type n)` | 返回第 `n` 个桶内第一个元素的局部迭代器 |
| `local_iterator end(size_type n)` | 返回第 `n` 个桶的尾后局部迭代器 |

**示例：**
```cpp
std::unordered_multimap<int, std::string> mm = {{1,"one"}, {1,"uno"}, {2,"two"}};
for (auto& p : mm) {
    std::cout << p.first << ":" << p.second << "\n";
}
// 遍历桶
for (size_t i = 0; i < mm.bucket_count(); ++i) {
    for (auto it = mm.begin(i); it != mm.end(i); ++it) {
        // 处理桶内元素
    }
}
```

**线程安全提示：**
遍历时若其他线程修改容器，迭代器可能失效，需要加锁。

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
| `insert(iterator hint, const value_type& value)` | 带提示插入（提示不影响复杂度） |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素，**返回指向新元素的迭代器**（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示原位构造（C++11） |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除范围 |
| `erase(const key_type& key)` | 移除所有键等于 `key` 的元素，**返回被删除的元素个数** |
| `swap(unordered_multimap& other)` | 交换内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(unordered_multimap& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::unordered_multimap<int, std::string> mm;
auto it = mm.insert({1, "one"});   // 返回 iterator
mm.insert({1, "uno"});             // 插入成功，键重复
mm.emplace(2, "two");
size_t n = mm.erase(1);            // 删除所有键为 1 的元素，n == 2
mm.clear();
```

**实现原理：**
- 插入时计算哈希值，定位桶，将新节点插入到桶的链表中（通常在头部或尾部）。
- 由于允许重复键，插入总是成功，不需要检查键是否存在。
- 删除键时遍历桶内链表，删除所有匹配的节点。

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
所有修改操作都需要同步。

---

### 6. 查找

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 返回指向第一个键等于 `key` 的元素的迭代器，若不存在则返回 `end()` |
| `count(const Key& key)` | 返回键等于 `key` 的元素个数 |
| `contains(const Key& key)` | 检查是否存在键等于 `key` 的元素（C++20） |
| `equal_range(const Key& key)` | 返回 `std::pair<iterator, iterator>`，表示键等于 `key` 的范围 |

**示例：**
```cpp
std::unordered_multimap<int, std::string> mm = {{1,"one"}, {1,"uno"}, {2,"two"}};
auto it = mm.find(1);
if (it != mm.end()) std::cout << it->second;   // 输出 "one" 或 "uno"（不确定）
size_t c = mm.count(1);                        // 2
auto range = mm.equal_range(1);
for (auto p = range.first; p != range.second; ++p) {
    std::cout << p->second << " ";              // "one uno"
}
```

**时间复杂度：** 平均 O(1)，最坏 O(n)

**线程安全提示：**
查找是只读的，多个线程可以并发查找（但不能有修改）。

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

**示例：**
```cpp
size_t buckets = mm.bucket_count();
for (size_t i = 0; i < buckets; ++i) {
    std::cout << "Bucket " << i << ": " << mm.bucket_size(i) << " elements\n";
}
```

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
mm.reserve(100);      // 提前预留空间，减少 rehash
mm.rehash(200);       // 强制重新哈希
float lf = mm.load_factor();
mm.max_load_factor(0.8f);
```

**实现原理：**
- 当负载因子超过 `max_load_factor()` 时，容器自动进行 rehash（增加桶数，重新分配元素）。
- `rehash` 和 `reserve` 允许用户控制哈希表的大小。

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
| `operator==`、`operator!=` | 比较两个 unordered_multimap 是否相等（不考虑顺序） |
| `swap(unordered_multimap& lhs, unordered_multimap& rhs)` | 交换内容 |
| `erase_if(unordered_multimap& c, Predicate pred)` | 移除所有满足谓词的元素（C++20） |

**注意：** 比较操作需要逐元素比较，时间复杂度 O(n)。

---

## 五、宏与常量

`<unordered_map>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::unordered_multimap` 通常基于**开链法（separate chaining）**哈希表实现：
- 内部维护一个桶数组（bucket array），每个桶是一个链表（或类似结构）。
- 插入时，计算键的哈希值 `h = hash(key)`，桶索引 `idx = h % bucket_count()`，然后将元素插入到桶的链表中（通常在头部或尾部）。由于允许重复键，不检查键是否存在。
- 查找时，计算桶索引，然后遍历桶内链表，使用 `KeyEqual` 比较键。
- 当负载因子超过阈值时，进行 rehash：分配更大的桶数组，重新计算所有元素的桶索引并移动。

**与 `unordered_map` 的区别：**
- `unordered_multimap` 的 `insert` 总是成功，返回 `iterator`。
- 没有 `operator[]` 和 `at()`。
- `equal_range` 可以返回一个范围。
- `count` 可以返回大于 1 的值。
- `extract` 和 `merge` 行为类似。

**节点结构（简化）：**
```cpp
struct Node {
    value_type value;
    Node* next;
};
```

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

**`std::unordered_multimap` 不是线程安全的**。规则与其它容器相同：
- **同时读取**：多个线程同时读取（`find`、`size` 等）是安全的。
- **读取 + 写入**：一个线程写入（`insert`、`erase`）时，其他线程不能进行任何操作，否则数据竞争。
- **同时写入**：多个线程同时写入会导致未定义行为。
- **rehash**：写入操作可能触发 rehash，使所有迭代器失效，并发更加危险。

**保证线程安全的常用方法：**
- 使用互斥锁（`std::mutex`）保护所有访问。
- 使用并发哈希表（如 `tbb::concurrent_unordered_map`，但注意它通常不支持多键）。

---

## 九、各标准版本新增特性

### C++11
- 首次引入 `std::unordered_multimap`。

### C++14
- 无显著新特性。

### C++17
- `extract` 和 `merge` 操作。
- `std::pmr::unordered_multimap` 多态分配器别名。

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
| `erase(const key_type& key)` | 所有指向被删除元素的迭代器失效 |
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