# constexpr（C++11，C++14/17/20 增强）

**一句话定义**：`constexpr` 指示编译器在**编译期**对表达式求值，将计算结果嵌入常量上下文，从而提升性能并允许编译期类型计算（如模板参数、数组大小）。

---

## 详细语法与用法

### 基础语法

```cpp
// C++11：constexpr 函数必须单行 return，且只能调用 constexpr 函数
constexpr int square(int x) {
    return x * x;
}
constexpr int val = square(5);   // 编译期计算，val == 25

// C++14：放宽限制，允许循环、局部变量、多条语句
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i) result *= i;
    return result;
}
constexpr int fact5 = factorial(5);  // 编译期计算 120

// C++20：constexpr 支持虚函数、动态分配（需满足编译期释放）、std::vector/std::string（非完全支持）
```

### constexpr 变量

```cpp
constexpr double pi = 3.141592653589793;   // 编译期常量
constexpr int arr_size = 10;
int arr[arr_size];                         // 合法：数组大小必须在编译期确定

// constexpr 变量隐含 const，必须立即初始化
constexpr int a = 10;
// a = 20;                                 // 错误：不可修改

// 可配合用户定义字面量
constexpr long long operator"" _kb(unsigned long long v) { return v * 1024; }
constexpr auto size = 4_kb;                // 4096，编译期计算
```

### constexpr 函数

```cpp
// C++11：函数体只能有一条 return 语句
constexpr int gcd(int a, int b) {
    return (b == 0) ? a : gcd(b, a % b);
}
static_assert(gcd(48, 18) == 6, "gcd failed");

// C++14：允许局部变量、循环、if、switch
constexpr int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; ++i) {
        int tmp = b;
        b = a + b;
        a = tmp;
    }
    return b;
}
constexpr int fib10 = fibonacci(10);   // 55

// C++20：constexpr 函数内可包含 try-catch（但不能实际抛出）、动态分配（需配合 new/delete 编译期版本）
constexpr std::vector<int> makeVec(int n) {  // C++20
    std::vector<int> v;
    for (int i = 0; i < n; ++i) v.push_back(i);
    return v;
}
// 注意：编译期 vector 的析构必须被编译期执行，故需 constexpr 分配器支持
```

### constexpr 构造函数与用户定义字面量

```cpp
// constexpr 类：可编译期构造、成员函数 constexpr
class Point {
    int x_, y_;
public:
    constexpr Point(int x, int y) noexcept : x_(x), y_(y) {}
    constexpr int x() const { return x_; }
    constexpr int y() const { return y_; }
    constexpr double distance() const { 
        return sqrt(x_*x_ + y_*y_);   // C++26 前 sqrt 不是 constexpr，需用编译期实现
    }
};
constexpr Point p(3, 4);
constexpr int px = p.x();   // 3，编译期

// constexpr 字面量类要求：析构函数 trivial / constexpr，所有成员是字面量类型
```

### consteval 与 constinit（C++20）

```cpp
// consteval：立即函数，必须在编译期求值（不能运行时调用）
consteval int square_compile(int x) {
    return x * x;
}
constexpr int a = square_compile(5);   // OK
// int b = square_compile(5);          // 错误：consteval 结果必须用于常量表达式

// constinit：确保变量在静态初始化阶段初始化（避免静态初始化顺序问题）
constinit int global_val = 42;         // 静态初始化
// constinit int another = get_rand(); // 错误：get_rand 不是常量表达式
```

---

## 底层原理与实现机制

### 1. 编译期求值的实现方式

编译器对 `constexpr` 的处理本质上是**常量折叠（constant folding）** 和 **编译期解释执行**：

- **常量表达式**：由字面量、`constexpr` 变量、`constexpr` 函数调用等组成，编译器在 AST 阶段就可以计算出值。
- **编译期虚拟机**：对于较复杂的 `constexpr` 函数（循环、递归），编译器使用一个**编译期解释器**模拟执行。这个解释器在编译过程中运行，所有操作必须是编译期可确定的（如不涉及 I/O、系统调用、运行时多态）。
- **结果替换**：计算出的常量值会直接替换到使用位置，类似宏展开但类型安全。

### 2. constexpr 函数的两阶段编译

`constexpr` 函数必须满足两阶段要求：
- **编译期求值版本**：所有参数在编译期已知时，函数在编译期运行，产生常量结果。
- **运行时版本**：参数在编译期未知时，函数退化为普通函数，在运行时执行。

编译器通常生成两份代码或标记函数同时支持两种上下文。

```cpp
constexpr int add(int a, int b) { return a + b; }

int x = get_runtime_value();   // 运行时输入
int y = add(x, 10);            // 运行时调用 add（普通函数）
constexpr int z = add(3, 4);   // 编译期调用 add，z 为 7
```

### 3. constexpr 变量与内存布局

- **编译期常量**：`constexpr` 变量通常不会分配运行时存储（除非取其地址），直接嵌入代码段。
- **静态存储期**：若 `constexpr` 变量具有静态存储期（如全局、static 局部），编译器会在只读数据段（`.rodata`）分配空间，并填入编译期计算的值。
- **ODR 使用**：如果对 `constexpr` 变量取地址或绑定到引用，编译器必须为其分配存储，可能导致多定义问题（需用 `inline` 或置于匿名命名空间）。

### 4. 递归与循环的编译期开销

- **递归深度限制**：编译器对 `constexpr` 递归调用有深度限制（通常 512 层，可由 `-fconstexpr-depth` 调节）。过深递归会导致编译时间爆炸或编译失败。
- **循环展开**：`constexpr` 函数中的循环在编译期解释执行，不是展开为大量代码，而是逐步迭代计算，最终结果被替换。因此不会生成巨大的二进制文件。

### 5. constexpr 与模板元编程对比

- **模板元编程**：使用模板特化和递归，生成新的类型或常量，编译期计算完全由模板实例化机制完成。
- **constexpr**：更像是普通函数，使用编译期解释执行，代码可读性远高于模板元编程，且调试相对容易（静态断言、编译器错误信息）。

但 `constexpr` 不能完全替代模板元编程：模板可以“计算类型”（如根据条件选择不同类型），而 `constexpr` 仅计算值。

---

## 与相关特性的对比

### 1. constexpr vs const

| 特性 | `const` | `constexpr` |
|------|---------|-------------|
| **核心含义** | 运行时不可修改（只读） | 编译期常量，或可在编译期求值 |
| **变量初始化** | 可在运行时初始化（如 `const int x = get_val()`） | 必须在编译期完成初始化 |
| **函数修饰** | 表示成员函数不修改对象（`const` 成员函数） | 表示函数可在编译期求值（若参数为常量） |
| **数组大小** | ❌ 不可用于数组大小（除非编译期已知） | ✅ 可用于数组大小 |
| **模板参数** | ❌ 不可用作非类型模板参数（除非常量） | ✅ 可用作非类型模板参数 |

```cpp
int runtime = 10;
const int a = runtime;       // OK，只读但值在运行时确定
// constexpr int b = runtime; // 错误：runtime 不是常量表达式

constexpr int size = 10;
int arr[size];               // OK
// const int size2 = runtime; int arr2[size2]; // 错误（非标准 VLA）
```

### 2. constexpr vs consteval vs constinit（C++20）

| 特性 | `constexpr` | `consteval` | `constinit` |
|------|-------------|-------------|-------------|
| **适用对象** | 变量、函数、构造函数 | 函数 | 变量 |
| **求值时机** | 编译期**或**运行时 | **强制**编译期（立即函数） | 静态初始化（编译期或常量初始化） |
| **运行时调用** | ✅ 允许（若参数不是常量） | ❌ 禁止，必须编译期 | 不适用 |
| **典型场景** | 编译期可计算、也需运行时调用的通用函数 | 必须完全编译期求值的函数（如元编程辅助） | 避免静态初始化顺序问题的全局变量 |

```cpp
consteval int square(int x) { return x * x; }
constexpr int cube(int x) { return x * x * x; }

int main() {
    int a = cube(3);     // 编译期计算（优化后可能运行时直接赋值 27）
    int b = cube(rand()); // OK：运行时调用 cube
    // int c = square(rand()); // 错误：rand() 不是常量，但 square 必须编译期调用
}
```

`constinit` 与 `constexpr` 的区别：
- `constinit` 不要求变量是 `const`，可以在运行时修改，仅保证初始化在静态阶段完成（常量初始化或零初始化）。
- `constexpr` 隐含 `const`，值不可修改。

---

## 常见面试问题与解答

### 问题 1：`constexpr` 函数是否一定在编译期求值？

**答**：**不一定**。`constexpr` 函数只是**可能**在编译期求值，取决于：
- 函数参数是否是常量表达式
- 函数调用结果是否用在常量上下文（如模板参数、数组大小、`constexpr` 变量初始化）

如果所有参数是常量表达式且结果用于常量上下文，编译器**必须**在编译期求值；否则函数退化为普通运行时调用。

```cpp
constexpr int add(int a, int b) { return a + b; }

constexpr int compile_time = add(2, 3);        // 编译期求值
int runtime = add(rand(), rand());             // 运行时调用（参数非常量）
int also_runtime = add(2, 3);                  // 也可能运行时，但优化器可能折叠为 5
```

现代编译器（如 GCC、Clang）在开启优化（`-O2`）时，即使不在常量上下文，也会尽量将 `constexpr` 函数内联计算，但这属于优化，不是语言保证。

---

### 问题 2：`constexpr` 构造函数有什么用？如何编写 `constexpr` 类？

**答**：`constexpr` 构造函数允许在编译期创建类对象，用于编译期计算、模板元编程、`static_assert` 检查等。

编写 `constexpr` 类的要求（C++11/14/17）：
1. 所有成员变量必须是字面量类型（`int`、`double`、`constexpr` 类等）。
2. 构造函数必须用 `constexpr` 修饰，函数体在 C++11 中为空或只有赋值语句（C++14 放宽）。
3. 析构函数必须为平凡或 `constexpr`（C++20 之前要求平凡）。
4. 所有成员函数若需在编译期调用，也必须是 `constexpr`。

```cpp
// C++14 允许非空 constexpr 构造函数
class Complex {
    double re, im;
public:
    constexpr Complex(double r, double i) : re(r), im(i) {}  // 允许
    constexpr double real() const { return re; }
    constexpr double imag() const { return im; }
    constexpr Complex conj() const { return Complex(re, -im); }
};

constexpr Complex c(1.0, 2.0);
constexpr double real_part = c.real();  // 1.0
constexpr Complex c_conj = c.conj();
```

---

### 问题 3：`constexpr` 函数中能否使用 `new` 和 `delete`？

**答**：从 **C++20** 开始，`constexpr` 函数中可以使用 `new` 和 `delete`，但有严格限制：
- 分配的内存必须在**同一个常量求值过程中释放**（不能泄漏）。
- 只能使用 `std::allocator` 或全局 `new`/`delete` 的编译期支持版本。
- 无法分配内存后返回指针给调用者（因为指针在编译期无意义）。
- 标准库容器如 `std::vector`、`std::string` 在 C++20 中标记了 `constexpr` 构造函数和析构函数，但需要分配器也是 `constexpr`（如 `std::allocator`）。

```cpp
// C++20
constexpr std::vector<int> build_vector() {
    std::vector<int> v;
    v.push_back(1);
    v.push_back(2);
    return v;   // 返回值会在编译期析构，释放内存
}

static_assert(build_vector().size() == 2);  // C++20 编译期断言
```

C++17 及之前 `constexpr` 函数中禁止 `new`/`delete`，只能使用栈内存和全局常量。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：误以为 `constexpr` 变量可以不初始化

```cpp
constexpr int x;   // ❌ 错误：constexpr 变量必须初始化
```

#### 错误 2：在 `constexpr` 函数中调用非 `constexpr` 函数

```cpp
int get_rand() { return rand(); }
constexpr int bad() { return get_rand(); }   // ❌ 错误：get_rand 不是 constexpr
```

#### 错误 3：C++11 中 `constexpr` 函数体包含多条语句

```cpp
// C++11 错误
constexpr int max(int a, int b) {
    if (a > b) return a;   // ❌ 多条语句，函数体不是单一 return
    else return b;
}
// C++14+ 正确
```

#### 错误 4：将 `constexpr` 用于非字面量类型的成员

```cpp
class NonLiteral {
    std::string s;   // C++20 前 std::string 非字面量类型
};
constexpr NonLiteral obj;   // ❌ C++17 错误，C++20 允许（string 支持 constexpr）
```

#### 错误 5：混淆 `constexpr` 与 `consteval`，误用运行时参数

```cpp
consteval int twice(int x) { return 2 * x; }
int a = twice(10);               // OK
int b = twice(rand());           // ❌ 错误：rand() 不是常量
```

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **优先使用 `constexpr` 而非宏** | 类型安全、作用域可控、调试友好 |
| **能加 `constexpr` 尽量加** | 函数和变量标记 `constexpr` 可让编译器在常量上下文中优化，但不破坏运行时调用（除非 `consteval`） |
| **复杂编译期计算用 `constexpr` 函数** | 替代模板元编程，代码可读性大幅提升 |
| **C++14 后放心使用循环和分支** | `constexpr` 函数能力接近普通函数 |
| **静态检查优先用 `static_assert`** | 配合 `constexpr` 函数在编译期验证逻辑 |
| **使用 `consteval` 强制编译期求值** | 确保某些函数不会意外逃逸到运行时，避免性能损失 |
| **`constexpr` 类尽量提供 `constexpr` 成员函数** | 保证类对象可在编译期完整使用 |

```cpp
// 好例子：编译期断言
constexpr bool is_prime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; ++i) if (n % i == 0) return false;
    return true;
}
static_assert(is_prime(17), "17 should be prime");
static_assert(!is_prime(15), "15 is not prime");

// 坏例子：宏定义
#define IS_PRIME(n) ...   // 危险，应使用 constexpr 函数
```