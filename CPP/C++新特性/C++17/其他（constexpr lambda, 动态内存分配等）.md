您提到的“constexpr lambda”和“动态内存分配”在之前的总结中可能没有被充分覆盖（constexpr lambda 在 C++17 语言特性中已提及，但 C++20 对其有重大增强；动态内存分配在 `constexpr` 上下文中是 C++20 的新特性）。此外，还有一些其他重要的 C++17/20 特性未被之前的专题文档涵盖。

以下是对这些“遗漏”特性的补充说明：

---

## 1. `constexpr` Lambda（C++17 引入，C++20 增强）

**C++17**：Lambda 表达式在满足 `constexpr` 函数条件时隐式成为 `constexpr`，也可以显式使用 `constexpr` 关键字。

**C++20 增强**：
- 允许在 `constexpr` lambda 中使用 `constexpr` 变量捕获（C++17 中捕获的变量必须在常量表达式中可用）。
- 允许在 `constexpr` lambda 中使用虚函数（C++20 的 `constexpr` 虚函数支持）。
- 允许在 `constexpr` lambda 中使用动态分配（C++20 的 `constexpr` 动态分配支持）。

```cpp
// C++20：constexpr lambda 中使用动态分配
constexpr auto lambda = []() constexpr {
    int* p = new int(42);
    int res = *p;
    delete p;
    return res;
};
static_assert(lambda() == 42);
```

---

## 2. `constexpr` 动态内存分配（C++20）

**一句话定义**：C++20 允许在 `constexpr` 函数（包括构造函数、析构函数）中使用 `new` 和 `delete`，前提是分配和释放都在编译期完成，且满足常量表达式的要求。

**核心语法**：

```cpp
constexpr std::vector<int> make_vector() {
    std::vector<int> v;            // C++20 vector 的 constexpr 构造函数
    v.push_back(1);                // push_back 也是 constexpr
    v.push_back(2);
    return v;                      // 析构也是 constexpr
}

constexpr auto vec = make_vector();  // 编译期构造 vector
static_assert(vec.size() == 2);
```

**底层原理**：编译器在常量求值时维护一个“堆内存”模拟（如静态分配池），所有 `new` 分配的内存必须在同一常量求值过程中 `delete`，否则编译错误。

**限制**：
- 不能抛出异常。
- 不能使用 `std::allocator`（除非实现标记为 `constexpr`，C++20 标准库已支持）。
- 不能将动态分配的内存返回给运行时（即不能返回指向动态分配内存的指针）。

---

## 3. `consteval` 立即函数（C++20）

**一句话定义**：`consteval` 修饰的函数必须在编译期求值，不能产生运行时调用；若无法在编译期求值则编译错误。

**核心语法**：

```cpp
consteval int square(int x) { return x * x; }
constexpr int c = square(5);   // OK
int x = 10;
// int y = square(x);          // 错误：x 不是常量表达式

consteval int only_compile_time(int x) {
    return x + x;
}
```

**与 `constexpr` 的区别**：
- `constexpr` 函数可能在编译期或运行时求值。
- `consteval` 函数**必须**在编译期求值，因此只能接收常量表达式参数。

---

## 4. `constinit` 常量初始化（C++20）

**一句话定义**：`constinit` 用于强制全局或静态变量在静态初始化阶段（编译期或常量初始化）完成初始化，避免静态初始化顺序问题。

**核心语法**：

```cpp
constinit int global = 42;           // OK，常量初始化
constinit int runtime = get_rand();   // 错误：get_rand() 不是常量表达式

struct A { int x; };
constinit A a{1};                     // 聚合初始化也是常量初始化

// 常见用法：防止动态初始化顺序问题
extern constinit int* p;
```

**与 `constexpr` 的区别**：
- `constexpr` 变量隐含 `const`，且必须在编译期初始化。
- `constinit` 变量**不是** `const`，可以在运行时修改，但初始化必须是编译期常量。

---

## 5. 类模板参数推导（CTAD, C++17）

**一句话定义**：类模板的构造函数参数可以自动推导模板参数，无需显式指定。

**核心语法**：

```cpp
std::pair p{1, 3.14};                 // std::pair<int, double>
std::vector v{1, 2, 3};               // std::vector<int>
std::mutex m;
std::lock_guard lg(m);                // std::lock_guard<std::mutex>

// 自定义类模板
template<typename T>
struct Box {
    Box(T val) : value(val) {}
    T value;
};
Box b(42);                            // Box<int>

// 提供推导指引（deduction guide）
template<typename T>
Box(T) -> Box<T>;
```

**底层原理**：编译器根据构造函数的参数类型推导模板参数，用户可自定义推导指引（类似 `std::pair` 的处理）。

---

## 6. 模板参数中的 `auto`（C++17）

**一句话定义**：非类型模板参数可以使用 `auto` 占位符，自动推导类型。

**核心语法**：

```cpp
template<auto N>
struct S { static constexpr auto value = N; };

S<42> s1;               // N 为 int
S<'c'> s2;              // N 为 char
S<&global_var> s3;      // N 为 int*（全局变量的地址）

// 结合变参模板
template<auto... Values>
struct Tuple { /* ... */ };
Tuple<1, 'a', 3.14> t;
```

**底层原理**：`auto` 在非类型模板参数中的推导规则与变量模板类似，允许更灵活的编译期常量。

---

## 7. 设计初始化器（Designated Initializers, C++20）

**一句话定义**：允许使用成员名进行聚合初始化（类似 C 语法），提高可读性。

**核心语法**：

```cpp
struct Point {
    int x;
    int y;
    int z = 0;   // 默认值
};

Point p = { .x = 10, .y = 20 };   // z 被初始化为 0

// 注意：必须按声明顺序，不能跳跃
struct A { int a; int b; };
// A a = { .b = 2, .a = 1 };      // 错误：顺序错误
```

**限制**：只能用于聚合体，且初始化列表中的名称顺序必须与成员声明顺序一致。

---

## 8. 三路比较运算符（`<=>`, C++20）

**一句话定义**：`operator<=>` 生成强/弱/部分序关系，自动生成所有比较运算符（`==`、`<`、`>` 等）。

**核心语法**：

```cpp
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;  // 自动生成所有比较
};

Point p1{1, 2}, p2{1, 3};
bool r = (p1 < p2);    // 自动生成，比较 x 先，再比较 y

// 自定义序
struct CaseInsensitiveString {
    std::string s;
    std::weak_ordering operator<=>(const CaseInsensitiveString& other) const {
        // 不区分大小写比较
        return case_insensitive_compare(s, other.s);
    }
};
```

**底层原理**：编译器根据成员顺序生成字典序比较，返回 `std::strong_ordering`、`std::weak_ordering` 或 `std::partial_ordering`。

---

## 9. 协程（Coroutines, C++20）

**一句话定义**：协程是可暂停和恢复的函数，通过 `co_await`、`co_yield`、`co_return` 控制，用于异步编程、生成器、惰性求值等。

**核心语法**：

```cpp
// 生成器示例
std::generator<int> fibonacci() {
    int a = 0, b = 1;
    while (true) {
        co_yield a;
        auto next = a + b;
        a = b;
        b = next;
    }
}

// 异步任务
std::task<int> async_compute() {
    int x = co_await async_read();
    int y = co_await async_process(x);
    co_return y;
}
```

**底层原理**：编译器将协程转换为状态机，分配协程帧存储局部变量和暂停点。需要 `<coroutine>` 头文件和 `std::coroutine_handle`。

---

## 10. 概念（Concepts, C++20）

**一句话定义**：概念是模板参数的编译期谓词，用于约束模板参数，替代 SFINAE，提供更清晰的错误信息。

**核心语法**：

```cpp
template<typename T>
concept Integral = std::is_integral_v<T>;

template<Integral T>
T add(T a, T b) { return a + b; }

// 或者使用简写语法
auto multiply(Integral auto a, Integral auto b) { return a * b; }

// requires 子句
template<typename T>
requires std::copyable<T>
void process(T t) { /* ... */ }
```

**底层原理**：概念被编译为常量表达式，编译器在模板实例化时检查约束，失败时输出用户友好的错误信息。

---

## 总结

| 特性 | 标准 | 核心价值 |
|------|------|----------|
| `constexpr` lambda 增强 | C++20 | 编译期函数对象支持动态分配、虚函数 |
| `constexpr` 动态分配 | C++20 | 编译期使用 `new`/`delete` |
| `consteval` | C++20 | 强制编译期求值 |
| `constinit` | C++20 | 强制静态初始化，避免顺序问题 |
| 类模板参数推导（CTAD） | C++17 | 减少模板参数显式指定 |
| 非类型模板参数 `auto` | C++17 | 更灵活的编译期常量 |
| 设计初始化器 | C++20 | 聚合体命名成员初始化 |
| 三路比较 `<=>` | C++20 | 自动生成比较运算符 |
| 协程 | C++20 | 异步编程、生成器 |
| 概念 | C++20 | 模板约束，改善错误信息 |

以上特性是 C++17/20 中常被遗漏但面试或现代 C++ 开发中重要的内容。如需针对某个特性生成详细的面试文档（参照之前的标准格式），请告知具体名称。