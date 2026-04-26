## `<numeric>` 头文件详解

`<numeric>` 是 C++ 标准模板库（STL）中定义的头文件，提供了**数值算法**。它包含了元素求和、内积、前缀和、差分、最大公约数、最小公倍数等通用数值操作的函数模板。这些算法以迭代器范型设计，可应用于任何容器的数值序列。

---

## 一、函数详解

### 1. 累加操作

#### `std::accumulate`

**函数原型：**
```cpp
template<class InputIt, class T>
T accumulate(InputIt first, InputIt last, T init);

template<class InputIt, class T, class BinaryOp>
T accumulate(InputIt first, InputIt last, T init, BinaryOp op);
```

**作用：** 对范围 `[first, last)` 进行累加（或自定义二元运算）求和。

**参数：**
- `first`, `last`：输入迭代器范围。
- `init`：初始值。
- `op`（可选）：二元运算，默认为 `std::plus<>()`（即加法）。

**返回值：** 累加结果，类型为 `T`。

**示例：**
```cpp
#include <numeric>
#include <vector>
std::vector<int> v{1,2,3,4};
int sum = std::accumulate(v.begin(), v.end(), 0); // 10
int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<>()); // 24
```

**实现原理：** 依次对每个元素应用操作：`init = op(init, *it)`。

**线程安全提示：** 对同一范围的并发访问不安全，需要外部同步。

---

### 2. 内积

#### `std::inner_product`

**函数原型：**
```cpp
template<class InputIt1, class InputIt2, class T>
T inner_product(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init);

template<class InputIt1, class InputIt2, class T,
         class BinaryOp1, class BinaryOp2>
T inner_product(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init,
                BinaryOp1 op1, BinaryOp2 op2);
```

**作用：** 计算两个序列的内积（默认对应元素相乘后累加）。`op1` 用于累加，`op2` 用于配对。

**返回值：** 内积结果（类型 `T`）。

**示例：**
```cpp
std::vector<int> a{1,2,3}, b{4,5,6};
int ip = std::inner_product(a.begin(), a.end(), b.begin(), 0); // 1*4+2*5+3*6=32
```

---

### 3. 部分和与差分

#### `std::partial_sum`

**函数原型：**
```cpp
template<class InputIt, class OutputIt>
OutputIt partial_sum(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class BinaryOp>
OutputIt partial_sum(InputIt first, InputIt last, OutputIt d_first, BinaryOp op);
```

**作用：** 计算前缀和（或前缀自定义运算），将结果写入输出范围。

**返回值：** 指向输出范围末尾的迭代器。

**示例：**
```cpp
std::vector<int> v{1,2,3,4}, result(4);
std::partial_sum(v.begin(), v.end(), result.begin()); // result = {1,3,6,10}
```

---

#### `std::adjacent_difference`

**函数原型：**
```cpp
template<class InputIt, class OutputIt>
OutputIt adjacent_difference(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class BinaryOp>
OutputIt adjacent_difference(InputIt first, InputIt last, OutputIt d_first, BinaryOp op);
```

**作用：** 计算相邻元素的差（或自定义运算），第一个元素保持不变，后续元素为 `*(d_first + i) = *(first + i) - *(first + i - 1)`。

**返回值：** 指向输出范围末尾的迭代器。

**示例：**
```cpp
std::vector<int> v{1,3,6,10}, result(4);
std::adjacent_difference(v.begin(), v.end(), result.begin()); // {1,2,3,4}
```

---

### 4. 最大公约数与最小公倍数（C++17）

#### `std::gcd`

**函数原型：**
```cpp
template<class M, class N>
constexpr std::common_type_t<M,N> gcd(M m, N n);
```

**作用：** 返回 `|m|` 和 `|n|` 的最大公约数。如果两个参数都为 0，则返回 0。

**示例：**
```cpp
int g = std::gcd(12, 18); // 6
```

**实现原理：** 使用欧几里得算法（递归或迭代）。

**线程安全提示：** 纯函数，安全。

---

#### `std::lcm`

**函数原型：**
```cpp
template<class M, class N>
constexpr std::common_type_t<M,N> lcm(M m, N n);
```

**作用：** 返回 `|m|` 和 `|n|` 的最小公倍数。任一参数为 0 则返回 0。

**示例：**
```cpp
int l = std::lcm(12, 18); // 36
```

---

### 5. 归约相关（C++17）

#### `std::reduce`

**函数原型：**
```cpp
template<class InputIt>
typename std::iterator_traits<InputIt>::value_type reduce(InputIt first, InputIt last);

template<class InputIt, class T>
T reduce(InputIt first, InputIt last, T init);

template<class InputIt, class T, class BinaryOp>
T reduce(InputIt first, InputIt last, T init, BinaryOp op);
```

**作用：** 类似 `accumulate`，但允许进行无序归约（元素顺序可任意结合），前提是二元运算满足结合律和交换律。可能在并行版本中用于提升性能。

**注意：** C++17 中 `reduce` 对顺序没有要求，结果可能不确定（如果运算不满足交换律）。

---

#### `std::inclusive_scan` / `std::exclusive_scan`

**函数原型（部分）：**
```cpp
template<class InputIt, class OutputIt>
OutputIt inclusive_scan(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class BinaryOp>
OutputIt inclusive_scan(InputIt first, InputIt last, OutputIt d_first, BinaryOp op);

template<class InputIt, class OutputIt>
OutputIt exclusive_scan(InputIt first, InputIt last, OutputIt d_first, T init);
```

**作用：** 计算前缀和（包含当前元素或不包含）。与 `partial_sum` 类似，但 `exclusive_scan` 的第一个输出元素是 `init`（不包含第一个输入元素）。

---

#### `std::transform_reduce`（C++17）

**函数原型：**
```cpp
template<class InputIt1, class InputIt2, class T>
T transform_reduce(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init);

template<class InputIt1, class InputIt2, class T, class BinaryOp1, class BinaryOp2>
T transform_reduce(InputIt1 first1, InputIt1 last1, InputIt2 first2,
                   T init, BinaryOp1 reduce_op, BinaryOp2 transform_op);
```

**作用：** 对两个序列的元素应用 `transform_op`，然后对结果应用 `reduce_op`（归约）。可以视为 `inner_product` 的广义版本，支持结合律和交换律。

---

## 二、结构体详解

`<numeric>` 本身不定义任何结构体。它仅提供模板算法函数。

---

## 三、宏与常量

`<numeric>` 头文件中没有定义任何宏。

---

## 四、类型定义

`<numeric>` 不定义新类型，但使用标准迭代器类型和 `std::common_type_t` 等工具类型。

---

## 五、模板声明

`<numeric>` 包含大量函数模板，上述已覆盖主要部分。C++20 还增加了 `std::midpoint` 和 `std::lerp`（线性插值）等函数。

---
