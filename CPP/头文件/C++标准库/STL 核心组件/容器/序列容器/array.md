## `<array>` 头文件详解

`<array>` 是 C++ 标准模板库（STL）中定义的头文件，提供了 `std::array` 容器。`std::array` 是一个固定大小的数组容器，封装了 C 风格数组，同时提供了 STL 容器的接口（如迭代器、大小获取、边界检查等），且没有动态内存分配的开销。

---

## 一、模板参数

```cpp
template< class T, std::size_t N >
struct array;
```

| 参数 | 说明 |
|------|------|
| `T` | 元素类型。必须满足 `Erased` 要求（即析构函数不抛出异常等），具体取决于实际执行的操作 |
| `N` | 数组大小（编译时常量，类型为 `std::size_t`）。`N` 不能为 0（C++11 起允许 `N == 0`，但此时 `array` 没有元素，某些操作行为特殊） |

---

## 二、成员类型

| 类型别名 | 说明 |
|----------|------|
| `value_type` | `T` |
| `size_type` | `std::size_t` |
| `difference_type` | `std::ptrdiff_t` |
| `reference` | `value_type&` |
| `const_reference` | `const value_type&` |
| `pointer` | `T*` |
| `const_pointer` | `const T*` |
| `iterator` | 随机访问迭代器（实现定义，通常为 `T*`） |
| `const_iterator` | 常量随机访问迭代器（通常为 `const T*`） |
| `reverse_iterator` | `std::reverse_iterator<iterator>` |
| `const_reverse_iterator` | `std::reverse_iterator<const_iterator>` |

---

## 三、成员函数详解

### 1. 构造函数与析构函数

| 构造函数 | 说明 |
|----------|------|
| `array()` | 默认构造函数。C++11 中默认是隐式定义的；C++14 起要求将元素默认初始化（对于非类类型，初始值不确定） |
| `array(const array& other)` | 拷贝构造函数（隐式定义） |
| `array(array&& other)` | 移动构造函数（隐式定义） |
| `array(std::initializer_list<T>)` | **不存在**！`std::array` 不能用初始化列表直接构造，但可以用聚合初始化 |

**析构函数：**
| 析构函数 | 说明 |
|----------|------|
| `~array()` | 隐式定义的析构函数，按逆序销毁元素 |

**示例：**
```cpp
std::array<int, 5> arr1;               // 元素未初始化（int 类型，值不确定）
std::array<int, 5> arr2 = {1,2,3,4,5}; // 聚合初始化
std::array<int, 5> arr3 = arr2;        // 拷贝构造
std::array<int, 5> arr4 = std::move(arr3); // 移动构造
```

**实现原理：**
`std::array` 是一个聚合体（aggregate），通常内部包含一个 C 风格数组（如 `T _M_elems[N]`）。它不管理动态内存，所有内存都在栈上或作为其他对象的一部分。默认构造时，若 `T` 是类类型，会调用其默认构造函数；若 `T` 是基本类型，则不被初始化（除非通过值初始化，如 `std::array<int, 5>()` 会进行零初始化）。

**线程安全提示：**
构造和析构操作应在单线程环境进行。如果多个线程访问不同的 `array` 实例，则没有数据竞争。

---

### 2. 赋值操作

| 操作符/函数 | 说明 |
|-------------|------|
| `operator=(const array& other)` | 拷贝赋值（隐式定义） |
| `operator=(array&& other)` | 移动赋值（隐式定义） |
| `fill(const T& value)` | 将数组中的所有元素设置为 `value` |

**示例：**
```cpp
std::array<int, 5> arr1 = {1,2,3,4,5};
std::array<int, 5> arr2;
arr2 = arr1;              // 拷贝赋值
arr2.fill(42);            // 全部变为 42
```

**实现原理：**
- 拷贝/移动赋值会逐一复制/移动每个元素。
- `fill` 循环为每个元素赋值。

**时间复杂度：**
- `operator=`：O(N)
- `fill`：O(N)

**线程安全提示：**
并发对同一个 `array` 进行赋值会导致数据竞争，需要加锁。

---

### 3. 元素访问

| 函数 | 说明 |
|------|------|
| `at(size_type pos)` | 返回指定位置的元素（带边界检查），越界抛出 `std::out_of_range` |
| `operator[](size_type pos)` | 返回指定位置的元素（不带边界检查），越界行为未定义 |
| `front()` | 返回第一个元素的引用（若 `N==0`，则行为未定义） |
| `back()` | 返回最后一个元素的引用（若 `N==0`，则行为未定义） |
| `data()` | 返回指向底层数组的指针 |

**示例：**
```cpp
std::array<int, 5> arr = {10,20,30,40,50};
int a = arr[0];            // 10
int b = arr.at(1);         // 20
int c = arr.front();       // 10
int d = arr.back();        // 50
int* p = arr.data();       // 指向第一个元素的指针
```

**实现原理：**
- `operator[]` 通常直接返回 `_M_elems[pos]`。
- `at()` 会检查 `pos < N`，否则抛出异常。
- `front()` 返回 `_M_elems[0]`，`back()` 返回 `_M_elems[N-1]`。
- `data()` 返回 `_M_elems` 的地址。

**时间复杂度：**
所有元素访问操作都是 O(1)。

**线程安全提示：**
多个线程同时读取不同元素是安全的；一个线程写入而另一个线程读取同一元素会导致数据竞争。

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
std::array<int, 5> arr = {1,2,3,4,5};
for (auto it = arr.begin(); it != arr.end(); ++it) {
    std::cout << *it << " ";
}
for (auto rit = arr.rbegin(); rit != arr.rend(); ++rit) {
    std::cout << *rit << " ";
}
```

**实现原理：**
`begin()` 返回指向 `_M_elems[0]` 的指针（或封装指针的迭代器），`end()` 返回指向 `_M_elems[N]` 的指针。

**线程安全提示：**
遍历过程中，如果另一个线程修改 `array`，则迭代器不会失效（因为大小固定），但可能读取到修改中的值，产生数据竞争。多线程遍历时应加锁。

---

### 5. 容量

| 函数 | 说明 |
|------|------|
| `empty()` | 检查容器是否为空（`N == 0`） |
| `size()` | 返回元素个数（`N`） |
| `max_size()` | 返回理论最大元素个数（也是 `N`） |

**示例：**
```cpp
std::array<int, 5> arr;
static_assert(arr.size() == 5);
bool empty = arr.empty();   // false (因为 N=5)
```

**实现原理：**
所有容量函数都是 `constexpr` 且返回常量值（基于模板参数 `N`）。

**时间复杂度：**
O(1)。

**线程安全提示：**
只读操作，线程安全。

---

### 6. 修改器

| 函数 | 说明 |
|------|------|
| `fill(const T& value)` | 将所有元素设置为 `value` |
| `swap(array& other)` | 交换两个 `array` 的内容（要求 `N` 相同） |

**示例：**
```cpp
std::array<int, 5> arr1 = {1,2,3,4,5};
std::array<int, 5> arr2 = {6,7,8,9,10};
arr1.swap(arr2);            // arr1 变成 {6,7,8,9,10}，arr2 反之
```

**实现原理：**
- `fill`：循环为每个元素赋值。
- `swap`：循环交换每个元素（C++11 中为线性复杂度，但 C++17 起要求为线性；实际可能通过逐个元素交换实现，不移动内存）。

**时间复杂度：**
- `fill`：O(N)
- `swap`：O(N)（因为必须交换所有元素，但不会抛出异常）

**线程安全提示：**
并发修改需要同步。

---

## 四、非成员函数

| 函数 | 说明 |
|------|------|
| `operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=` | 比较两个 `array`（按字典序） |
| `swap(std::array& lhs, std::array& rhs)` | 交换两个 `array` 的内容 |
| `std::get<I>(array)` | 通过编译时索引访问元素（C++14 起 `constexpr`） |
| `std::tuple_size<array>` | 获取 `array` 大小（用于元组协议） |
| `std::tuple_element<I, array>` | 获取元素类型（用于元组协议） |
| `std::to_array` (C++20) | 从内置数组或初始化列表创建 `std::array` |

**示例（C++20）：**
```cpp
auto arr1 = std::to_array({1,2,3,4,5});  // std::array<int,5>
int a[] = {1,2,3};
auto arr2 = std::to_array(a);            // std::array<int,3>
```

---

## 五、宏与常量

`<array>` 头文件中**没有定义任何宏**。所有信息通过模板参数和成员函数获得。

---

## 六、实现原理

`std::array` 是一个聚合类，典型实现如下（简化）：

```cpp
template<typename T, size_t N>
struct array {
    T _M_elems[N];
    // 成员函数...
};
```

由于没有动态内存分配，`array` 对象的大小就是 `N * sizeof(T)`，没有额外的开销。所有元素直接存储在对象内部（栈上或作为包含对象的成员）。

**`array` 与内置数组的对比：**
- 支持 STL 算法（有迭代器）。
- 支持 `size()`、`empty()` 等成员函数。
- 支持赋值操作（拷贝/移动）。
- 可作函数返回值（内置数组不能直接返回）。
- 元素访问时可通过 `at()` 进行边界检查。
- 不退化到指针（保持大小信息）。

**`array` 与 `vector` 的对比：**
- `array` 大小固定，无动态内存分配，性能与内置数组相同。
- `vector` 大小可变，有动态分配开销。
- `array` 的 `swap` 是线性复杂度（必须交换每个元素），而 `vector` 的 `swap` 是常数复杂度（只交换指针）。

**`std::array< T, 0 >` 特化：**
当 `N == 0` 时，`array` 不包含任何元素。`front()`、`back()` 和 `data()` 的行为未定义（但 `data()` 可能返回一个有效指针，解引用它未定义）。迭代器的 `begin()` 和 `end()` 返回相同值。

---

## 七、时间复杂度

| 操作 | 时间复杂度 |
|------|------------|
| 随机访问（`operator[]`、`at()`） | O(1) |
| 迭代器操作（`begin`、`end` 等） | O(1) |
| `empty()`、`size()`、`max_size()` | O(1) |
| `fill()` | O(N) |
| `swap()` | O(N) |
| 元素比较操作 | O(N) |

---

## 八、线程安全

`std::array` 与内置数组的线程安全规则相同：
- **同时读取**：多个线程读取不同元素是安全的。
- **读取 + 写入**：一个线程写入某元素时，其他线程不能读取或写入同一元素（会导致数据竞争）。
- **同时写入**：多个线程写入不同元素也是不安全的（因为对内存的并发写入可能撕裂，虽然 `array` 本身不提供原子性）。
- **修改容器本身**：如调用 `fill` 或 `swap`，需要全局同步。

**保证线程安全的常用方法：**
- 使用互斥锁保护整个 `array`。
- 使用原子类型 `std::atomic<T>` 作为元素（但 `array<std::atomic<int>, N>` 的原子操作是单独对每个元素的）。
- 线程局部存储（每个线程独立 `array`）。

---

## 九、各标准版本新增特性

### C++11
- 首次引入 `std::array`。

### C++14
- `begin()`、`end()`、`cbegin()`、`cend()`、`rbegin()`、`rend()`、`front()`、`back()`、`data()` 变为 `constexpr`。
- `at()` 变为 `constexpr`（但无法编译时检测异常）。

### C++17
- 支持结构化绑定（因为 `array` 是聚合体）。
- `empty()` 变为 `constexpr`。

### C++20
- 添加 `std::to_array` 函数。
- `operator==` 等比较操作符由编译器自动生成（通过 `<=>`）。
- `swap` 函数变为 `constexpr`。

---

## 十、迭代器失效规则

`std::array` 的迭代器**永远不会失效**，除非 `array` 对象本身被销毁。因为：
- `array` 大小固定，不进行内存重新分配。
- 元素始终存储在固定内存地址。
- 没有插入或删除操作。

**注意：** 移动 `array` 对象后，原 `array` 的迭代器仍然有效（但指向的是被移动后的元素，内容已移动，不应再使用）。通常移动后不应再使用原对象。

---

## 十一、相关结构体与类型特征

| 组件 | 说明 |
|------|------|
| `std::tuple_size<std::array<T, N>>` | 提供 `value` 静态常量等于 `N` |
| `std::tuple_element<I, std::array<T, N>>` | 提供 `type` 成员类型等于 `T`（若 `I < N`） |
| `std::to_array` (C++20) | 辅助函数模板 |
| `std::get<I>(array)` | 非成员函数，通过索引访问元素 |

---
