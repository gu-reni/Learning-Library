# constexpr 增强（C++14，C++17/20 持续演进）

**一句话定义**：C++14 对 `constexpr` 进行了重大增强，允许函数包含局部变量、循环、分支和修改对象等多条语句，使编译期计算能力大幅提升，让 `constexpr` 函数更接近普通函数；后续 C++17/20 进一步放宽限制（如 lambda、虚函数、动态分配等）。

---

## 详细语法与用法

### C++11 的 `constexpr` 限制回顾

C++11 中 `constexpr` 函数只能包含**一条 `return` 语句**，通常使用递归或条件运算符（`?:`）实现逻辑。

```cpp
// C++11 风格
constexpr int factorial_cpp11(int n) {
    return n <= 1 ? 1 : n * factorial_cpp11(n - 1);
}
```

### C++14 的增强：多语句、循环、局部变量

```cpp
// C++14：允许局部变量、循环、if 语句
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) {
        result *= i;
    }
    return result;
}
static_assert(factorial(5) == 120);

// 允许修改局部对象
constexpr int modify(int x) {
    int y = x;        // 局部变量
    y += 10;          // 修改局部对象（允许）
    return y;
}
static_assert(modify(5) == 15);

// 允许 switch、多个语句
constexpr int digit_sum(int n) {
    int sum = 0;
    while (n > 0) {
        sum += n % 10;
        n /= 10;
    }
    return sum;
}
static_assert(digit_sum(12345) == 15);
```

### `constexpr` 成员函数与构造函数

```cpp
class Point {
    int x_, y_;
public:
    constexpr Point(int x, int y) noexcept : x_(x), y_(y) {}
    constexpr int x() const { return x_; }
    constexpr int y() const { return y_; }
    // C++14 允许修改成员（注意：const 成员函数不能修改，但非 const 可）
    constexpr void setX(int x) { x_ = x; }   // C++14 允许
};

constexpr Point p(3, 4);
constexpr int px = p.x();   // 3
```

### C++17 的进一步增强

```cpp
// constexpr lambda（C++17 自动成为 constexpr 若可能）
constexpr auto square = [](int x) { return x * x; };
static_assert(square(5) == 25);

// if constexpr：编译期条件分支（C++17）
template<typename T>
auto get_value(T t) {
    if constexpr (std::is_pointer_v<T>) {
        return *t;
    } else {
        return t;
    }
}
```

### C++20 的增强

```cpp
// constexpr 虚函数（C++20）
struct Base {
    constexpr virtual int f() const { return 0; }
};
struct Derived : Base {
    constexpr int f() const override { return 42; }
};
constexpr Base* b = new Derived{};
static_assert(b->f() == 42);  // 编译期多态

// constexpr 动态分配（C++20）
constexpr auto make_vec() {
    std::vector<int> v = {1, 2, 3};
    v.push_back(4);
    return v;
}
static_assert(make_vec().size() == 4);  // 需 constexpr 分配器支持
```

---

## 底层原理与实现机制

### 编译期虚拟机（Compile-time Virtual Machine）

C++14 的 `constexpr` 函数放宽后，编译器在常量求值时需要能够模拟执行**局部变量、循环、分支**等操作。这通常通过一个**编译期解释器**实现，该解释器在编译过程中运行，跟踪栈帧、局部变量和控制流。

- 所有操作必须是编译期可确定的（不能有 I/O、系统调用、运行时多态等）。
- 循环会被展开或解释执行，但不会生成运行时代码。
- 局部变量在常量表达式上下文中被分配在编译器的符号表中。

### 与 C++11 的区别

| 方面 | C++11 | C++14+ |
|------|-------|--------|
| 函数体限制 | 单条 `return` 语句 | 多条语句，允许 `if`、`switch`、`while`、`for` |
| 局部变量 | 不允许 | 允许，但必须是字面量类型 |
| 修改对象 | 不允许（函数式） | 允许修改局部对象和传入的非 const 引用 |
| 递归深度限制 | 有（通常 512） | 同，但可用循环避免递归 |

### 编译期内存模型

C++14 中 `constexpr` 函数内的局部变量生命周期仅限于常量求值期间，它们不会被分配到运行时的栈上，而是由编译器内部管理。C++20 引入 `constexpr` 动态分配后，编译器需要在编译期模拟堆内存（通常使用静态分配池）。

### 性能影响

- `constexpr` 函数在编译期求值后，结果直接嵌入代码，运行时零开销。
- 若 `constexpr` 函数在运行时调用（参数非常量），编译器生成普通函数代码，性能与普通函数相同（可能内联）。

---

## 与相关特性的对比

### 1. `constexpr` vs `consteval`（C++20）

| 特性 | `constexpr` | `consteval`（立即函数） |
|------|-------------|------------------------|
| 求值时机 | 编译期或运行时 | **强制**编译期 |
| 运行时调用 | ✅ 允许（参数非常量时） | ❌ 禁止，必须常量参数 |
| 适用场景 | 通用编译期计算 | 必须编译期求值的函数（如元编程辅助） |

### 2. `constexpr` vs `constinit`（C++20）

- `constinit` 保证变量在静态初始化阶段初始化（编译期或常量初始化），但变量本身不是 `const`，可在运行时修改。
- `constexpr` 变量隐含 `const`，必须在编译期初始化且不可修改。

### 3. C++14 `constexpr` vs 模板元编程

| 方面 | C++14 `constexpr` 函数 | 模板元编程 |
|------|------------------------|------------|
| 可读性 | 高（像普通函数） | 低（晦涩的递归特化） |
| 调试 | 可在编译期断言 | 模板错误信息冗长 |
| 性能 | 编译期求值后零开销 | 同 |
| 类型计算 | 不能（只能计算值） | 可以（计算类型） |

`constexpr` 无法替代模板元编程的类型计算（如根据条件选择类型），但可替代大部分数值计算。

---

## 常见面试问题与解答

### 问题 1：C++14 中 `constexpr` 函数能否修改传入的引用参数？为什么？

**答**：可以，只要该引用在常量求值上下文中绑定到可修改的对象（如局部变量或全局常量）。例如：

```cpp
constexpr void increment(int& x) {
    ++x;   // C++14 允许
}
constexpr int test() {
    int a = 5;
    increment(a);
    return a;
}
static_assert(test() == 6);
```

限制：不能修改具有静态存储期的对象（除非是编译期常量），因为常量表达式要求结果仍是常量。

---

### 问题 2：`constexpr` 函数中能否使用 `goto` 或 `try-catch`？

**答**：C++14/17 中不允许 `goto` 和 `try-catch`（除了 C++20 允许 `try-catch` 但不能实际抛出）。这些语句破坏了常量求值的确定性。

---

### 问题 3：C++14 中 `constexpr` 函数可以递归吗？递归深度有限制吗？

**答**：可以递归，但编译器有递归深度限制（通常 512 层，可由 `-fconstexpr-depth` 调节）。C++14 支持循环后，建议用循环代替递归以避免深度问题。

---

### 问题 4：C++14 中 `constexpr` 成员函数是否可以修改 `this` 指向的对象？

**答**：可以，但该成员函数不能是 `const`。`const` 成员函数中即使标记 `constexpr` 也不能修改对象。

```cpp
struct S {
    int x = 0;
    constexpr void set(int v) { x = v; }   // OK，非 const
    constexpr int get() const { return x; } // const，不能修改
};
constexpr S s;
s.set(10);   // 错误：s 是 const 对象，不能调用非 const 成员
```

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：在 `constexpr` 函数中使用静态局部变量

```cpp
constexpr int bad() {
    static int x = 0;   // 错误：静态变量不允许
    return ++x;
}
```

#### 错误 2：在 `constexpr` 函数中调用非 `constexpr` 函数

```cpp
int rand() { return 42; }
constexpr int f() { return rand(); }   // 错误：rand 不是 constexpr
```

#### 错误 3：C++14 中误以为可以修改全局变量

```cpp
int global = 10;
constexpr void modify_global() {
    global = 20;   // 错误：修改全局变量在常量表达式中不允许
}
```

#### 错误 4：`constexpr` 函数内使用 `new`/`delete`（C++20 前不允许）

```cpp
constexpr int* alloc() {
    return new int(5);   // C++14 错误，C++20 允许（但需配套 delete）
}
```

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **用 `constexpr` 替代模板元编程计算数值** | 代码更清晰，如 `factorial`、`gcd` 等 |
| **在可能的地方标记 `constexpr`** | 函数和变量加 `constexpr` 可提升性能（编译期计算） |
| **C++14 后优先使用循环而非递归** | 避免递归深度限制，编译更快 |
| **复杂 `constexpr` 函数用 `static_assert` 测试** | 确保编译期行为符合预期 |
| **注意 `constexpr` 构造函数和析构函数** | C++20 前析构函数不能是 `constexpr`（C++20 放宽） |

```cpp
// 好的示例：编译期校验
constexpr bool is_prime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; ++i)
        if (n % i == 0) return false;
    return true;
}
static_assert(is_prime(17), "17 is prime");
static_assert(!is_prime(15), "15 is not prime");
```

---

## 编译器支持与版本演进

| 标准 | 主要增强 | 编译器最低版本（完整支持） |
|------|----------|---------------------------|
| C++11 | 引入 `constexpr`（单语句函数） | GCC 4.6, Clang 3.1, MSVC 2015 |
| C++14 | 多语句、循环、局部变量 | GCC 5, Clang 3.4, MSVC 2015 |
| C++17 | `constexpr` lambda、`if constexpr` | GCC 7, Clang 5, MSVC 2017 |
| C++20 | `constexpr` 虚函数、动态分配、`constexpr` 容器（如 `std::vector`） | GCC 10, Clang 10, MSVC 2019 16.10 |

**特性测试宏**：
```cpp
#ifdef __cpp_constexpr
// 值：200704（C++11）、201304（C++14）、201603（C++17）、201907（C++20）
#endif
```

---

**总结**：C++14 的 `constexpr` 增强是里程碑式的改进，使得编译期编程从“晦涩的函数式风格”进化为“自然的过程式风格”，极大降低了使用门槛。配合 C++17/20 的持续演进，`constexpr` 已经成为现代 C++ 中不可或缺的编译期计算工具。面试中应重点掌握 C++14 的增强点、与 `consteval` 的区别、以及典型使用场景。