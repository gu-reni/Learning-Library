# C++11 其他核心语言特性（auto、范围 for、override/final、default/delete、委托/继承构造函数、static_assert 等）

本文档涵盖 C++11 中除右值引用、lambda、constexpr、智能指针等专题外的重要语言特性，每个特性独立成节，保持深度与实用性。

---

## 1. auto – 类型自动推导

**一句话定义**：`auto` 让编译器根据初始化表达式自动推导变量类型，简化冗长类型名书写，尤其适用于模板和迭代器。

### 详细语法与用法

```cpp
auto x = 42;               // int
auto y = 3.14;             // double
auto p = &x;               // int*
auto it = vec.begin();     // std::vector<int>::iterator

// const/引用需要显式标注
const auto& cr = x;        // const int&
auto&& ur = x;             // 万能引用，推导为 int&（因为 x 是左值）

// C++14 起 auto 可用于函数返回类型推导
auto add(int a, int b) { return a + b; }

// 多个变量声明，类型必须一致
auto a = 10, b = 20;       // OK
// auto c = 10, d = 3.14;  // 错误：类型不一致
```

### 底层原理与实现机制

`auto` 使用与模板参数推导**完全相同**的规则：
- 去除顶层 `const` 和 `volatile`
- 去除引用（除非显式声明 `auto&`）
- 数组退化为指针，函数退化为函数指针

```cpp
const int x = 10;
auto a = x;          // int（顶层 const 去除）
const auto b = x;    // const int（显式保留）

int& ref = x;
auto c = ref;        // int（引用被去除）
auto& d = ref;       // int&（显式引用）
```

特殊规则：`auto` 用于 `initializer_list` 时，`auto a = {1,2,3};` 推导为 `std::initializer_list<int>`（C++11/14），C++17 起有变化（见常见面试题）。

### 与相关特性的对比

| 特性 | `auto` | `decltype` | 模板参数推导 |
|------|--------|------------|--------------|
| 去除顶层 const/引用 | ✅ | ❌（保留精确类型） | ✅（同 auto） |
| 保留引用性 | 需显式 `auto&` | 是（表达式依赖） | 需显式 `T&` |
| 适用场景 | 变量初始化 | 返回类型、转发 | 泛型参数 |

### 常见面试问题与解答

**问题 1：`auto` 与 `decltype(auto)`（C++14）有什么区别？**

`auto` 会去除引用和顶层 const，而 `decltype(auto)` 保留表达式的精确类型（包括引用）。例如：
```cpp
int x = 10;
int& get() { return x; }
auto a = get();          // int
decltype(auto) b = get(); // int&
```
`decltype(auto)` 常用于完美转发函数返回值。

**问题 2：C++17 中 `auto a{1}` 和 `auto a = {1}` 有什么区别？**

C++17 起：
- `auto a = {1}` → `std::initializer_list<int>`
- `auto a{1}` → `int`（直接列表初始化，单元素推导为元素类型）
- `auto a{1,2}` → 错误（直接列表初始化多元素非法）

此修改让 `auto` 直接初始化的行为更符合直觉。

### 典型错误与最佳实践

```cpp
// ❌ 错误：auto 不能用于函数参数（C++11/14，C++20 允许 auto 参数即泛型 lambda）
void func(auto x) {}   // C++11 错误，C++20 允许（简写模板）

// ❌ 错误：auto 无法推导初始化列表（除非 = 形式）
auto v = {1, 2, 3};    // OK：initializer_list
// auto v2{1, 2, 3};   // C++11/14 OK（initializer_list），C++17 错误

// ✅ 最佳实践
// 1. 类型明显时使用 auto
auto i = 0;            // 好：int 明显
auto it = map.begin(); // 好：避免冗长类型

// 2. 需要拷贝时用 auto，需要引用时用 auto&
for (auto& x : vec) x *= 2;

// 3. 避免在 API 边界使用 auto 返回非显然类型
```

---

## 2. 范围 for 循环（Range-based for loop）

**一句话定义**：提供简洁语法遍历容器或数组的所有元素，自动处理迭代器边界，减少迭代器显式操作。

### 详细语法与用法

```cpp
std::vector<int> v = {1, 2, 3, 4};
// 按值遍历（拷贝）
for (int x : v) std::cout << x;

// 按引用遍历（可修改）
for (int& x : v) x *= 2;

// const 引用（只读且避免拷贝）
for (const auto& x : v) std::cout << x;

// C++17 起，可以配合结构化绑定
std::map<int, std::string> m = {{1,"a"}, {2,"b"}};
for (const auto& [key, val] : m) { ... }

// 也可用于初始化列表
for (int x : {1,2,3,4}) { ... }
```

### 底层原理与实现机制

范围 for 循环等价于以下代码（C++11 标准描述）：

```cpp
{
    auto && __range = range_expression;
    for (auto __begin = begin_expr, __end = end_expr;
         __begin != __end; ++__begin) {
        declaration = *__begin;
        loop_statement;
    }
}
```

- `begin_expr` 和 `end_expr` 的查找依赖于 ADL：若 `__range` 是数组，则使用指针运算；若类有 `begin()`/`end()` 成员，则调用之；否则调用非成员 `begin`/`end`（C++11 起 `std::begin`/`std::end` 被 ADL 找到）。
- 范围 for 对数组安全：自动推导数组大小。
- 循环变量声明为 `auto&&` 引用，保证临时对象的生命周期（如函数返回的容器）。

### 与相关特性的对比

| 方式 | 代码量 | 性能 | 可修改元素 | 适用场景 |
|------|--------|------|------------|----------|
| 范围 for | 最少 | 与迭代器相同 | 通过引用 | 遍历全部元素 |
| 传统 for + 下标 | 多 | 相同 | 是 | 需要索引时 |
| 迭代器 for | 冗长 | 相同 | 是 | 需要细粒度控制（如 erase） |

### 常见面试问题与解答

**问题 1：范围 for 循环能否在遍历时修改容器大小（如删除元素）？**

不能。修改容器大小（如 `erase`、`push_back`）会使迭代器失效，导致未定义行为。若需删除，应使用传统迭代器循环并小心处理失效。

**问题 2：如何让自己的类型支持范围 for 循环？**

提供 `begin()` 和 `end()` 成员函数（或非成员函数），返回迭代器。例如：

```cpp
class MyRange {
    int* data;
    size_t len;
public:
    int* begin() { return data; }
    int* end()   { return data + len; }
};
```

### 典型错误与最佳实践

```cpp
// ❌ 错误：遍历时修改容器大小
std::vector<int> v = {1,2,3};
for (int x : v) {
    if (x == 2) v.push_back(4); // 迭代器失效，UB
}

// ❌ 错误：不必要的拷贝（大对象）
for (auto x : vec_of_strings) { ... } // 每次拷贝 string

// ✅ 正确：const 引用避免拷贝
for (const auto& s : vec_of_strings) { ... }

// ✅ 最佳实践：需要修改元素时使用引用
for (auto& x : vec) x = process(x);
```

---

## 3. `override` 与 `final` 说明符

**一句话定义**：`override` 显式声明成员函数覆盖基类虚函数，让编译器检查签名匹配；`final` 禁止虚函数被进一步覆盖或禁止类被继承。

### 详细语法与用法

```cpp
struct Base {
    virtual void foo(int);
    virtual void bar() const;
    virtual void baz() final;   // 该函数不能被覆盖
};

struct Derived : Base {
    void foo(int) override;       // OK：正确覆盖
    // void foo(double) override; // 错误：签名不匹配
    void bar() const override;    // OK：const 匹配
    // void baz() override;       // 错误：Base::baz 是 final
};

struct Final final { };           // 该类不能被继承
// struct Bad : Final {};         // 错误
```

### 底层原理与实现机制

- `override` 在编译期检查：编译器查找基类中是否存在签名完全相同的虚函数（包括 const、参数类型、返回类型协变）。若无，则报错，避免因拼写错误或签名变更而意外创建新虚函数。
- `final` 作用于虚函数时，编译器禁止派生类覆盖该函数，可优化虚调用（devirtualization），因为最终覆盖者已确定。
- `final` 作用于类时，禁止其他类继承，编译器可简化虚表布局。

### 与相关特性的对比

| 特性 | 作用 | 编译期检查 | 优化潜力 |
|------|------|------------|----------|
| `override` | 检查正确覆盖 | ✅ | 无 |
| `final` (函数) | 禁止覆盖 | ✅ | 有（去虚拟化） |
| `final` (类) | 禁止继承 | ✅ | 有（虚调用优化） |
| 无说明符 | 无检查 | ❌ | 无 |

### 常见面试问题与解答

**问题 1：如果不写 `override`，但函数签名正确，会怎样？**

正确覆盖仍然发生，但无编译检查。若基类函数签名后来被修改（如参数类型变化），派生类函数将变成独立的新虚函数，造成意外行为。因此总是建议使用 `override`。

**问题 2：`final` 函数是否仍然可以是虚函数？**

是的，`final` 函数仍然是虚函数，但不能再被派生类覆盖。它仍有虚表条目，但编译器可优化。

### 典型错误与最佳实践

```cpp
// ❌ 错误：const 不匹配
struct Base { virtual void f(); };
struct Derived : Base { void f() const override; }; // 错误：不是覆盖

// ❌ 错误：参数类型不同
struct Derived2 : Base { void f(int) override; };   // 错误

// ✅ 最佳实践：所有覆盖基类虚函数的函数都加 override
// ✅ 最佳实践：设计类层次时，将不应被覆盖的函数标记 final
// ✅ 最佳实践：不希望被继承的类标记 final
```

---

## 4. `= default` 与 `= delete` – 特殊成员函数控制

**一句话定义**：`= default` 要求编译器生成默认实现（如默认构造函数、拷贝/移动操作）；`= delete` 禁止使用某函数，常用于禁用拷贝或重载。

### 详细语法与用法

```cpp
class Widget {
public:
    Widget() = default;                    // 默认构造函数
    Widget(const Widget&) = delete;        // 禁止拷贝构造
    Widget& operator=(const Widget&) = delete;
    ~Widget() = default;                   // 默认析构函数
    
    // 允许移动，但不允许拷贝
    Widget(Widget&&) = default;
    Widget& operator=(Widget&&) = default;
};

// 删除普通函数重载
void process(int) {}
void process(double) = delete;             // 禁止 double 参数

// 删除特定模板特化
template<typename T>
void f(T) {}
void f<int>(int) = delete;                 // 禁止 int 版本
```

### 底层原理与实现机制

- `= default` 在类内定义时，编译器生成的函数为 inline；类外定义时可放在 .cpp 文件减少编译依赖。
- 编译器生成的默认实现是 **trivial**（如简单拷贝）时，该函数具有特殊性质（可被 memcpy 等）。
- `= delete` 的函数仍参与重载决议，但调用时产生编译错误。相比将函数声明为 private 而不定义，`= delete` 更早（编译期）且错误信息清晰。

### 与相关特性的对比

| 方式 | 效果 | 编译器生成 | 可显式指定位置 |
|------|------|------------|----------------|
| `= default` | 生成默认实现 | 是 | 类内或类外 |
| 隐式生成 | 同 `= default` | 是 | 仅当未用户声明时 |
| `= delete` | 禁用函数 | 否 | 任何地方 |
| private 未定义 | 链接期错误 | 否 | 类内声明 |

### 常见面试问题与解答

**问题 1：`= default` 和隐式生成的默认构造函数有什么区别？**

当类有其他构造函数时，隐式默认构造函数不会被生成，但 `= default` 可以强制生成。此外，`= default` 可以在类外定义，使默认实现不是内联。

**问题 2：如何实现一个不可拷贝但可移动的类？**

```cpp
class Movable {
public:
    Movable() = default;
    Movable(const Movable&) = delete;
    Movable& operator=(const Movable&) = delete;
    Movable(Movable&&) = default;
    Movable& operator=(Movable&&) = default;
};
```

### 典型错误与最佳实践

```cpp
// ❌ 错误：delete 后仍然调用
void f(int) = delete;
f(10);  // 编译错误

// ❌ 错误：default 与用户定义冲突
class Bad {
public:
    Bad() = default;
    Bad(int) {}   // 此时默认构造函数仍存在（因为 =default），但初始化列表？实际上没问题
    // 但注意：一旦声明了任何构造函数，默认构造函数不会隐式生成，但 =default 可强制生成。
};

// ✅ 最佳实践：使用 = delete 替代 private 未定义（用于禁用拷贝）
// ✅ 最佳实践：使用 = default 简化空构造函数/析构函数
```

---

## 5. 委托构造函数（Delegating Constructor）

**一句话定义**：一个构造函数可以调用同一类的另一个构造函数，避免重复初始化代码。

### 详细语法与用法

```cpp
class Point {
    int x, y;
public:
    Point() : Point(0, 0) {}           // 委托给双参构造函数
    Point(int a) : Point(a, a) {}      // 委托
    Point(int a, int b) : x(a), y(b) {}
};

// 不能形成循环
// Point(int a) : Point(a, 0) {}       // 如果双参又委托回单参，则循环错误
```

### 底层原理与实现机制

编译器将委托构造展开为：首先执行被委托构造函数的**初始化列表**和**函数体**，然后执行当前构造函数的函数体（若有）。注意：当前构造函数的初始化列表不能再初始化任何成员（因为成员已被被委托构造函数初始化）。这是 C++11 规定的语法限制。

### 常见面试问题与解答

**问题：委托构造函数能否在初始化列表中同时初始化其他成员？**

不能。一旦委托，当前构造函数的初始化列表必须为空（除了可能的虚基类，但罕见）。所有成员初始化都在被委托构造函数中完成。

```cpp
Point(int a, int b) : x(a), y(b) {}
Point(int a) : Point(a, a), y(a) {}   // 错误：不能额外初始化 y
```

### 典型错误与最佳实践

```cpp
// ❌ 错误：循环委托
class Circle {
public:
    Circle() : Circle(0) {}
    Circle(int r) : Circle() {}       // 循环，未定义行为（通常编译错误）
};

// ✅ 最佳实践：将通用初始化逻辑集中到一个“主构造函数”，其他构造函数委托给它
```

---

## 6. 继承构造函数（Inheriting Constructor）

**一句话定义**：派生类通过 `using Base::Base` 继承基类的所有构造函数（除默认、拷贝、移动外），简化派生类构造代码。

### 详细语法与用法

```cpp
struct Base {
    Base(int x) {}
    Base(double d) {}
    Base(int, double) {}
};

struct Derived : Base {
    using Base::Base;   // 继承所有 Base 的构造函数（不包括拷贝/移动）
    Derived(int x, int y) : Base(x) {} // 自定义构造函数
};

Derived d1(42);        // 调用 Base(int)
Derived d2(3.14);      // 调用 Base(double)
Derived d3(1, 2.5);    // 调用 Base(int, double)
```

### 底层原理与实现机制

编译器为每个继承的构造函数生成一个派生类版本的构造函数，其参数完美转发给基类构造函数。派生类新增成员会被**默认初始化**（若没有类内初始化器）。注意：继承构造函数不会继承默认、拷贝、移动构造函数。

### 常见面试问题与解答

**问题：继承构造函数能否初始化派生类新增的成员？**

不能直接初始化。新增成员会执行默认初始化（对于内置类型，若未在类内初始化，则值不确定）。如果需要初始化新增成员，应自定义构造函数。

```cpp
struct Derived : Base {
    int extra;
    using Base::Base;
    Derived(int x) : Base(x), extra(42) {} // 自定义，覆盖继承版本
};
```

### 典型错误与最佳实践

```cpp
// ❌ 错误：基类构造函数私有，派生类无法使用
class Base { Base(int) {} }; // 私有
class Derived : Base { using Base::Base; }; // 错误：无法访问

// ✅ 最佳实践：当派生类没有新增成员或新增成员有默认初始化器时，使用继承构造函数简化代码
```

---

## 7. `static_assert` – 编译期断言

**一句话定义**：在编译期检查常量表达式，若为 false 则产生编译错误，用于验证模板参数、平台假设等。

### 详细语法与用法

```cpp
static_assert(sizeof(int) >= 4, "int must be at least 4 bytes");
static_assert(std::is_integral<T>::value, "T must be integral"); // C++11
static_assert(std::is_integral_v<T>, "T must be integral");      // C++17

// 无消息版本（C++17）
static_assert(true);
```

### 底层原理与实现机制

编译器在编译期求值表达式，若结果为 false，则停止编译并输出错误信息（包括自定义字符串）。`static_assert` 不产生任何运行时代码。

### 典型错误与最佳实践

```cpp
// ❌ 错误：表达式不是常量
int x = 10;
static_assert(x > 0, "not constant"); // 错误

// ✅ 最佳实践：在模板中检查类型约束
template<typename T>
void sorted(T& container) {
    static_assert(std::is_same<typename T::value_type, int>::value,
                  "Only int containers supported");
}
```

---

## 8. 其他重要但较简单的特性

### 8.1 `long long` 与 `<cstdint>`

```cpp
long long big = 9223372036854775807LL;
unsigned long long huge = 18446744073709551615ULL;
#include <cstdint>
int32_t i32 = 100;
uint64_t u64 = 1ULL << 60;
```

### 8.2 原始字符串字面量

```cpp
std::string path = R"(C:\Program Files\MyApp)";
std::string regex = R"(\d+\.\d+)";
std::string custom = R"delim(Hello "world")delim";
```

### 8.3 用户定义字面量

```cpp
std::string operator"" _s(const char* str, size_t len) {
    return std::string(str, len);
}
auto str = "hello"_s;   // std::string

long long operator"" _km(long long x) { return x * 1000; }
auto dist = 5_km;       // 5000
```

### 8.4 `alignof` / `alignas`

```cpp
std::cout << alignof(int);           // 通常为 4
alignas(64) double arr[100];
struct alignas(16) Vec4 { float x,y,z,w; };
```

### 8.5 `noexcept` 说明符

```cpp
void f() noexcept;
void g() noexcept(true);
void h() noexcept(false);
template<typename T>
void swap(T& a, T& b) noexcept(noexcept(T(std::move(a))));
```

### 8.6 `thread_local` 存储期

```cpp
thread_local int counter = 0;
```

### 8.7 显式转换运算符

```cpp
struct SafeBool {
    explicit operator bool() const { return true; }
};
SafeBool sb;
if (sb) { ... }        // OK
// bool b = sb;        // 错误：需要显式转换
```

---

## 总结

C++11 语言特性除了移动语义、lambda、constexpr、变参模板等核心外，上述特性在日常编码中同样高频使用：

| 特性 | 核心价值 |
|------|----------|
| `auto` | 简化类型书写，支持泛型编程 |
| 范围 for | 简洁安全的遍历语法 |
| `override`/`final` | 虚函数覆盖安全与继承控制 |
| `= default`/`= delete` | 精细控制特殊成员函数 |
| 委托/继承构造函数 | 减少构造代码重复 |
| `static_assert` | 编译期契约检查 |
| 其他 | 类型安全、可移植性、性能优化 |

掌握这些特性是编写现代 C++ 代码的基础，也是面试中的高频考点。