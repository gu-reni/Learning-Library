## `<set>` 头文件详解

`<set>` 是 C++ 标准模板库（STL）中定义的头文件，提供了两个有序关联容器：`std::set` 和 `std::multiset`。`std::set` 是一个有序集合，元素唯一，按排序规则（默认为 `std::less<Key>`）自动排序；`std::multiset` 与 `std::set` 类似，但允许元素重复。两者均基于红黑树实现，插入、删除和查找操作的时间复杂度为 O(log n)。

---

## 一、模板参数

### 1. std::set
```cpp
template< class Key, class Compare = std::less<Key>,
          class Allocator = std::allocator<Key> >
class set;
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
    using set = std::set<Key, Compare, std::pmr::polymorphic_allocator<Key>>;
}
```

### 2. std::multiset
```cpp
template< class Key, class Compare = std::less<Key>,
          class Allocator = std::allocator<Key> >
class multiset;
```
模板参数与 `std::set` 相同，区别在于允许元素重复。

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
| `iterator` | 双向迭代器，指向 `const value_type`（**注意：`set` 的迭代器是常量迭代器**，因为修改元素会破坏排序） |
| `const_iterator` | 常量双向迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

**注意：** `set` 的 `iterator` 实际上指向常量元素，因此不能通过迭代器修改元素值。

---

## 三、成员函数详解

以下以 `std::set` 为例，`std::multiset` 的接口类似，但部分行为不同（如 `insert` 返回类型、`count` 等）。

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `set()` | 默认构造一个空 set |
| `explicit set(const Compare& comp, const Allocator& alloc = Allocator())` | 使用给定的比较函数对象和分配器构造空 set |
| `template<class InputIt> set(InputIt first, InputIt last, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 用迭代器范围 `[first, last)` 构造 set |
| `set(const set& other)` | 拷贝构造函数 |
| `set(set&& other)` | 移动构造函数（C++11） |
| `set(std::initializer_list<value_type> init, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~set()` | 销毁 set 中的所有元素并释放内存 |

**示例：**
```cpp
std::set<int> s1;                               // 空 set
std::set<int> s2 = {1, 2, 3, 2, 1};             // 实际存储 {1, 2, 3}
std::set<int> s3(s2);                           // 拷贝构造
std::set<int> s4(std::move(s3));                // 移动构造
```

**实现原理：**
- 默认构造时，创建一个空的红黑树（通常只有一个哨兵节点）。
- 拷贝构造时，遍历原 set 的节点并逐个复制到新树中。
- 移动构造时，直接接管原树的根节点和大小计数器，原 set 变为空。

**线程安全提示：**
构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const set& other)` | 拷贝赋值 |
| `operator=(set&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |

**示例：**
```cpp
std::set<int> s1 = {1, 2};
std::set<int> s2;
s2 = s1;                           // 拷贝赋值
s2 = {3, 4, 5};                    // 初始化列表赋值
```

**实现原理：**
- 拷贝赋值时，先清空自身，再复制元素。
- 移动赋值时，直接接管源 set 的资源。

**线程安全提示：**
多线程中并发对同一个 set 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（按排序规则的最小元素） |
| `end()` / `cend()` | 返回指向尾后位置的迭代器 |
| `rbegin()` / `crbegin()` | 返回指向最后一个元素的反向迭代器 |
| `rend()` / `crend()` | 返回指向第一个元素前一个位置的反向迭代器 |

**示例：**
```cpp
std::set<int> s = {3, 1, 4, 1, 5};
for (auto it = s.begin(); it != s.end(); ++it) {
    std::cout << *it << " ";   // 输出: 1 3 4 5
}
for (auto rit = s.rbegin(); rit != s.rend(); ++rit) {
    std::cout << *rit << " ";   // 输出: 5 4 3 1
}
```

**实现原理：**
- 迭代器按照红黑树的中序遍历顺序访问元素（即按排序规则升序）。
- `begin()` 返回最左节点的迭代器，`end()` 返回哨兵节点。

**线程安全提示：**
遍历过程中，如果另一个线程修改 set，迭代器可能失效。多线程遍历时应加锁。

---

### 4. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查 set 是否为空 |
| `size()` | 返回元素个数 |
| `max_size()` | 返回理论最大元素个数 |

**示例：**
```cpp
std::set<int> s = {1, 2, 3};
bool e = s.empty();   // false
size_t sz = s.size(); // 3
```

**时间复杂度：** O(1)

**线程安全提示：** 只读操作，线程安全。

---

### 5. 修改器

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert(const value_type& value)` | 插入元素，返回 `std::pair<iterator, bool>`（bool 表示是否插入成功） |
| `insert(value_type&& value)` | 插入右值元素（C++11） |
| `insert(iterator hint, const value_type& value)` | 带提示的插入（可优化） |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入迭代器范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示的原位构造（C++11） |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除迭代器范围内的元素 |
| `erase(const key_type& key)` | 移除指定键的元素（返回移除个数，对于 set 为 0 或 1） |
| `swap(set& other)` | 交换两个 set 的内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(set& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::set<int> s;
auto [it, inserted] = s.insert(42);   // inserted == true
s.insert(42);                         // 插入失败，元素已存在
s.emplace(100);
s.erase(42);                          // 移除键 42
auto it2 = s.find(100);
s.erase(it2);                         // 移除迭代器指向的元素
```

**实现原理：**
- `insert` 和 `emplace` 基于红黑树插入算法，维持排序和平衡。
- 由于键唯一，插入时会先查找，若存在则不插入。
- `erase` 基于红黑树删除算法，可能触发旋转和颜色调整。

**时间复杂度：**
- `insert` / `emplace` / `erase`（键）：O(log n)
- `erase`（迭代器）：均摊 O(1)
- `swap`：O(1)

**线程安全提示：**
所有修改操作都需要同步。

---

### 6. 查找

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 查找键为 `key` 的元素，返回迭代器；若不存在则返回 `end()` |
| `count(const Key& key)` | 返回键为 `key` 的元素个数（对于 set 为 0 或 1） |
| `contains(const Key& key)` | 检查是否包含键为 `key` 的元素（C++20） |
| `lower_bound(const Key& key)` | 返回第一个键**不小于** `key` 的迭代器 |
| `upper_bound(const Key& key)` | 返回第一个键**大于** `key` 的迭代器 |
| `equal_range(const Key& key)` | 返回 `std::pair<iterator, iterator>`，表示键等于 `key` 的范围 |

**示例：**
```cpp
std::set<int> s = {1, 2, 3, 4, 5};
auto it = s.find(3);
if (it != s.end()) std::cout << *it;   // 3
bool has = s.contains(3);              // true (C++20)
auto lb = s.lower_bound(3);            // 指向 3
auto ub = s.upper_bound(3);            // 指向 4
auto range = s.equal_range(3);         // [lb, ub)
```

**实现原理：**
- 基于红黑树的查找算法，时间复杂度 O(log n)。
- `lower_bound` / `upper_bound` 利用树的有序性。

**线程安全提示：**
查找操作是只读的，多个线程可以并发查找（但不能有修改）。

---

### 7. 观察器

| 函数 | 说明 |
|------|------|
| `key_comp()` | 返回键比较函数对象的副本 |
| `value_comp()` | 返回值比较函数对象的副本（同 `key_comp`） |

**示例：**
```cpp
std::set<int> s;
auto cmp = s.key_comp();
bool b = cmp(1, 2);   // true
```

**时间复杂度：** O(1)

**线程安全提示：** 只读，安全。

---

### 8. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器的副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 set 的内容 |
| `swap(std::set& lhs, std::set& rhs)` | 交换两个 set 的内容 |
| `erase_if(std::set& c, Predicate pred)` | 移除所有满足谓词 `pred` 的元素（C++20） |

---

## 五、宏与常量

`<set>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::set` 和 `std::multiset` 通常基于**红黑树**实现（一种自平衡二叉搜索树）。红黑树具有以下性质：
- 每个节点是红色或黑色。
- 根节点是黑色。
- 红色节点的子节点必须是黑色。
- 从任一节点到其每个叶子节点的所有路径都包含相同数量的黑色节点。
- 新插入的节点初始为红色，通过旋转和重新着色维持平衡。

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

**主要操作：**
- **插入**：找到插入位置，插入红色节点，然后进行修复（旋转和着色）以维持红黑树性质。
- **删除**：删除节点后，通过旋转和着色修复平衡。
- **查找**：利用二叉搜索树性质，从根向下比较键，时间复杂度 O(log n)。

**multiset 的区别：**
- 允许重复键，插入时在相等键的右侧插入。
- `insert` 返回 `iterator`（而不是 `pair<iterator,bool>`）。
- `count` 可以返回大于 1 的值。
- 没有 `operator[]`（因为键即值，且允许重复）。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 查找（`find`、`lower_bound`、`upper_bound`、`equal_range`） | O(log n) |
| 插入（`insert`、`emplace`） | O(log n) |
| 带提示的插入（提示正确） | 均摊 O(1) |
| 删除（键） | O(log n) |
| 删除（迭代器） | 均摊 O(1) |
| `size`、`empty` | O(1) |
| `swap` | O(1) |

---

## 八、线程安全

**`std::set` 本身不是线程安全的**。规则：
- **同时读取**：多个线程同时读取 set 是安全的（只读操作）。
- **读取 + 写入**：一个线程写入（插入、删除）时，其他线程不能进行任何操作（包括读取），否则数据竞争。
- **同时写入**：多个线程同时写入 set 会导致未定义行为。

**保证线程安全的常用方法：**
- 使用 `std::mutex` 或 `std::shared_mutex` 保护整个 set。
- 使用线程局部存储。
- 使用并发容器（如 `tbb::concurrent_unordered_set`）。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace`、`emplace_hint` 原位构造。
- 初始化列表构造和赋值。
- `cbegin()` / `cend()` / `crbegin()` / `crend()`。
- 右值引用版本的 `insert`。

### C++14
- 无显著新特性。

### C++17
- `extract` 节点提取操作。
- `merge` 合并操作。
- `std::pmr::set` 多态分配器别名。
- 带提示的 `insert` 返回迭代器和 bool 的组合（已存在）。

### C++20
- `contains` 成员函数。
- `operator<=>` 三路比较支持。
- `erase_if` 非成员函数。
- `constexpr` 修饰部分成员函数（有限支持）。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `insert` | 不影响已有迭代器（不使任何现有迭代器失效） |
| `emplace` | 不影响已有迭代器 |
| `erase(iterator pos)` | 仅被删除的迭代器失效 |
| `erase(const key_type& key)` | 仅被删除的键对应的迭代器失效 |
| `clear` | 所有迭代器失效 |
| `extract` | 被提取节点的迭代器失效，但节点指针仍有效 |
| `merge` | 被移动节点的迭代器在原 set 中失效，在目标 set 中重新有效 |
| `swap` | 所有迭代器失效（指向其他 set 的元素） |

**注意：** set 的插入操作不会使任何已有迭代器失效（除了被删除的节点），这是关联容器的优势。

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::less<Key>` | 默认比较器 |
| `std::allocator<Key>` | 默认分配器 |
| `std::pmr::polymorphic_allocator<Key>` | 多态分配器（C++17） |

---

## 十二、补充：std::multiset 简要说明

`std::multiset` 与 `std::set` 的区别：
- 允许多个元素具有相同的键。
- `insert` 总是成功，返回指向新元素的迭代器（而不是 `std::pair<iterator,bool>`）。
- `count`、`lower_bound`、`upper_bound`、`equal_range` 对于重复键有意义。
- 使用 `erase(const key_type& key)` 会删除所有等于该键的元素，返回删除个数。
- 没有 `operator[]`（因为键即值，且不唯一）。

其他方面与 `std::set` 基本一致。