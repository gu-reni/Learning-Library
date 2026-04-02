# lambda 表达式（C++11，C++14/17/20 增强）

**一句话定义**：lambda 表达式是一种可调用的匿名函数对象，能够就地定义并捕获周围作用域中的变量，极大地提升了 STL 算法、回调函数和异步编程的表达力。

---

## 详细语法与用法

### 基本语法

```cpp
[capture](parameters) -> return_type { body }
```

- `[capture]`：捕获列表，定义 lambda 可以访问的外部变量及方式。
- `(parameters)`：参数列表，与普通函数相同（C++14 起允许 `auto` 参数）。
- `-> return_type`：返回类型（可省略，编译器自动推导）。
- `{ body }`：函数体。

### 捕获方式

| 捕获方式 | 说明 |
|----------|------|
| `[]` | 不捕获任何外部变量 |
| `[=]` | 按值捕获所有自动变量（副本），lambda 内不可修改（除非 `mutable`） |
| `[&]` | 按引用捕获所有自动变量 |
| `[x]` | 按值捕获变量 `x` |
| `[&x]` | 按引用捕获变量 `x` |
| `[=, &x]` | 默认按值捕获，但 `x` 按引用捕获 |
| `[&, x]` | 默认按引用捕获，但 `x` 按值捕获 |
| `[this]` | 捕获当前对象的 `this` 指针（C++11），可访问成员变量 |
| `[*this]` | 按值捕获当前对象（C++17），拷贝整个 `*this` |
| `[&, this]` | C++20 前需要显式写出，C++20 允许隐式捕获 `this` |

### 基础示例

```cpp
// C++11 最简单的 lambda
auto f = []() { std::cout << "Hello\n"; };
f();

// 带参数和返回类型（自动推导）
auto add = [](int a, int b) { return a + b; };
int sum = add(3, 4);  // 7

// 显式指定返回类型（用于复杂表达式）
auto divide = [](double a, double b) -> double {
    if (b == 0) return 0.0;
    return a / b;
};

// 捕获外部变量
int x = 10, y = 20;
auto by_value = [=]() { return x + y; };        // 按值捕获副本
auto by_ref   = [&]() { x++; y++; };            // 按引用捕获
by_ref();
std::cout << x << " " << y;  // 11 21
```

### mutable 关键字

默认情况下，按值捕获的变量在 lambda 内部是**只读**的（相当于 `const`）。使用 `mutable` 可以修改副本。

```cpp
// C++11
int count = 0;
auto inc = [count]() mutable { return ++count; };  // 修改的是副本
std::cout << inc();  // 1
std::cout << inc();  // 2
std::cout << count;  // 0（原变量不变）

// 按引用捕获不需要 mutable
auto inc_ref = [&count]() { return ++count; };
```

### 泛型 lambda（C++14）

```cpp
// C++14：参数使用 auto，成为模板 operator()
auto generic_add = [](auto a, auto b) { return a + b; };
int i = generic_add(3, 4);           // int
double d = generic_add(3.14, 2.0);   // double
std::string s = generic_add(std::string("Hello"), " World");  // 字符串拼接
```

### 模板 lambda（C++20）

```cpp
// C++20：显式模板参数
auto lambda = []<typename T>(T a, T b) {
    return a + b;
};
int r = lambda(3, 4);   // T 推导为 int

// 可配合概念（C++20）
auto addable = []<std::integral T>(T a, T b) {
    return a + b;
};
```

### constexpr lambda（C++17）

```cpp
// C++17：若满足 constexpr 条件，lambda 自动成为 constexpr
constexpr auto square = [](int x) { return x * x; };
constexpr int result = square(5);  // 编译期计算

// 强制要求 constexpr（C++17 可显式指定）
auto cube = [](int x) constexpr { return x * x * x; };
static_assert(cube(3) == 27);
```

### 捕获 `*this`（C++17）

```cpp
class Widget {
    int value = 42;
public:
    auto getLambda() {
        // C++11/14：捕获 this 指针，风险：对象析构后 lambda 可能悬垂
        auto l1 = [this]() { return value; };
        
        // C++17：按值捕获整个对象，安全
        auto l2 = [*this]() { return value; };  // 拷贝 *this
        return l2;
    }
};
auto lam = Widget().getLambda();  // Widget 已销毁，但 lambda 持有其副本，安全
```

### lambda 作为函数返回值

```cpp
// 使用 std::function 包装（有运行时开销）
std::function<int(int)> get_multiplier(int factor) {
    return [factor](int x) { return x * factor; };
}

// 使用 auto + decltype（C++14 更好，但注意 lambda 类型不可写）
auto get_multiplier_v2(int factor) {
    return [factor](int x) { return x * factor; };  // 返回类型推导为匿名类
}
```

---

## 底层原理与实现机制

### 编译器翻译为仿函数类

lambda 表达式在编译时被转换为一个**匿名的函数对象（仿函数）类**，并实例化一个该类的对象。

```cpp
// 源代码
int x = 10;
auto lambda = [x](int y) mutable { return x + y; };
int z = lambda(5);
```

编译器大致生成如下代码（简化）：

```cpp
class __Lambda_123 {
private:
    int __x;                     // 按值捕获的成员变量
public:
    __Lambda_123(int x) : __x(x) {}
    
    auto operator()(int y) /* 非 const，因 mutable */ {
        return __x + y;
    }
};

int x = 10;
__Lambda_123 lambda(x);   // 构造 lambda 对象
int z = lambda.operator()(5);
```

- **按值捕获**：成员变量与捕获变量的类型相同，在 lambda 对象构造时拷贝。
- **按引用捕获**：成员变量是引用类型（`T&`），存储的是引用，不拷贝对象。
- **`mutable` 影响**：无 `mutable` 时，`operator()` 被声明为 `const`，因此不能修改按值捕获的成员；有 `mutable` 则 `operator()` 非 `const`。

### 捕获方式对内存布局的影响

| 捕获列表 | 生成的成员变量 | 对象大小 |
|----------|---------------|----------|
| `[]` | 无 | 1 字节（占位） |
| `[x, y]` (int) | `int x; int y;` | 8 字节 |
| `[&x, &y]` | `int& x; int& y;` | 引用大小（通常 8 字节/个） |
| `[=]` | 所有用到的自动变量按值 | 各变量大小之和 |
| `[&]` | 所有用到的自动变量按引用 | 引用大小之和 |
| `[this]` | `T* const this` | 指针大小 |

注意：未使用的变量即使被 `[=]` 或 `[&]` 捕获，编译器也会优化掉，不增加成员。

### 泛型 lambda 的实现

`auto` 参数被转换为模板 `operator()`：

```cpp
// 源代码
auto gen = [](auto a, auto b) { return a + b; };
```

编译器生成：

```cpp
class __GenLambda {
public:
    template <typename T1, typename T2>
    auto operator()(T1 a, T2 b) const {
        return a + b;
    }
};
```

每次调用可能实例化不同的 `operator()` 版本。

### 无捕获 lambda 可转换为函数指针

如果 lambda 不捕获任何变量（`[]`），则它有一个**隐式转换运算符**，转换为相同签名的函数指针。

```cpp
int (*func_ptr)(int, int) = [](int a, int b) { return a + b; };
```

底层原理：编译器为无捕获 lambda 生成一个静态的普通函数（或转换函数），lambda 对象转换为该函数指针。

### 生命周期与悬垂引用

- **按引用捕获**：lambda 对象存储的是外部变量的引用。若 lambda 的生命周期超过了被引用变量的生命周期，则引用悬垂，使用会导致未定义行为。
- **按值捕获**：拷贝副本，无悬垂风险（但注意 `this` 指针仍可能悬垂，C++17 的 `[*this]` 可解决）。

```cpp
auto bad() {
    int local = 42;
    return [&local] { return local; };  // 返回的 lambda 持有 local 的引用，local 已销毁
}
auto lam = bad();
std::cout << lam();  // 未定义行为
```

---

## 与相关特性的对比

### lambda vs std::function

| 特性 | lambda | `std::function` |
|------|--------|-----------------|
| **类型** | 每个 lambda 有独特的匿名类型 | 类型擦除，统一为 `std::function<Signature>` |
| **存储** | 可栈上分配，无额外堆开销（除非捕获大量变量） | 可能堆分配存储可调用对象 |
| **性能** | 零开销（直接调用） | 有间接调用和类型擦除开销 |
| **赋值** | 只能移动（捕获后），不可拷贝（除非捕获可拷贝） | 可拷贝，可赋值 |
| **适用场景** | 短小、局部使用、类型已知 | 需要存储多种可调用对象、多态回调 |

```cpp
// lambda 零开销
auto l = [](int x) { return x * 2; };
int r1 = l(5);  // 直接调用

// std::function 有开销
std::function<int(int)> f = l;
int r2 = f(5);  // 虚函数或间接调用
```

### lambda vs 手写仿函数类

| 方面 | lambda | 手写仿函数类 |
|------|--------|-------------|
| **代码量** | 极少 | 需要定义类、成员变量、构造函数、`operator()` |
| **可读性** | 就地定义，上下文清晰 | 需要跳转到类定义处 |
| **捕获灵活性** | 自动捕获，按值/按引用任意组合 | 手动定义成员，初始化复杂 |
| **重复使用** | 通常单次使用 | 可多次实例化 |
| **调试** | 编译器生成名称，调试器显示 `__lambda` 等 | 自定义类名，调试友好 |

一般而言，除非 lambda 需要在多个地方重复使用且逻辑复杂，否则 lambda 更优。

### lambda 与 std::bind

`std::bind` 是 C++11 之前的绑定工具，lambda 更清晰、性能更好。

```cpp
// 使用 bind（复杂）
auto add = std::bind([](int a, int b) { return a + b; }, std::placeholders::_1, 5);
int r = add(3);  // 8

// 使用 lambda（直观）
auto add_lambda = [](int x) { return x + 5; };
```

lambda 的优势：可读性、编译期类型检查、内联友好、无 `std::placeholders` 的晦涩。

---

## 常见面试问题与解答

### 问题 1：lambda 按值捕获的变量是否总是只读？如何修改？

**答**：默认情况下，按值捕获的变量在 lambda 体内是**只读**的，因为 lambda 生成的 `operator()` 是 `const` 成员函数。如果需要修改副本，必须使用 `mutable` 关键字。

```cpp
int x = 10;
auto lam1 = [x]() { x++; };         // 错误：不能修改 const 成员
auto lam2 = [x]() mutable { x++; }; // 正确：修改的是副本
```

注意：按引用捕获的变量始终可以修改（除非原变量本身是 `const`），不需要 `mutable`。

---

### 问题 2：lambda 的大小（sizeof）是多少？与捕获什么有关？

**答**：`sizeof(lambda)` 等于其所有捕获成员的大小之和，加上可能的对齐填充。无捕获的 lambda 大小为 **1 字节**（C++ 要求对象大小至少为 1）。

```cpp
int a = 1, b = 2;
auto l0 = []{};                    // sizeof(l0) == 1
auto l1 = [a]{};                   // sizeof(l1) == sizeof(int) == 4
auto l2 = [&a]{};                  // sizeof(l2) == sizeof(int*) == 8（64位）
auto l3 = [a, &b]{};               // 4 + 8 = 12 字节
auto l4 = [=]{};                   // 若未使用 a,b，编译器优化为 1 字节
```

---

### 问题 3：泛型 lambda 的 `auto` 参数与模板 lambda 有什么区别？

**答**：

- **泛型 lambda**（C++14）：参数类型用 `auto`，编译器生成模板化的 `operator()`，但无法显式指定模板参数或使用模板特化。
- **模板 lambda**（C++20）：允许显式写出模板参数列表，可以使用概念约束、模板特化、`decltype` 等。

```cpp
// 泛型 lambda (C++14)
auto gl = [](auto x, auto y) { return x + y; };

// 模板 lambda (C++20)
auto tl = []<typename T>(T x, T y) { return x + y; };

// 模板 lambda 可加概念
auto cl = []<std::integral T>(T x, T y) { return x + y; };
```

模板 lambda 提供了更多控制，但代码略显冗长。泛型 lambda 足够多数场景。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：返回持有引用捕获的 lambda

```cpp
auto make_lambda() {
    int local = 42;
    return [&] { return local; };  // 危险：local 被销毁
}
// 调用返回的 lambda 导致未定义行为
```

**正确做法**：按值捕获 `[=]` 或捕获具体值 `[local]`，或者确保 lambda 生命周期短于被引用变量。

---

#### 错误 2：在循环中捕获引用导致所有 lambda 引用同一变量

```cpp
std::vector<std::function<void()>> funcs;
for (int i = 0; i < 5; ++i) {
    funcs.push_back([&] { std::cout << i; });  // 所有 lambda 捕获同一 i 的引用
}
for (auto& f : funcs) f();  // 输出可能是 55555 或未定义（i 已失效）
```

**正确做法**：按值捕获 `[i]` 或创建副本 `[=]`。

---

#### 错误 3：不必要的按值捕获大对象

```cpp
std::vector<int> huge_vec(1000000);
auto lam = [huge_vec] { ... };  // 拷贝大 vector，开销巨大
```

**正确做法**：按引用捕获 `[&huge_vec]`，或移动捕获（C++14 中使用初始化捕获）。

```cpp
// C++14 移动捕获
auto lam = [vec = std::move(huge_vec)] { ... };  // 移动而非拷贝
```

---

#### 错误 4：误用 mutable 导致意外行为

```cpp
int x = 5;
auto lam = [x]() mutable { x++; return x; };
std::cout << lam();  // 6
std::cout << lam();  // 7
std::cout << x;      // 5（原变量不变）
```

如果不理解副本语义，可能错误认为原变量被修改。

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **默认使用按值捕获** | 除非确实需要修改外部变量或追求性能（避免拷贝），优先 `[=]` 或显式值捕获，避免悬垂引用 |
| **捕获列表最小化** | 只捕获需要的变量，提高可读性，可能减少对象大小（编译器会优化未使用变量） |
| **避免默认捕获 `[&]` 和 `[=]` 混用** | 显式列出捕获变量，清晰表达意图，减少意外捕获 |
| **使用初始化捕获（C++14）实现移动** | `[p = std::move(ptr)]` 将 `std::unique_ptr` 移入 lambda |
| **无捕获 lambda 优先转换为函数指针** | 需要函数指针回调时，使用 `[]{}` 避免 `std::function` 开销 |
| **长 lambda 或复用 lambda 可命名** | 提高代码可读性，避免重复定义 |
| **谨慎在 lambda 中捕获 `this`** | 确保对象生命周期长于 lambda；C++17 可使用 `[*this]` 安全拷贝 |

```cpp
// 好的示例：初始化捕获实现移动语义（C++14）
std::unique_ptr<int> uptr = std::make_unique<int>(10);
auto task = [uptr = std::move(uptr)] {
    return *uptr;  // uptr 所有权转移进 lambda
};

// 好的示例：显式捕获，避免默认
int a, b, c;
auto lam = [a, &b] { /* 只使用 a 和 b */ };  // 清晰
```