## `<map>` 头文件详解

`<map>` 是 C++ 标准模板库（STL）中定义的头文件，提供了 `std::map` 和 `std::multimap` 两个关联容器。`std::map` 是一个有序的键值对容器，键唯一，元素按照键的排序规则（默认为 `std::less<Key>`）自动排序。`std::multimap` 与 `std::map` 类似，但允许键重复。两者均基于红黑树实现，插入、删除和查找操作的时间复杂度为 O(log n)。

---

## 一、模板参数

### 1. std::map
```cpp
template< class Key, class T, class Compare = std::less<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class map;
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
    using map = std::map<Key, T, Compare, std::pmr::polymorphic_allocator<std::pair<const Key, T>>>;
}
```

### 2. std::multimap
```cpp
template< class Key, class T, class Compare = std::less<Key>,
          class Allocator = std::allocator<std::pair<const Key, T>> >
class multimap;
```
模板参数与 `std::map` 相同，区别在于允许键重复。

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `key_type` | `Key` |
| `mapped_type` | `T` |
| `value_type` | `std::pair<const Key, T>` |
| `key_compare` | `Compare`，用于比较键的函数对象类型 |
| `value_compare` | 嵌套函数对象类型，用于比较 `value_type`（通过键比较） |
| `allocator_type` | `Allocator` |
| `size_type` | 无符号整数类型（通常为 `size_t`） |
| `difference_type` | 有符号整数类型（通常为 `ptrdiff_t`） |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*`（或 `Allocator::pointer`） |
| `const_pointer` | `const T*` |
| `iterator` | 双向迭代器，指向 `value_type` |
| `const_iterator` | 常量双向迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

**注意：** `value_type` 中的 `Key` 是 const 限定的，因此不能修改键，但可以修改值。

---

## 三、成员函数详解

以下以 `std::map` 为例，`std::multimap` 的接口类似，但部分行为不同（如 `insert`、`operator[]` 等）。

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `map()` | 默认构造一个空 map |
| `explicit map(const Compare& comp, const Allocator& alloc = Allocator())` | 使用给定的比较函数对象和分配器构造空 map |
| `template<class InputIt> map(InputIt first, InputIt last, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 用迭代器范围 `[first, last)` 构造 map |
| `map(const map& other)` | 拷贝构造函数 |
| `map(map&& other)` | 移动构造函数（C++11） |
| `map(std::initializer_list<value_type> init, const Compare& comp = Compare(), const Allocator& alloc = Allocator())` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~map()` | 销毁 map 中的所有元素并释放内存 |

**示例：**
```cpp
std::map<int, std::string> m1;                               // 空 map
std::map<int, std::string> m2 = {{1,"one"}, {2,"two"}};      // 初始化列表
std::map<int, std::string> m3(m2);                           // 拷贝构造
std::map<int, std::string> m4(std::move(m3));                // 移动构造
```

**实现原理：**
- 默认构造时，创建一个空的红黑树（通常只有一个哨兵节点）。
- 拷贝构造时，遍历原 map 的节点并逐个复制到新树中。
- 移动构造时，直接接管原树的根节点和大小计数器，原 map 变为空。

**线程安全提示：**
构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const map& other)` | 拷贝赋值 |
| `operator=(map&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<value_type> init)` | 初始化列表赋值（C++11） |
| `assign(InputIt first, InputIt last)` | 用迭代器范围替换内容（C++23） |

**示例：**
```cpp
std::map<int, std::string> m1 = {{1,"one"}};
std::map<int, std::string> m2;
m2 = m1;                           // 拷贝赋值
m2 = {{2,"two"}, {3,"three"}};     // 初始化列表赋值
```

**实现原理：**
- 拷贝赋值时，先清空自身，再复制元素。
- 移动赋值时，直接接管源 map 的资源。

**线程安全提示：**
多线程中并发对同一个 map 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `at(const Key& key)` | 返回键 `key` 对应的值的引用，若键不存在则抛出 `std::out_of_range` |
| `at(const Key& key) const` | 常量版本 |
| `operator[](const Key& key)` | 返回键 `key` 对应的值的引用；若键不存在，则插入一个默认值（值初始化）并返回其引用 |
| `operator[](Key&& key)` | 右值引用版本（C++11） |

**示例：**
```cpp
std::map<int, std::string> m = {{1,"one"}};
std::string& s = m[1];          // s == "one"
m[2] = "two";                   // 插入键 2，值设为 "two"
std::string t = m.at(3);        // 抛出 std::out_of_range
```

**实现原理：**
- `operator[]`：查找键，如果存在则返回对应值的引用；否则插入一个默认构造的值，然后返回引用。
- `at()`：查找键，如果存在则返回引用；否则抛出异常。

**时间复杂度：** O(log n)

**线程安全提示：**
`operator[]` 可能会修改 map（插入新元素），并发使用需要同步。

---

### 4. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器（按键排序的最小元素） |
| `end()` / `cend()` | 返回指向尾后位置的迭代器 |
| `rbegin()` / `crbegin()` | 返回指向最后一个元素的反向迭代器 |
| `rend()` / `crend()` | 返回指向第一个元素前一个位置的反向迭代器 |

**示例：**
```cpp
std::map<int, std::string> m = {{2,"two"}, {1,"one"}, {3,"three"}};
for (auto it = m.begin(); it != m.end(); ++it) {
    std::cout << it->first << ":" << it->second << " ";
}
// 输出: 1:one 2:two 3:three （按键排序）
```

**实现原理：**
- 迭代器按照红黑树的中序遍历顺序访问元素（即按键升序）。
- `begin()` 返回最左节点的迭代器，`end()` 返回哨兵节点。

**线程安全提示：**
遍历过程中，如果另一个线程修改 map，迭代器可能失效。多线程遍历时应加锁。

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查 map 是否为空 |
| `size()` | 返回元素个数 |
| `max_size()` | 返回理论最大元素个数 |

**示例：**
```cpp
std::map<int, std::string> m = {{1,"one"}};
bool e = m.empty();   // false
size_t s = m.size();  // 1
```

**时间复杂度：** O(1)

**线程安全提示：** 只读操作，线程安全。

---

### 6. 修改器

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert(const value_type& value)` | 插入元素 |
| `insert(value_type&& value)` | 插入右值元素（C++11） |
| `insert(iterator hint, const value_type& value)` | 带提示的插入（可优化） |
| `template<class InputIt> insert(InputIt first, InputIt last)` | 插入迭代器范围 |
| `insert(std::initializer_list<value_type> init)` | 插入初始化列表（C++11） |
| `emplace(Args&&... args)` | 原位构造元素（C++11） |
| `emplace_hint(const_iterator hint, Args&&... args)` | 带提示的原位构造（C++11） |
| `erase(iterator pos)` | 移除指定位置的元素 |
| `erase(iterator first, iterator last)` | 移除迭代器范围内的元素 |
| `erase(const key_type& key)` | 移除指定键的元素（返回移除个数） |
| `swap(map& other)` | 交换两个 map 的内容 |
| `extract(const key_type& key)` | 提取节点（C++17） |
| `extract(iterator pos)` | 提取节点（C++17） |
| `merge(map& source)` | 合并节点（C++17） |

**示例：**
```cpp
std::map<int, std::string> m;
m.insert({1, "one"});
m.insert(std::make_pair(2, "two"));
m.emplace(3, "three");
m.erase(2);                       // 移除键 2
auto it = m.find(3);
m.erase(it);                      // 移除迭代器指向的元素
```

**实现原理：**
- `insert` 和 `emplace` 基于红黑树插入算法，维持排序和平衡。
- `erase` 基于红黑树删除算法，可能触发旋转和颜色调整。
- `emplace` 直接使用参数构造 `value_type`，避免拷贝或移动。

**时间复杂度：**
- `insert` / `emplace` / `erase`（键）：O(log n)
- `erase`（迭代器）：均摊 O(1)
- `swap`：O(1)

**线程安全提示：**
所有修改操作都需要同步。

---

### 7. 查找

| 函数 | 说明 |
|------|------|
| `find(const Key& key)` | 查找键为 `key` 的元素，返回迭代器；若不存在则返回 `end()` |
| `count(const Key& key)` | 返回键为 `key` 的元素个数（对于 map 为 0 或 1） |
| `contains(const Key& key)` | 检查是否包含键为 `key` 的元素（C++20） |
| `lower_bound(const Key& key)` | 返回第一个键不小于 `key` 的迭代器 |
| `upper_bound(const Key& key)` | 返回第一个键大于 `key` 的迭代器 |
| `equal_range(const Key& key)` | 返回 `std::pair<iterator, iterator>`，表示键等于 `key` 的范围 |

**示例：**
```cpp
std::map<int, std::string> m = {{1,"one"}, {2,"two"}, {3,"three"}};
auto it = m.find(2);
if (it != m.end()) std::cout << it->second;   // "two"
bool has = m.contains(3);                     // true (C++20)
auto lb = m.lower_bound(2);                   // 指向键 2
auto ub = m.upper_bound(2);                   // 指向键 3
auto range = m.equal_range(2);                // [lb, ub)
```

**实现原理：**
- 基于红黑树的查找算法，时间复杂度 O(log n)。
- `lower_bound` / `upper_bound` 利用树的有序性。

**线程安全提示：**
查找操作是只读的，多个线程可以并发查找（但不能有修改）。

---

### 8. 观察器

| 函数 | 说明 |
|------|------|
| `key_comp()` | 返回键比较函数对象的副本 |
| `value_comp()` | 返回值比较函数对象的副本（通过键比较） |

**示例：**
```cpp
std::map<int, std::string> m;
auto cmp = m.key_comp();
bool b = cmp(1, 2);   // true
```

**实现原理：**
- 返回存储的比较器对象。

**时间复杂度：** O(1)

**线程安全提示：** 只读，安全。

---

### 9. 分配器

| 函数 | 说明 |
|------|------|
| `get_allocator()` | 返回分配器的副本 |

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 map 的内容 |
| `swap(std::map& lhs, std::map& rhs)` | 交换两个 map 的内容 |

---

## 五、宏与常量

`<map>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::map` 和 `std::multimap` 通常基于**红黑树**实现（一种自平衡二叉搜索树）。红黑树具有以下性质：
- 每个节点是红色或黑色。
- 根节点是黑色。
- 红色节点的子节点必须是黑色。
- 从任一节点到其每个叶子节点的所有路径都包含相同数量的黑色节点。
- 新插入的节点初始为红色，通过旋转和重新着色维持平衡。

**节点结构（简化）：**
```cpp
struct Node {
    std::pair<const Key, T> value;  // 键值对，Key 不可修改
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

**multimap 的区别：**
- 允许键重复，插入时在相等键的右侧插入。
- 没有 `operator[]` 和 `at()`（因为键不唯一）。

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
| 元素访问（`operator[]`、`at`） | O(log n) |

---

## 八、线程安全

**`std::map` 本身不是线程安全的**。规则：
- **同时读取**：多个线程同时读取 map 是安全的（只读操作）。
- **读取 + 写入**：一个线程写入（插入、删除、修改）时，其他线程不能进行任何操作（包括读取），否则数据竞争。
- **同时写入**：多个线程同时写入 map 会导致未定义行为。

**保证线程安全的常用方法：**
- 使用 `std::mutex` 或 `std::shared_mutex` 保护整个 map。
- 使用线程局部存储。
- 使用并发容器（如 `tbb::concurrent_hash_map`）。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace`、`emplace_hint` 原位构造。
- 初始化列表构造和赋值。
- `cbegin()` / `cend()` / `crbegin()` / `crend()`。
- 右值引用版本的 `insert`。

### C++14
- 无显著针对 map 的新特性。

### C++17
- `extract` 节点提取操作。
- `merge` 合并操作。
- `std::pmr::map` 多态分配器别名。
- 带提示的 `insert` 返回迭代器和 bool 的组合（已存在）。

### C++20
- `contains` 成员函数。
- `operator<=>` 三路比较支持。
- `erase_if` 非成员函数。
- `constexpr` 修饰部分成员函数（有限支持）。

### C++23
- `assign` 成员函数（范围赋值）。

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
| `merge` | 被移动节点的迭代器在原 map 中失效，在目标 map 中重新有效 |
| `swap` | 所有迭代器失效（指向其他 map 的元素） |
| `operator[]` / `at` | 不影响迭代器 |

**注意：** map 的插入操作不会使任何已有迭代器失效（除了被删除的节点），这是关联容器的优势。

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::pair<const Key, T>` | 元素类型 |
| `std::less<Key>` | 默认比较器 |
| `std::allocator<std::pair<const Key, T>>` | 默认分配器 |
| `std::pmr::polymorphic_allocator` | 多态分配器（C++17） |
| `std::map::value_compare` | 嵌套比较器类型 |

---

## 十二、补充：std::multimap 简要说明

`std::multimap` 与 `std::map` 的区别：
- 允许多个元素具有相同的键。
- 没有 `operator[]` 和 `at()` 成员函数（因为键不唯一）。
- `insert` 总是成功，返回指向新元素的迭代器（而不是 `std::pair<iterator,bool>`）。
- `count`、`lower_bound`、`upper_bound`、`equal_range` 对于重复键有意义。
- 使用 `erase(const key_type& key)` 会删除所有等于该键的元素，返回删除个数。

其他方面与 `std::map` 基本一致。