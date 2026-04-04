## `<forward_list>` 头文件详解

`<forward_list>` 是 C++ 标准模板库（STL）中定义的头文件（C++11 起），提供了 `std::forward_list` 容器。`std::forward_list` 是一个单向链表容器，支持在任意位置进行 O(1) 的插入和删除（已知前驱迭代器），但只提供前向迭代器，不支持反向迭代和随机访问。它的设计目标是比 `std::list` 更轻量，内存开销更小（每个节点只存储一个指针，而不是两个）。

---

## 一、模板参数

```cpp
template< class T, class Allocator = std::allocator<T> >
class forward_list;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型。必须满足 `Erasable` 要求 |
| `Allocator` | 分配器类型，默认为 `std::allocator<T>`。用于管理 `forward_list` 的内存分配和释放 |

**多态分配器别名**（C++17 起）：
```cpp
namespace pmr {
    template< class T >
    using forward_list = std::forward_list<T, std::pmr::polymorphic_allocator<T>>;
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
| `iterator` | 前向迭代器（实现定义，只能递增） |
| `const_iterator` | 常量前向迭代器 |

**注意：** `forward_list` 没有反向迭代器类型（无 `reverse_iterator`），也没有 `const_reverse_iterator`。

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `forward_list()` | 默认构造一个空 `forward_list` |
| `explicit forward_list(const Allocator& alloc)` | 使用指定分配器构造空 `forward_list` |
| `explicit forward_list(size_type count)` | 构造包含 `count` 个默认插入元素的 `forward_list` |
| `forward_list(size_type count, const T& value)` | 构造包含 `count` 个 `value` 副本的 `forward_list` |
| `template<class InputIt> forward_list(InputIt first, InputIt last)` | 用迭代器范围 `[first, last)` 构造 `forward_list` |
| `forward_list(const forward_list& other)` | 拷贝构造函数 |
| `forward_list(forward_list&& other)` | 移动构造函数（C++11） |
| `forward_list(std::initializer_list<T> init)` | 用初始化列表构造（C++11） |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~forward_list()` | 销毁 `forward_list` 中的所有元素并释放内存 |

**示例：**
```cpp
std::forward_list<int> fl1;                     // 空
std::forward_list<int> fl2(5);                  // 5 个默认值（0）
std::forward_list<int> fl3(5, 42);              // 5 个 42
std::forward_list<int> fl4(fl3);                // 拷贝构造
std::forward_list<int> fl5 = {1,2,3,4,5};       // 初始化列表
std::forward_list<int> fl6(std::move(fl5));     // 移动构造
```

**实现原理：**
- 默认构造时，通常只创建一个空链表（头指针为 `nullptr`）。
- 拷贝构造时，遍历原链表并逐节点复制。
- 移动构造时，直接接管原链表的头指针，原链表变为空。

**线程安全提示：**
构造和析构操作应在单线程环境进行，或在有锁保护的情况下执行。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const forward_list& other)` | 拷贝赋值 |
| `operator=(forward_list&& other)` | 移动赋值（C++11） |
| `operator=(std::initializer_list<T> init)` | 初始化列表赋值（C++11） |
| `assign(size_type count, const T& value)` | 用 `count` 个 `value` 副本替换内容 |
| `template<class InputIt> assign(InputIt first, InputIt last)` | 用迭代器范围替换内容 |
| `assign(std::initializer_list<T> init)` | 用初始化列表替换内容（C++11） |

**示例：**
```cpp
std::forward_list<int> fl1 = {1,2,3};
std::forward_list<int> fl2;
fl2 = fl1;                           // 拷贝赋值
fl2.assign(5, 42);                   // 变成 5 个 42
fl2.assign({10, 20, 30});            // 变成 10, 20, 30
```

**实现原理：**
- 拷贝/移动赋值时，先清空自身，再复制或接管节点。

**线程安全提示：**
多线程中并发对同一个 `forward_list` 进行赋值操作会导致数据竞争，需要加锁保护。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `front()` | 返回第一个元素的引用（链表不能为空，否则未定义行为） |

**示例：**
```cpp
std::forward_list<int> fl = {10, 20, 30};
int a = fl.front();   // 10
```

**实现原理：**
直接返回头节点指向的元素的引用。

**时间复杂度：** O(1)

**线程安全提示：**
多个线程同时读取 `front()` 是安全的；一个线程写入而另一个线程读取会导致数据竞争。

---

### 4. 迭代器

| 函数 | 说明 |
|------|------|
| `begin()` / `cbegin()` | 返回指向第一个元素的迭代器 |
| `end()` / `cend()` | 返回指向尾后位置的迭代器 |
| `before_begin()` | 返回指向第一个元素之前位置的迭代器（C++11） |
| `cbefore_begin()` | 常量版本（C++11） |

**示例：**
```cpp
std::forward_list<int> fl = {1,2,3,4,5};
for (auto it = fl.begin(); it != fl.end(); ++it) {
    std::cout << *it << " ";
}
// 在头部之前插入
auto it = fl.before_begin();
fl.insert_after(it, 0);   // 在头部插入 0
```

**实现原理：**
- `begin()` 返回指向头节点的迭代器。
- `end()` 返回一个代表 `nullptr` 的迭代器。
- `before_begin()` 返回一个特殊的迭代器，指向头节点之前的位置，用于在头部之前插入（因为单向链表无法直接获取前驱）。

**线程安全提示：**
遍历过程中，如果另一个线程修改链表，迭代器可能失效。多线程遍历时应加锁。

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查链表是否为空 |
| `max_size()` | 返回理论最大元素个数 |

**注意：** `forward_list` **没有 `size()` 成员函数**。因为单向链表不维护大小计数器（为了节省内存和性能），需要大小可以通过 `std::distance(begin(), end())` 获得，但复杂度为 O(n)。

**示例：**
```cpp
std::forward_list<int> fl;
bool e = fl.empty();   // true
size_t max = fl.max_size();
```

**实现原理：**
- `empty()` 检查头指针是否为 `nullptr`。
- `max_size()` 由分配器决定。

**时间复杂度：** O(1)

**线程安全提示：** 只读，安全。

---

### 6. 修改器

| 函数 | 说明 |
|------|------|
| `clear()` | 清空所有元素 |
| `insert_after(const_iterator pos, const T& value)` | 在 `pos` 之后插入元素 |
| `insert_after(const_iterator pos, T&& value)` | 插入右值（C++11） |
| `insert_after(const_iterator pos, size_type count, const T& value)` | 在 `pos` 之后插入多个相同元素 |
| `template<class InputIt> insert_after(const_iterator pos, InputIt first, InputIt last)` | 插入迭代器范围 |
| `insert_after(const_iterator pos, std::initializer_list<T> init)` | 插入初始化列表（C++11） |
| `emplace_after(const_iterator pos, Args&&... args)` | 在 `pos` 之后原位构造元素（C++11） |
| `emplace_front(Args&&... args)` | 在头部原位构造元素（C++11） |
| `push_front(const T& value)` | 在头部添加元素 |
| `push_front(T&& value)` | 在头部添加右值元素（C++11） |
| `pop_front()` | 移除第一个元素 |
| `erase_after(const_iterator pos)` | 移除 `pos` 之后的一个元素 |
| `erase_after(const_iterator pos, const_iterator last)` | 移除 `(pos, last)` 范围内的元素 |
| `resize(size_type count)` | 调整链表大小 |
| `resize(size_type count, const value_type& value)` | 调整链表大小，新元素填充 `value` |
| `swap(forward_list& other)` | 交换两个链表的内容 |

**示例：**
```cpp
std::forward_list<int> fl = {2,3,4};
fl.push_front(1);                    // 1,2,3,4
fl.pop_front();                      // 2,3,4
auto it = fl.begin();
fl.insert_after(it, 99);             // 2,99,3,4
fl.erase_after(it);                  // 2,3,4
fl.emplace_front(0);                 // 0,2,3,4
```

**实现原理：**
- `push_front`：分配新节点，使其 `next` 指向原头节点，更新头指针。
- `insert_after`：在给定节点之后插入新节点，O(1)。
- `erase_after`：删除给定节点之后的节点，O(1)。
- 单向链表无法直接获取前驱，因此没有 `insert_before`，而是提供 `before_begin()` + `insert_after` 实现在头部之前插入。

**时间复杂度：**
- `push_front` / `pop_front` / `insert_after` / `erase_after`：O(1)。
- `insert_after`（多个元素）：O(k)（k 为插入数量）。
- `erase_after`（范围）：O(n)（n 为删除元素个数）。
- `resize`：O(n)（n 为链表大小）。

**线程安全提示：**
多个线程并发修改需要加锁。

---

### 7. 操作（Operations）

`forward_list` 提供了链表特有的操作，如拼接、合并、去重、排序等。

| 函数 | 说明 |
|------|------|
| `splice_after(const_iterator pos, forward_list& other)` | 将 `other` 的所有节点移动到 `pos` 之后 |
| `splice_after(const_iterator pos, forward_list&& other)` | 移动版本 |
| `splice_after(const_iterator pos, forward_list& other, const_iterator it)` | 将 `it` 之后的单个节点移动到 `pos` 之后 |
| `splice_after(const_iterator pos, forward_list& other, const_iterator first, const_iterator last)` | 将 `(first, last)` 范围内的节点移动 |
| `remove(const T& value)` | 移除所有等于 `value` 的元素 |
| `remove_if(UnaryPredicate p)` | 移除所有满足谓词 `p` 的元素 |
| `unique()` | 移除相邻重复元素（要求已排序） |
| `unique(BinaryPredicate p)` | 移除相邻满足 `p` 的元素 |
| `merge(forward_list& other)` | 合并两个已排序的链表（归并） |
| `merge(forward_list&& other)` | 移动版本 |
| `sort()` | 排序（默认升序） |
| `sort(Compare comp)` | 自定义比较器排序 |
| `reverse()` | 反转链表 |

**示例：**
```cpp
std::forward_list<int> fl1 = {1,3,5};
std::forward_list<int> fl2 = {2,4,6};
fl1.merge(fl2);                    // fl1: 1,2,3,4,5,6, fl2 为空
fl1.reverse();                     // 6,5,4,3,2,1
fl1.sort();                        // 1,2,3,4,5,6
fl1.unique();                      // 移除相邻重复
fl1.remove(3);                     // 移除所有 3
```

**实现原理：**
- `splice_after`：通过调整节点指针移动节点，不复制元素。
- `merge`：归并两个有序链表，O(n)。
- `sort`：通常实现为归并排序，O(n log n)。
- `reverse`：遍历链表，反转指针方向，O(n)。

**线程安全提示：**
所有操作都修改链表，需要同步。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 `forward_list` |
| `swap(std::forward_list& lhs, std::forward_list& rhs)` | 交换两个链表的内容 |
| `erase(std::forward_list& c, const U& value)` | 移除所有等于 `value` 的元素（C++20） |
| `erase_if(std::forward_list& c, Predicate pred)` | 移除所有满足谓词 `pred` 的元素（C++20） |

---

## 五、宏与常量

`<forward_list>` 头文件中**没有定义任何宏**。

---

## 六、实现原理

`std::forward_list` 的典型实现是一个单向链表，每个节点包含一个元素和指向下一个节点的指针。为了支持 `before_begin()` 迭代器，通常存在一个虚拟头节点（不存储元素），或者将头指针视为 `before_begin` 位置。

**节点结构（简化）：**
```cpp
struct Node {
    T value;
    Node* next;
};
```

**迭代器：** 前向迭代器，只支持 `++` 操作，不支持 `--`。迭代器内部封装一个指向 `Node` 的指针。

**`before_begin()` 迭代器：** 代表“虚拟头节点”的位置，用于在链表头部之前插入元素。`insert_after(before_begin(), x)` 等价于 `push_front(x)`。

**优点：**
- 内存开销小（每个节点一个指针）。
- 在已知前驱的情况下，插入和删除 O(1)。
- 不需要维护双向指针，操作简单。

**缺点：**
- 只能单向遍历。
- 无法直接获取前驱，因此没有 `insert_before`，只能 `insert_after`。
- 不能反向迭代，没有 `rbegin()` / `rend()`。
- 不提供 `size()`，需要 O(n) 计算。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| `push_front` / `pop_front` | O(1) |
| `insert_after` / `erase_after`（单个） | O(1) |
| `insert_after` / `erase_after`（范围） | O(k) |
| 查找（按值） | O(n) |
| `empty()` / `max_size()` | O(1) |
| `splice_after` | O(1) 移动单个节点；O(n) 移动范围（因需确定 last） |
| `merge` | O(n) |
| `sort` | O(n log n) |
| `reverse` | O(n) |
| `unique` / `remove` / `remove_if` | O(n) |

---

## 八、线程安全

**`std::forward_list` 本身不是线程安全的**。规则与其它容器类似：
- **同时读取**：多个线程同时读取不同节点是安全的。
- **读取 + 写入**：一个线程修改链表（添加、删除、拼接等）时，其他线程不能进行任何操作，否则数据竞争。
- **同时写入**：多个线程同时修改链表会导致未定义行为。

**保证线程安全的常用方法：**
- 使用互斥锁保护整个 `forward_list`。
- 使用线程局部存储。
- 使用第三方并发容器。

---

## 九、各标准版本新增特性

### C++11
- 首次引入 `std::forward_list`。

### C++14
- 无明显针对 `forward_list` 的新特性。

### C++17
- `std::pmr::forward_list` 多态分配器别名模板。
- `insert_after` 等成员函数支持节点提取。

### C++20
- `erase()` 和 `erase_if()` 非成员函数。
- `constexpr` 修饰部分成员函数。

---

## 十、迭代器失效规则

| 操作 | 失效范围 |
|------|----------|
| `push_front` | 所有迭代器不失效（但 `before_begin()` 仍有效） |
| `pop_front` | 指向被删除元素的迭代器失效 |
| `insert_after(pos, ...)` | 不影响 `pos` 之前的迭代器；`pos` 之后的迭代器可能失效（因新节点插入后，`pos` 的 `next` 改变） |
| `erase_after(pos)` | 被删除的节点及之后的迭代器失效；`pos` 本身有效 |
| `splice_after` | 被移动的节点在原链表中的迭代器失效；在目标链表中重新有效 |
| `sort` / `reverse` | 所有迭代器失效（因为节点重新链接） |
| `swap` | 所有迭代器失效（指向其他容器） |
| `clear` | 所有迭代器失效 |

---

## 十一、相关结构体与分配器

| 组件 | 说明 |
|------|------|
| `std::allocator<T>` | 默认分配器 |
| `std::pmr::polymorphic_allocator<T>` | 多态分配器（C++17） |
