# 泛型 Lambda（C++14）

**一句话定义**：泛型 Lambda 允许 Lambda 表达式的参数使用 `auto` 关键字，从而使编译器生成的匿名函数对象具有模板化的 `operator()`，能够接受任意类型的参数，极大提升了 Lambda 的复用性和泛型编程能力。

---

## 详细语法与用法

### 基本语法

```cpp
// C++14：参数使用 auto
auto generic_lambda = [](auto a, auto b) { return a + b; };

// 使用示例
int i = generic_lambda(3, 4);              // int + int → int
double d = generic_lambda(3.14, 2.0);      // double + double → double
std::string s = generic_lambda(std::string("Hello"), " World"); // 字符串拼接
```

### 多个参数独立泛型

```cpp
// 参数类型可独立推导
auto mix = [](auto a, auto b) { return a + b; };
int r1 = mix(10, 20.5);        // int + double → double（返回 double）
auto r2 = mix(10, 20.5);       // r2 类型为 double
```

### 结合完美转发

```cpp
auto forwarder = [](auto&&... args) {
    return std::forward<decltype(args)>(args)...;
};

// 或者使用 decltype(auto) 保持精确类型
auto perfect_forward = [](auto&& arg) -> decltype(auto) {
    return std::forward<decltype(arg)>(arg);
};
```

### 使用 `decltype` 和 `std::decay` 处理类型

```cpp
auto get_size = [](const auto& container) -> size_t {
    return container.size();
};

// 更复杂的类型操作
auto as_vector = [](auto&& range) {
    using T = std::decay_t<decltype(*std::begin(range))>;
    return std::vector<T>(std::begin(range), std::end(range));
};
```

### 泛型 Lambda 与 `constexpr`（C++17 自动，C++14 需显式？）

C++14 中 Lambda 不能显式声明 `constexpr`（C++17 允许），但若满足条件可隐式成为 `constexpr`。

```cpp
// C++14：隐式 constexpr（若函数体满足常量表达式要求）
constexpr auto square = [](auto x) { return x * x; };
static_assert(square(5) == 25, "compile-time");
```

### 泛型 Lambda 与可变参数

```cpp
// 接受任意数量、任意类型参数
auto print_all = [](auto&&... args) {
    ((std::cout << std::forward<decltype(args)>(args) << " "), ...);
};
print_all(1, 2.5, "hello", 'c');
```

### 作为模板参数的泛型 Lambda

```cpp
#include <vector>
#include <algorithm>

std::vector<int> v = {1, 3, 2, 5, 4};
std::sort(v.begin(), v.end(), [](auto a, auto b) { return a > b; });
// 降序排序，Lambda 可比较任意类型（要求支持 >）
```

---

## 底层原理与实现机制

### 1. 编译器翻译为模板化仿函数

泛型 Lambda 与普通 Lambda 类似，编译器将其转换为一个匿名的函数对象类。区别在于 `operator()` 变为**模板成员函数**。

```cpp
// 源代码
auto lambda = [](auto x, auto y) { return x + y; };
```

编译器生成类似：

```cpp
class __AnonymousLambda {
public:
    template<typename T1, typename T2>
    auto operator()(T1 x, T2 y) const {
        return x + y;
    }
};
```

每次调用时，编译器根据实际参数类型实例化 `operator()` 的对应版本。

### 2. `auto` 参数的类型推导规则

泛型 Lambda 的 `auto` 参数使用与模板参数推导**完全相同**的规则：
- 若参数声明为 `auto`，则按值传递，推导时去除引用和 cv 限定（如同 `template<typename T> void f(T x)`）。
- 若参数声明为 `auto&`，则推导为左值引用。
- 若参数声明为 `const auto&`，则推导为 const 左值引用。
- 若参数声明为 `auto&&`，则使用万能引用规则（左值→左值引用，右值→右值引用）。

```cpp
auto lambda = [](auto a, auto& b, const auto& c, auto&& d) {
    // a: 按值，去除引用/cv
    // b: 左值引用
    // c: const 左值引用
    // d: 万能引用
};
```

### 3. 与普通模板函数的等价性

```cpp
// 泛型 Lambda
auto lambda = [](auto a, auto b) { return a + b; };

// 等价的手写函数对象
struct {
    template<typename T, typename U>
    auto operator()(T a, U b) const { return a + b; }
} lambda;
```

### 4. 闭包类型与实例化

每个泛型 Lambda 表达式产生一个独特的闭包类型。该类型的 `operator()` 模板为每个不同的参数类型组合生成独立的函数实例。编译器可能内联这些调用，减少运行时开销。

### 5. 与 `std::function` 的交互

泛型 Lambda 不能直接赋值给 `std::function`，因为 `std::function` 需要具体签名。但可通过包装或使用 `auto` 存储。

```cpp
auto gl = [](auto x) { return x; };
// std::function<int(int)> f = gl;   // 错误：不能从泛型 Lambda 转换（类型不唯一）
std::function<int(int)> f = [](int x) { return x; }; // 需显式类型
```

---

## 与相关特性的对比

### 1. 泛型 Lambda vs 普通 Lambda

| 特性 | 普通 Lambda（C++11） | 泛型 Lambda（C++14） |
|------|---------------------|---------------------|
| 参数类型 | 固定类型 | `auto` 推导，模板化 |
| 复用性 | 每类型需单独 Lambda | 一个 Lambda 适用多类型 |
| 生成代码 | 单个 `operator()` | 模板 `operator()`，多实例化 |
| 适用场景 | 类型已知或单一 | 需要处理多种类型（如 STL 算法） |

### 2. 泛型 Lambda vs 函数模板

| 特性 | 泛型 Lambda | 函数模板 |
|------|-------------|----------|
| 定义位置 | 局部作用域（可捕获变量） | 命名空间/类作用域 |
| 捕获能力 | 可捕获周围变量 | 不能捕获（只能通过参数传递） |
| 语法简洁 | 更简洁（无需 `template` 关键字） | 需要 `template<typename T>` |
| 类型重用 | 闭包类型匿名，难以重用 | 函数名可多处调用 |
| 性能 | 相同（均为编译期实例化） | 相同 |

泛型 Lambda 适合在局部需要快速定义泛型操作，而函数模板适合库接口或多次复用的泛型函数。

### 3. 泛型 Lambda vs `std::bind` + 占位符

`std::bind` 可以绑定模板函数，但语法晦涩且类型擦除有开销。泛型 Lambda 更直观高效。

```cpp
// 使用 bind（复杂）
auto add5 = std::bind(std::plus<>(), std::placeholders::_1, 5);
// 使用泛型 Lambda（清晰）
auto add5 = [](auto x) { return x + 5; };
```

---

## 常见面试问题与解答

### 问题 1：泛型 Lambda 的参数 `auto` 与函数模板参数 `typename T` 在类型推导上有何异同？

**答**：完全相同。泛型 Lambda 的 `auto` 参数遵循模板参数推导规则。例如 `auto` 对应按值传递，去除引用和 cv 限定；`auto&` 对应左值引用；`auto&&` 对应万能引用。实际上，编译器将每个 `auto` 参数转换为一个独立的模板类型参数。唯一的区别在于语法：Lambda 中使用 `auto` 更简洁，而函数模板需要显式写出 `template<typename T>`。

---

### 问题 2：泛型 Lambda 是否可以捕获模板参数？如何实现类似模板模板参数的效果？

**答**：泛型 Lambda 不能直接捕获类型（因为类型不是运行时实体），但可以通过 `decltype` 和 `std::decay` 在函数体内获取参数类型并操作。若需要类似“模板模板参数”（即模板接受模板），C++14 泛型 Lambda 无法直接做到，但可结合 `decltype` 和辅助 traits。C++20 的模板 Lambda 显式支持模板参数列表，可以更直接地实现。

```cpp
// C++14 间接模拟：通过参数类型推导
auto make_vector = [](auto&& range) {
    using T = std::decay_t<decltype(*std::begin(range))>;
    return std::vector<T>(std::begin(range), std::end(range));
};

// C++20 显式模板 Lambda
auto make_vec = []<typename T>(const std::vector<T>& v) {
    return std::vector<T>(v.begin(), v.end());
};
```

---

### 问题 3：泛型 Lambda 如何实现完美转发？为什么需要 `decltype`？

**答**：完美转发需要保持参数的值类别（左值/右值）和 cv 限定。在泛型 Lambda 中，参数声明为 `auto&&` 表示万能引用，但转发时需使用 `std::forward`。由于参数名是变量（左值），直接使用 `std::forward` 需要提供模板参数，而 `auto&&` 推导的类型可通过 `decltype(arg)` 获取。

```cpp
auto perfect_forward = [](auto&& arg) -> decltype(auto) {
    return std::forward<decltype(arg)>(arg);
};
```

`decltype(arg)` 对于 `auto&&` 参数能准确返回原始类型（左值→左值引用，右值→右值引用），配合 `std::forward` 实现完美转发。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：泛型 Lambda 中返回类型不一致导致编译错误

```cpp
auto inconsistent = [](auto x) {
    if (x > 0) return x;       // 返回 int（若 x 是 int）
    else return 0.0;           // 返回 double，类型不一致
};  // 错误：无法推导返回类型
```

**解决**：显式指定返回类型 `-> decltype(auto)` 或统一类型。

#### 错误 2：误用 `auto` 导致不必要的拷贝

```cpp
std::vector<std::string> vec = {"a", "bb", "ccc"};
auto lambda = [](auto x) { return x.size(); };  // x 按值拷贝 string
```

**解决**：使用 `const auto&` 避免拷贝。

#### 错误 3：试图存储泛型 Lambda 到 `std::function`

```cpp
std::function<int(int)> f = [](auto x) { return x; }; // 错误：不能推导具体类型
```

**解决**：使用 `auto` 存储，或为特定类型实例化（如 `std::function<int(int)> f = [](int x){return x;}`）。

#### 错误 4：泛型 Lambda 内部调用自身（递归）困难

```cpp
auto factorial = [](auto n) {
    return n == 0 ? 1 : n * factorial(n-1);  // 错误：factorial 未捕获
};
```

**解决**：使用 `std::function` 或 Y 组合子，或 C++14 中通过捕获自身（需要显式类型推导）。

```cpp
auto factorial = [](auto self, auto n) -> int {
    return n == 0 ? 1 : n * self(self, n-1);
};
auto fact = [](int n) { return factorial(factorial, n); };
```

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **默认使用 `const auto&` 参数** | 避免不必要的拷贝，除非需要修改或按值语义 |
| **使用 `auto&&` + `decltype` 实现完美转发** | 保留值类别，用于转发到其他函数 |
| **泛型 Lambda 常用于 STL 算法** | 替代冗长的函数对象或函数模板 |
| **结合 `decltype` 和 `std::decay` 处理类型** | 提取参数类型用于构造其他对象 |
| **避免将泛型 Lambda 存储在 `std::function`** | 使用 `auto` 保持类型，减少开销 |
| **返回类型复杂时使用 `-> decltype(auto)`** | 自动推导返回类型，保持引用性 |

```cpp
// 好的示例：STL 算法中使用泛型 Lambda
std::vector<int> src = {1,2,3};
std::vector<double> dst;
std::transform(src.begin(), src.end(), std::back_inserter(dst),
               [](auto x) { return static_cast<double>(x) / 2; });

// 好的示例：完美转发包装器
auto call = [](auto&& func, auto&&... args) -> decltype(auto) {
    return std::forward<decltype(func)>(func)(std::forward<decltype(args)>(args)...);
};
```

---

## 编译器支持与版本演进

| 标准 | 特性 | 编译器最低版本 |
|------|------|----------------|
| **C++14** | 泛型 Lambda（`auto` 参数） | GCC 5.0+、Clang 3.4+、MSVC 2015+ |
| C++17 | Lambda 显式 `constexpr`，泛型 Lambda 自动 `constexpr` 若可能 | GCC 7+、Clang 5+、MSVC 2017+ |
| C++20 | 模板 Lambda（显式模板参数列表） | GCC 10+、Clang 10+、MSVC 2019 16.10+ |

**特性测试宏**：
```cpp
#ifdef __cpp_generic_lambdas
    // 泛型 Lambda 可用，值为 201304
#endif
```