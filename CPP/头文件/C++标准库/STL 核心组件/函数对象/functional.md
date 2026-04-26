## `<functional>` 头文件详解

`<functional>` 是 C++ 标准库中提供**函数对象（functor）**、**绑定器**、**包装器**以及**哈希函数**的头文件。它包含了 `std::function` 通用函数包装器、`std::bind` 参数绑定器、`std::ref` / `std::cref` 引用包装器、预定义的函数对象（如 `std::plus`、`std::logical_and`）、以及 `std::hash` 哈希函数特化等。该头文件是 C++ 泛型编程和函数式编程风格的重要基础设施，自 C++11 起大幅强化。

---

## 一、函数对象（预定义仿函数）

以下结构体模板均定义了 `operator()`，用于实现算术运算、比较、逻辑运算和位运算等。它们通常与算法（如 `std::sort`）或容器（如 `std::set`）一起使用，作为比较或操作策略。

### 1. 算术运算

| 类型 | 作用 |
|------|------|
| `std::plus<T>` | `operator()(const T& x, const T& y)` 返回 `x + y` |
| `std::minus<T>` | `x - y` |
| `std::multiplies<T>` | `x * y` |
| `std::divides<T>` | `x / y` |
| `std::modulus<T>` | `x % y` |
| `std::negate<T>` | `-x` |

### 2. 比较运算

| 类型 | 作用 |
|------|------|
| `std::equal_to<T>` | `x == y` |
| `std::not_equal_to<T>` | `x != y` |
| `std::greater<T>` | `x > y` |
| `std::less<T>` | `x < y` |
| `std::greater_equal<T>` | `x >= y` |
| `std::less_equal<T>` | `x <= y` |

### 3. 逻辑运算

| 类型 | 作用 |
|------|------|
| `std::logical_and<T>` | `x && y` |
| `std::logical_or<T>` | `x \|\| y` |
| `std::logical_not<T>` | `!x` |

### 4. 位运算（C++14）

| 类型 | 作用 |
|------|------|
| `std::bit_and<T>` | `x & y` |
| `std::bit_or<T>` | `x \| y` |
| `std::bit_xor<T>` | `x ^ y` |
| `std::bit_not<T>` | `~x` |

这些算术、比较、逻辑、位运算的仿函数都继承自 `std::binary_function` 或 `std::unary_function`（C++11 前），C++17 后不再需要继承。

**示例：**
```cpp
#include <functional>
#include <algorithm>
#include <vector>
std::vector<int> v = {1,2,3};
std::transform(v.begin(), v.end(), v.begin(), std::negate<int>());
std::sort(v.begin(), v.end(), std::greater<int>());
```

---

## 二、函数包装器

### 1. `std::function`

**模板参数：**
```cpp
template<class> class function; // 未定义
template<class Ret, class... Args>
class function<Ret(Args...)>;
```

**作用：** 可调用对象的包装器，可以存储任何可调用对象（函数、函数指针、lambda、bind 表达式、其他 `function` 对象等），只要其调用签名与模板参数兼容。

**主要成员函数：**

| 成员 | 说明 |
|------|------|
| `function()` | 默认构造，为空 |
| `function(const function&)` | 拷贝构造 |
| `function(function&&)` | 移动构造（C++11） |
| `function(nullptr_t)` | 构造空对象 |
| `template<class F> function(F f)` | 从可调用对象构造 |
| `~function()` | 析构 |
| `function& operator=(const function&)` | 拷贝赋值 |
| `function& operator=(function&&)` | 移动赋值 |
| `void swap(function&)` | 交换内容 |
| `explicit operator bool() const` | 检查是否非空 |
| `Ret operator()(Args... args) const` | 调用存储的可调用对象 |

**示例：**
```cpp
#include <functional>
#include <iostream>
int add(int a, int b) { return a + b; }
int main() {
    std::function<int(int,int)> func = add;
    std::cout << func(2,3); // 5
    func = [](int x, int y) { return x * y; };
    std::cout << func(2,3); // 6
}
```

**实现原理：** 采用类型擦除技术，内部存储一个可调用对象的基类指针，通过虚调用或类型特化实现。

**线程安全提示：** `std::function` 对象本身不是线程安全的；对同一实例的并发读写需要同步。

---

### 2. `std::bad_function_call`

**定义：**
```cpp
class bad_function_call : public std::exception;
```

**作用：** 当调用空的 `std::function` 对象时抛出的异常类型。

---

## 三、绑定器

### `std::bind`

**函数原型：**
```cpp
template<class F, class... Args>
/*unspecified*/ bind(F&& f, Args&&... args);
```

**作用：** 生成一个函数包装器，将参数绑定到可调用对象 `f` 的特定参数上，并支持占位符 `std::placeholders::_1, _2, ...` 表示用户提供的参数。

**返回类型：** 未指定但满足 `std::is_bind_expression` 和 `std::is_placeholder` 特性，可存储在 `std::function` 中或直接调用。

**占位符命名空间：** `std::placeholders` 中的 `_1`, `_2`, …, `_N`。

**示例：**
```cpp
#include <functional>
#include <iostream>
int add(int a, int b) { return a + b; }
int main() {
    using namespace std::placeholders;
    auto inc = std::bind(add, _1, 1); // 绑定第二个参数为 1
    std::cout << inc(5); // 6
    auto add5 = std::bind(add, 5, _1);
    std::cout << add5(3); // 8
}
```

**实现原理：** 返回一个内部仿函数，存储所有绑定参数，调用时使用 `INVOKE` 语义。

**线程安全提示：** 构造和调用不涉及共享状态，但结果可调用对象可能包含拷贝的参数，线程安全取决于参数类型。

---

## 四、引用包装器

### `std::reference_wrapper`

**定义：**
```cpp
template<class T> class reference_wrapper;
```

**作用：** 包装引用，允许将引用存储在容器中或传递给函数对象（通常用于 `std::bind` 避免拷贝）。

**主要成员：**
- 构造函数（拷贝构造，从 `T&` 转换）
- `operator T&() const`：隐式转换回引用
- `T& get() const`：显式获取引用

**辅助函数：**
- `std::ref(T& t)` → `reference_wrapper<T>`
- `std::cref(const T& t)` → `reference_wrapper<const T>`

**示例：**
```cpp
#include <functional>
void inc(int& x) { ++x; }
int main() {
    int a = 0;
    auto f = std::bind(inc, std::ref(a));
    f();
    // a == 1
}
```

---

## 五、哈希函数

### `std::hash`

**定义：**
```cpp
template<class Key> struct hash;
```

**作用：** 提供对常用类型（基本类型、指针、`std::string`、`std::thread::id` 等）的哈希函数对象，用于无序关联容器（`std::unordered_set`、`std::unordered_map`）。用户可特化 `hash` 用于自定义类型。

**示例：**
```cpp
#include <functional>
#include <unordered_set>
struct MyType { int x; };
namespace std {
    template<> struct hash<MyType> {
        size_t operator()(const MyType& t) const {
            return hash<int>()(t.x);
        }
    };
}
std::unordered_set<MyType> s;
```

---

## 六、类型特征（辅助 traits）

### `std::is_bind_expression`

```cpp
template<class T> struct is_bind_expression;
```

**作用：** 指示 `T` 是否为 `std::bind` 返回的类型。`std::is_bind_expression<T>::value` 在 `T` 为 `bind` 表达式时返回 `true`。

### `std::is_placeholder`

```cpp
template<class T> struct is_placeholder;
```

**作用：** 指示 `T` 是否为占位符（`_1`, `_2` …）。`is_placeholder<T>::value` 返回占位符索引（1, 2, …）。

### `std::invoke_result`（C++17） / `result_of`（C++14 前）

```cpp
template<class F, class... ArgTypes>
struct invoke_result;
template<class F, class... ArgTypes>
using invoke_result_t = typename invoke_result<F, ArgTypes...>::type;
```

**作用：** 计算以参数 `ArgTypes...` 调用可调用对象 `F` 的返回类型。

---

## 七、其他实用工具

### `std::not_fn`（C++17）

**函数原型：**
```cpp
template<class F>
/*unspecified*/ not_fn(F&& f);
```

**作用：** 返回一个函数对象，其对参数的调用结果是对 `f` 调用结果的逻辑否定。

**示例：**
```cpp
auto is_even = [](int x) { return x % 2 == 0; };
auto is_odd = std::not_fn(is_even);
```

### `std::mem_fn`（C++11）

**函数原型：**
```cpp
template<class R, class T>
/*unspecified*/ mem_fn(R T::* pm) noexcept;
```

**作用：** 将成员指针包装为可调用对象，可以像普通函数那样调用。

**示例：**
```cpp
struct S { int value; };
std::vector<S> v{{1},{2}};
std::vector<int> res;
std::transform(v.begin(), v.end(), std::back_inserter(res), std::mem_fn(&S::value));
```

---

## 八、宏与常量

`<functional>` 头文件中没有定义任何宏。`std::placeholders` 中占位符的实现是对象，而非宏。

---

## 九、类型定义汇总

| 类型 | 说明 |
|------|------|
| `std::function<Ret(Args...)>` | 通用函数包装器 |
| `std::reference_wrapper<T>` | 引用包装器 |
| `std::hash<Key>` | 哈希函数模板 |
| `std::bad_function_call` | 空 `function` 调用异常 |
| `std::is_bind_expression<T>` | 判断是否为 `bind` 表达式 |
| `std::is_placeholder<T>` | 判断是否为占位符 |
| `std::invoke_result_t<F, Args...>` | 调用结果类型 |
| `std::plus<T>` 等 | 预定义仿函数类模板 |

---

## 十、模板声明

`<functional>` 包含大量模板类和模板函数，上述已覆盖主要部分。C++20 还增加了 `std::bind_front`（简化 `bind`）、`std::not_fn` 的 `constexpr` 支持等。

---
