## `<list>` 头文件详解

`<list>` 是 C++ 标准模板库（STL）中定义的头文件，提供了 `std::list` 容器。`std::list` 是一个双向链表容器，支持在任意位置进行 O(1) 的插入和删除（已知迭代器位置），提供双向迭代器，支持正向和反向遍历。与 `std::forward_list` 相比，每个节点多存储一个指向前驱的指针，但功能更丰富（如 `size()`、`reverse_iterator` 等）。

---

## 一、模板参数

```cpp
template< class T, class Allocator = std::allocator<T> >
class list;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型。必须满足 `Erasable` 要求 |
| `Allocator` | 分配器类型，默认为 `std::allocator<T>`。用于管理 `list` 的内存分配和释放 |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class T >
    using list = std::list<T, std::pmr::polymorphic_allocator<T>>;
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
| `iterator` | 双向迭代器（实现定义） |
| `const_iterator` | 常量双向迭代器 |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `list()` | 默认构造一个空 `list` |
| `explicit list(const Allocator& alloc)` | 使用指定分配器构造空 `list` |
| `explicit list(size_type count)` | 构造包含 `count` 个默认插入元素的 `list` |
| `list(size_type count, const T& value)` | 构造包含 `count` 个 `value` 副本的 `list` |
| `template<class InputIt> list(InputIt first, InputIt last)` | 用迭代器范围 `[first, last)` 构造 `list` |
| `list(const list& other)` | 拷贝构造函数 |
| `list(list&& other)` | 移动构造函数（C++11） |
| `list(std::initializer_list<T> init)` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~list()` | 销毁 `list` 中的所有元素并释放内存 |

**示例：**
```cpp
std::list<int> l1;                     // 空 list
std::list<int> l2(5);                  // 5 个默认值（0）
std::list<int> l3(5, 42);              // 5 个 42
std::list<int> l4(l3);                 // 拷贝构造
std::list<int> l5 = {1,2,3,4,5};       // 初始化列表
std::list<int> l6(std::move(l5));      // 移动构造
```

**实现原理：**
- 默认构造时，通常创建一个空链表（头尾指针为 `nullptr`，或一个循环哨兵节点）。
- 拷贝构造时，遍历原链表并逐节点复制。
- 移动构造时，直接接管原链表的头尾指针和大小计数器，原链表变为空。

**线程安全提示：**
构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const list& other)` | 拷贝赋值 |
| `operator=(list&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<T> init)` | 初始化列表赋值（C++11） |
| `assign(size_type count, const T& value)` | 用 `count` 个 `value` 副本替换内容 |
| `template<class InputIt> assign(InputIt first, InputIt last)` | 用迭代器范围替换内容 |
| `assign(std::initializer_list<T> init)` | 用初始化列表替换内容（C++11） |

**示例：**
```cpp
std::list<int> l1 = {1,2,3};
std::list<int> l2;
l2 = l1;                           // 拷贝赋值
l2.assign(5, 42);                  // 变成 5 个 42
l2.assign({10, 20, 30});           // 变成 10, 20, 30
```

**实现原理：**
- 拷贝/移动赋值时，先清空自身，再复制或接管节点。

**线程安全提示：**
多线程中并发对同一个 `list` 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `front()` | 返回第一个元素的引用（链表不能为空，否则未定义行为） |
| `back()` | 返回最后一个元素的引用（链表不能为空，否则未定义行为） |

**示例：**
```cpp
std::list<int> l = {10, 20, 30};
int a = l.front();   // 10
int b = l.back();    // 30
```

**实现原理：**
- `front()` 返回头节点指向的元素的引用。
- `back()` 返回尾节点指向的元素的引用。

**时间复杂度：** O(1)

**线程安全提示：**
多个线程同时读取 `front()`/`back()` 是安全的；一个线程写入而另一个线程读取会导致数据竞争。

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
std::list<int> l = {1,2,3,4,5};
for (auto it = l.begin(); it != l.end(); ++it) {
    std::cout << *it << " ";
}
for (auto rit = l.rbegin(); rit != l.rend(); ++rit) {
    std::cout << *rit << " ";
}
```

**实现原理：**
- `list` 的迭代器封装了指向节点的指针，支持 `++`、`--` 操作。
- `begin()` 返回指向第一个有效节点的迭代器。
- `end()` 返回一个指向哨兵节点（或 `nullptr`）的迭代器。

**线程安全提示：**
遍历过程中，如果另一个线程修改链表，迭代器可能失效。多线程遍历时应加锁。

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查链表是否为空 |
| `size()` | 返回当前元素个数（C++11 前可能为 O(n)，C++11 起要求 O(1)） |
| `max_size()` | 返回理论最大元素个数 |

**示例：**
```cpp
std::list<int> l = {1,2,3};
bool e = l.empty();   // false
size_t s = l.size();  // 3
```

**实现原理：**
- C++11 起，`list` 必须维护一个大小计数器，使得 `size()` 为 O(1)。
- `empty()` 检查头指针或大小计数器。

**时间复杂度：** O(1)（C++11 起）

**线程安全提示：** 只读，安全。

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
| `resize(size_type count)` | 调整链表大小 |
| `resize(size_type count, const value_type& value)` | 调整链表大小，新元素填充 `value` |
| `swap(list& other)` | 交换两个链表的内容 |

**示例：**
```cpp
std::list<int> l = {2,3,4};
l.push_front(1);                   // 1,2,3,4
l.push_back(5);                    // 1,2,3,4,5
l.pop_front();                     // 2,3,4,5
l.pop_back();                      // 2,3,4
auto it = l.begin();
++it;
l.insert(it, 99);                  // 2,99,3,4
l.erase(it);                       // 2,3,4
l.emplace_front(0);                // 0,2,3,4
l.emplace_back(5);                 // 0,2,3,4,5
```

**实现原理：**
- `push_front` / `push_back`：分配新节点，调整头/尾指针。
- `insert`：在给定位置之前插入新节点（双向链表可以方便地获取前驱）。
- `erase`：删除节点，调整前后节点的指针。

**时间复杂度：**
- `push_front` / `pop_front` / `push_back` / `pop_back`：O(1)。
- `insert` / `erase`：O(1)（给定迭代器位置）。
- `resize`：O(n)。

**线程安全提示：**
多个线程并发修改需要加锁。

---

### 7. 操作（Operations）

`list` 提供了链表特有的操作，如拼接、合并、去重、排序等。

| 函数 | 说明 |
|------|------|
| `splice(const_iterator pos, list& other)` | 将 `other` 的所有节点移动到 `pos` 之前 |
| `splice(const_iterator pos, list&& other)` | 移动版本 |
| `splice(const_iterator pos, list& other, const_iterator it)` | 将 `it` 指向的单个节点移动到 `pos` 之前 |
| `splice(const_iterator pos, list& other, const_iterator first, const_iterator last)` | 将 `[first, last)` 范围内的节点移动 |
| `remove(const T& value)` | 移除所有等于 `value` 的元素 |
| `remove_if(UnaryPredicate p)` | 移除所有满足谓词 `p` 的元素 |
| `unique()` | 移除相邻重复元素（要求已排序） |
| `unique(BinaryPredicate p)` | 移除相邻满足 `p` 的元素 |
| `merge(list& other)` | 合并两个已排序的链表（归并） |
| `merge(list&& other)` | 移动版本 |
| `sort()` | 排序（默认升序） |
| `sort(Compare comp)` | 自定义比较器排序 |
| `reverse()` | 反转链表 |

**示例：**
```cpp
std::list<int> l1 = {1,3,5};
std::list<int> l2 = {2,4,6};
l1.merge(l2);                    // l1: 1,2,3,4,5,6, l2 为空
l1.reverse();                    // 6,5,4,3,2,1
l1.sort();                       // 1,2,3,4,5,6
l1.unique();                     // 移除相邻重复
l1.remove(3);                    // 移除所有 3
```

**实现原理：**
- `splice`：通过调整节点指针移动节点，不复制元素。
- `merge`：归并两个有序链表，O(n)。
- `sort`：通常实现为归并排序，O(n log n)。
- `reverse`：遍历链表，反转指针方向，O(n)。

**线程安全提示：**
所有操作都修改链表，需要同步。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 `list` |
| `swap(std::list& lhs, std::list& rhs)` | 交换两个链表的内容 |
| `erase(std::list& c, const U& value)` | 移除所有等于 `value` 的元素（C++20） |
| `erase_if(std::list& c, Predicate pred)` | 移除所有满足谓词 `pred` 的元素（C++20） |

---

## 五、宏与常量

`<list>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::list` 的典型实现是一个双向循环链表（或带哨兵节点的双向链表）。每个节点包含一个元素和两个指针（`prev` 和 `next`）。通常有一个哨兵节点（不存储有效数据），使得 `begin()` 和 `end()` 的表示更统一。

**节点结构（简化）：**
```cpp
struct Node {
    T value;
    Node* prev;
    Node* next;
};
```

**迭代器：** 双向迭代器，支持 `++` 和 `--`。迭代器内部封装一个指向 `Node` 的指针。

**大小计数器：** C++11 起，`list` 必须维护一个 `size_type` 成员，使得 `size()` 为 O(1)。

**优点：**
- 在已知位置插入和删除 O(1)。
- 支持双向遍历。
- 不要求元素可移动（元素在节点中稳定，移动元素只是调整指针）。

**缺点：**
- 每个节点多一个指针，内存开销较大。
- 缓存不友好，遍历性能较低。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| `push_front` / `pop_front` / `push_back` / `pop_back` | O(1) |
| `insert` / `erase`（已知位置） | O(1) |
| `insert` / `erase`（查找位置） | O(n) |
| 查找（按值） | O(n) |
| `size` / `empty` | O(1) |
| `splice`（移动单个节点） | O(1) |
| `splice`（移动范围） | O(1)（如果移动整个链表）或 O(n)（需计算节点数） |
| `merge` | O(n) |
| `sort` | O(n log n) |
| `reverse` | O(n) |
| `unique` / `remove` / `remove_if` | O(n) |

---

## 八、线程安全

**`std::list` 本身不是线程安全的**。规则与其它容器类似：
- **同时读取**：多个线程同时读取不同节点是安全的。
- **读取 + 写入**：一个线程修改链表时，其他线程不能进行任何操作，否则数据竞争。
- **同时写入**：多个线程同时修改链表会导致未定义行为。

**保证线程安全的常用方法：**
- 使用互斥锁保护整个 `list`。
- 使用线程局部存储。
- 使用第三方并发容器。

---

## 九、各标准版本新增特性

### C++11
- 移动构造函数和移动赋值运算符。
- `emplace`、`emplace_front`、`emplace_back` 原位构造。
- `cbegin()` / `cend()` / `crbegin()` / `crend()`。
- 初始化列表构造和赋值。
- 右值引用版本的 `push_back`、`push_front`、`insert`。
- `size()` 要求 O(1)（此前某些实现为 O(n)）。

### C++14
- 无明显针对 `list` 的新特性。

### C++17
- `std::pmr::list` 多态分配器别名模板。
- `insert` 等成员函数支持节点提取（`extract`）。

### C++20
- `erase()` 和 `erase_if()` 非成员函数。
- `constexpr` 修饰部分成员函数（有限支持）。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `insert` | 不影响已有迭代器 |
| `emplace` | 不影响已有迭代器 |
| `push_front` / `push_back` | 不影响已有迭代器 |
| `erase` | 指向被删除元素的迭代器失效 |
| `pop_front` / `pop_back` | 指向被删除元素的迭代器失效 |
| `clear` | 所有迭代器失效 |
| `splice` | 被移动的节点在原链表中的迭代器失效；在目标链表中重新有效 |
| `sort` / `reverse` | 所有迭代器失效（节点重新链接） |
| `swap` | 所有迭代器失效（指向其他容器） |
| `resize`（缩小） | 被删除元素的迭代器失效 |

**注意：** `list` 的插入操作通常不会使任何已有迭代器失效（除了被删除的节点），这是链表的优势。

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::allocator<T>` | 默认分配器 |
| `std::pmr::polymorphic_allocator<T>` | 多态分配器（C++17） |

---