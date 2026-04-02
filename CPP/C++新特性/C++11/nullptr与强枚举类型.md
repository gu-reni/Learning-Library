# nullptr（C++11）

**一句话定义**：`nullptr` 是一个类型安全的空指针字面量，可隐式转换为任意指针类型或成员指针类型，但不能转换为整数类型，从而消除了传统 `NULL` 在重载解析中的二义性问题。

---

## 详细语法与用法

### 基本用法

```cpp
// C++11
int* p1 = nullptr;          // 指向整数的空指针
void* p2 = nullptr;         // 通用空指针
auto p3 = nullptr;          // 类型为 std::nullptr_t

// 检查空指针
if (p1 == nullptr) { /* ... */ }
if (p1) { /* ... */ }       // 也可直接作为布尔条件

// 函数重载
void f(int* p) { std::cout << "pointer\n"; }
void f(int i) { std::cout << "int\n"; }

f(nullptr);   // 调用 f(int*)
// f(NULL);   // 二义性：有些实现中 NULL 是 0，可能调用 f(int)
```

### `std::nullptr_t`

```cpp
#include <cstddef>

std::nullptr_t np = nullptr;   // nullptr 的类型
int* q = np;                   // 可隐式转换

// 可以用于模板参数推导
template<typename T>
void func(T t) { /* ... */ }
func(nullptr);  // T 推导为 std::nullptr_t

// 作为函数参数类型，只接受 nullptr
void accept_only_nullptr(std::nullptr_t) {
    std::cout << "got nullptr\n";
}
accept_only_nullptr(nullptr);   // OK
// accept_only_nullptr(0);      // 错误：类型不匹配
```

### 与 NULL 的对比

```cpp
// C++98/03 中的 NULL 通常是 #define NULL 0 或 #define NULL ((void*)0)
// 导致的问题：
void g(int) { std::cout << "int\n"; }
void g(char*) { std::cout << "char*\n"; }

g(NULL);   // 调用 g(int)（如果 NULL 是 0），而非 g(char*)，可能不符合预期

// C++11 使用 nullptr 解决
g(nullptr);   // 总是调用 g(char*)
```

---

## 底层原理与实现机制

### 1. `std::nullptr_t` 的类型定义

`nullptr` 的类型是 `std::nullptr_t`（定义在 `<cstddef>` 中）。该类型是一个**平凡的字面量类型**，其大小通常与 `void*` 相同（但不要求完全相同）。

```cpp
namespace std {
    using nullptr_t = decltype(nullptr);
}
```

`std::nullptr_t` 的特质：
- 可隐式转换为任何指针类型（包括成员指针）。
- 可隐式转换为 `bool`（`false`）。
- 不可隐式转换为整数类型（`int`、`long` 等）。
- 可比较（`==`、`!=`、`<=`、`>=`、`<`、`>`）与指针或 `nullptr_t` 类型。

### 2. 实现原理

编译器将 `nullptr` 视为一个**内建关键字**，其类型为 `std::nullptr_t`。在底层，它通常被实现为一个特殊的常量，值为 0 但类型不同。转换规则由语言标准规定：

- 从 `nullptr_t` 到指针类型的转换：生成一个该指针类型的空指针值（内存地址为 0，但具体表示可能因架构而异，不过总是比较相等）。
- 从 `nullptr_t` 到整数类型：标准禁止隐式转换，防止与整数 0 混淆。

### 3. 重载解析规则

`nullptr_t` 到指针类型的转换是**标准转换序列**，其等级为**指针转换**。而整数 `0` 到指针类型的转换也是**指针转换**，但整数 `0` 本身是 `int`，如果同时存在 `int` 和指针类型的重载，整数 `0` 会更匹配 `int` 版本。

```cpp
void h(int*);
void h(int);

h(0);        // 调用 h(int)
h(nullptr);  // 调用 h(int*)
```

### 4. 内存表示

`nullptr` 的底层值通常为 `0`（所有位为零），但这由实现定义。不同指针类型的空指针表示可能不同（某些架构有特殊空指针值），但 `nullptr` 转换后能得到正确的空指针值。

---

## 与相关特性的对比

### 1. `nullptr` vs `NULL` vs `0`

| 特性 | `0` | `NULL`（典型实现） | `nullptr` |
|------|-----|-------------------|-----------|
| 类型 | `int` | 整数类型（`0` 或 `0L`）或 `void*` | `std::nullptr_t` |
| 可转换为指针 | ✅ 隐式 | ✅ 隐式 | ✅ 隐式 |
| 可转换为整数 | ✅ 是整数 | ✅ 是整数 | ❌ 禁止隐式转换 |
| 重载 `int` vs `int*` | 调用 `int` 版本 | 可能调用 `int`（若定义为 `0`） | 调用 `int*` 版本 |
| 类型安全 | 差 | 差 | 好 |
| 模板推导 | `int` | `int` 或 `void*` | `std::nullptr_t` |

### 2. `nullptr` vs `nullptr_t` 变量

```cpp
std::nullptr_t mynull = nullptr;  // 定义自己的空指针对象
int* p = mynull;                  // 同样有效
```

区别：`nullptr` 是关键字字面量，而 `std::nullptr_t` 是类型。可以定义 `nullptr_t` 类型的变量，但通常不需要。

---

## 常见面试问题与解答

### 问题 1：为什么 C++11 引入 `nullptr` 而不是继续使用 `NULL`？

**答**：C++98/03 中 `NULL` 通常被定义为 `0` 或 `(void*)0`。这带来两个主要问题：

1. **重载二义性**：当函数同时重载 `int` 和指针类型时，`NULL`（作为 `0`）会匹配 `int` 版本，而不是预期的指针版本。
2. **类型不安全**：`NULL` 本质是整数，可以参与整数运算，导致错误代码通过编译（如 `int i = NULL;`）。

`nullptr` 拥有独立的类型 `std::nullptr_t`，只能隐式转换为指针类型，不能转换为整数，彻底解决了这些问题。

---

### 问题 2：`nullptr` 的类型是 `std::nullptr_t`，能否定义 `std::nullptr_t` 类型的变量？有什么用途？

**答**：可以。`std::nullptr_t` 是普通类型，可以定义变量，且该变量可以隐式转换为任意指针类型。

```cpp
std::nullptr_t np = nullptr;
int* p = np;   // 合法，p 为空指针
```

**用途**：
- 作为函数参数，强制要求传入 `nullptr` 而非其他整数。
- 模板元编程中检测是否为 `nullptr` 类型。
- 实现自定义的空指针标记。

---

### 问题 3：`nullptr` 是否保证等于 `0`？`(int)nullptr` 合法吗？

**答**：`nullptr` 转换为指针后，该指针与任何空指针比较相等（包括 `0` 转换来的空指针），但 `nullptr` **不能隐式转换为整数**。若使用 `reinterpret_cast<int>(nullptr)` 或 `(int)nullptr` 是**非法**的（编译错误）。如果需要获取数值表示，通常不需要；如果坚持要，可用 `(int)(intptr_t)(void*)nullptr` 但这是实现相关的，不推荐。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：将 `nullptr` 当作整数使用

```cpp
int x = nullptr;          // 错误：不能隐式转换
if (nullptr == 0) { }     // 错误：不允许比较 nullptr 和 int（某些编译器可能允许但不可移植）
```

#### 错误 2：在需要整数 0 的地方误用 `nullptr`

```cpp
int arr[10];
int* p = arr + nullptr;   // 错误：指针算术不能加 nullptr
int index = 0;
// int* p2 = arr + index; // 正确：应使用 0
```

#### 错误 3：假设所有空指针值都是全 0 位

某些架构（如旧式 CDC Cyber）的空指针不是全 0 位，但 `nullptr` 转换后保证是正确的空指针值。不应直接比较内存字节。

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **始终使用 `nullptr` 代替 `NULL` 和 `0`** | 提高类型安全，避免重载歧义 |
| **检查指针是否为空用 `if (ptr == nullptr)` 或 `if (!ptr)`** | 后者简洁，两者等价 |
| **函数重载中利用 `nullptr` 区分整数和指针** | 提供更清晰的 API |
| **在模板中传递空指针时用 `nullptr`** | 避免 `0` 被推导为整数类型 |

```cpp
// 好的示例：模板中统一使用 nullptr
template<typename T>
T* create() {
    return nullptr;   // 总是返回正确的空指针
}
```

---

# 强枚举类型（enum class，C++11）

**一句话定义**：强枚举类型（`enum class`）是一种作用域限定的、类型安全的枚举，其枚举值不会隐式转换为整数，解决了传统 C 风格枚举的命名冲突和类型安全问题。

---

## 详细语法与用法

### 基本语法

```cpp
// C++11 强枚举类型
enum class Color {
    Red,
    Green,
    Blue
};

// 使用：必须加上作用域
Color c = Color::Red;

// 传统 C 风格枚举（无作用域）
enum OldColor {
    OldRed,    // 全局作用域污染
    OldGreen,
    OldBlue
};
int x = OldRed;   // 隐式转换为 int

// 强枚举类型不能隐式转换
// int y = Color::Red;   // 错误：不能隐式转换
int y = static_cast<int>(Color::Red);  // OK，显式转换
```

### 指定底层类型

```cpp
// 默认底层类型为 int
enum class Status : unsigned int {
    Ok = 0,
    Warning = 1,
    Error = 2
};

// 可以使用任何整数类型（char, short, long long 等）
enum class ByteFlags : uint8_t {
    Read = 0x01,
    Write = 0x02,
    Execute = 0x04
};
```

### 前向声明

```cpp
// 必须指定底层类型才能前向声明
enum class MyEnum : int;   // 前向声明

// 之后定义
enum class MyEnum : int {
    A, B, C
};
```

### 运算符重载与使用

```cpp
enum class Flags : unsigned int {
    None = 0,
    Bit0 = 1 << 0,
    Bit1 = 1 << 1,
    Bit2 = 1 << 2
};

// 为强枚举定义位运算
inline Flags operator|(Flags a, Flags b) {
    return static_cast<Flags>(static_cast<unsigned int>(a) | static_cast<unsigned int>(b));
}

Flags f = Flags::Bit0 | Flags::Bit2;
```

### C++17 的初始化改进

```cpp
// C++17 允许在 if/switch 中初始化枚举变量
if (auto c = get_color(); c == Color::Red) {
    // ...
}
```

### C++20 的 `using enum`

```cpp
// C++20：引入枚举值到当前作用域
enum class Color { Red, Green, Blue };

void print(Color c) {
    using enum Color;   // 将 Red, Green, Blue 引入当前作用域
    switch (c) {
        case Red:   std::cout << "Red\n"; break;
        case Green: std::cout << "Green\n"; break;
        case Blue:  std::cout << "Blue\n"; break;
    }
}
```

---

## 底层原理与实现机制

### 1. 内存布局与表示

强枚举类型在内存中的表示由其**底层类型**决定。如果未显式指定，默认是 `int`。枚举值从 0 开始递增，但可显式赋值。枚举类型的大小等于其底层类型的大小，且对齐要求相同。

```cpp
enum class Small : char { A, B };   // sizeof(Small) == 1
enum class Large : long long { X }; // sizeof(Large) == 8
```

编译器通常将枚举变量直接存储为底层整数值，访问时只需读取该整数。

### 2. 作用域实现

传统 C 枚举将枚举值注入到**外围作用域**（如全局或命名空间），导致容易命名冲突。强枚举类型将枚举值放在枚举类型的作用域内，通过名称修饰（name mangling）实现。编译器生成符号时，`Color::Red` 会编码为类似 `Color_Red` 的内部名称，避免冲突。

### 3. 类型安全性

强枚举类型**禁止隐式转换为整数**，因为编译器不会为 `enum class` 生成到整数的隐式转换运算符。如果需要转换，必须使用 `static_cast`。这避免了意外将枚举值当作整数使用（如作为数组下标）。

### 4. 底层类型的影响

指定底层类型允许控制枚举的大小和对齐，对于序列化、嵌入式编程、跨平台协议非常重要。同时，前向声明需要知道底层类型，以便编译器确定对象大小。

### 5. 与传统枚举的二进制兼容性

如果强枚举的底层类型与传统枚举相同，并且布局一致，则可以通过 `reinterpret_cast` 转换（但通常不推荐）。标准未保证两者二进制兼容，但实践中通常可以。

---

## 与相关特性的对比

### 1. `enum class` vs 传统 `enum`

| 特性 | 传统 `enum` | `enum class` |
|------|-------------|--------------|
| 作用域 | 无作用域，枚举值暴露到外围 | 有作用域，需 `EnumName::Value` |
| 隐式转换到整数 | ✅ 允许 | ❌ 禁止（需显式 `static_cast`） |
| 底层类型 | 不固定，由编译器选择 | 默认 `int`，可显式指定任何整数类型 |
| 前向声明 | 不可（除非指定底层类型，C++11 后也可） | 可（需指定底层类型） |
| 重载 | 枚举值可能意外匹配整数重载 | 类型安全，不会匹配整数 |
| 内存占用 | 编译器决定，通常 `int` | 由底层类型决定，可控 |

### 2. `enum class` vs `enum` 指定底层类型（C++11 传统枚举也可指定）

```cpp
enum OldEnum : unsigned int { A, B };   // C++11 允许传统枚举指定底层类型
enum class NewEnum : unsigned int { A, B };
```

区别仍在于作用域和隐式转换。传统枚举即使指定底层类型，仍会隐式转换为整数。

### 3. `enum class` vs 常量或 `constexpr` 变量

对于需要一组相关常量的场景，可以用 `constexpr` 变量命名空间模拟：

```cpp
namespace Color {
    constexpr int Red = 0;
    constexpr int Green = 1;
    constexpr int Blue = 2;
}
```

但这种方式没有类型安全（仍是 `int`），且无法利用枚举的 switch 检查和调试器显示。`enum class` 更好表达一组有限的值。

---

## 常见面试问题与解答

### 问题 1：强枚举类型相比传统枚举有哪些优势？为什么要引入？

**答**：三大优势：

1. **作用域控制**：传统枚举的枚举值注入外围作用域，容易命名冲突。强枚举值在 `enum class` 作用域内，避免污染。
2. **类型安全**：传统枚举可隐式转换为整数，导致函数重载混乱、意外作为数组下标。强枚举禁止隐式转换，必须显式 `static_cast`。
3. **底层类型可控**：强枚举默认底层类型为 `int`，可显式指定更小的类型（如 `char`），节省内存且布局可预测。

引入 `enum class` 是为了提供更现代、更安全的枚举设施，同时保持与 C 的向后兼容（传统枚举仍保留）。

---

### 问题 2：如何将强枚举转换为整数，以及整数转换为强枚举？

**答**：必须使用显式转换。

```cpp
enum class Color { Red, Green, Blue };

// 枚举 → 整数
int val = static_cast<int>(Color::Red);   // 0

// 整数 → 枚举
Color c = static_cast<Color>(1);          // Color::Green

// 但需要注意：整数不一定对应有效的枚举值，这是未定义行为（但通常只是得到未列出的值）
```

如果需要更安全的转换，可以编写辅助函数进行边界检查。

---

### 问题 3：`enum class` 能否用于位标志（bit flags）？如何支持位运算？

**答**：可以，但需要显式重载位运算符，因为强枚举不隐式转换为整数。

```cpp
enum class Permissions : uint8_t {
    None  = 0,
    Read  = 1 << 0,
    Write = 1 << 1,
    Exec  = 1 << 2
};

inline Permissions operator|(Permissions a, Permissions b) {
    return static_cast<Permissions>(static_cast<uint8_t>(a) | static_cast<uint8_t>(b));
}

inline bool has(Permissions value, Permissions flag) {
    return (static_cast<uint8_t>(value) & static_cast<uint8_t>(flag)) != 0;
}
```

C++ 标准库没有为 `enum class` 提供自动的位运算符，但用户可轻松添加。

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：试图隐式转换强枚举到整数

```cpp
enum class Color { Red };
int r = Color::Red;   // 错误：不能隐式转换
```

#### 错误 2：忘记作用域，直接使用枚举值

```cpp
enum class Color { Red };
Color c = Red;   // 错误：Red 不在作用域内
```

#### 错误 3：使用强枚举作为数组下标（忘记转换）

```cpp
enum class Index { Zero, One, Two };
int arr[3];
arr[Index::Zero] = 10;   // 错误：不能作为下标，需要 static_cast<int>
```

#### 错误 4：switch 中漏掉枚举值但未处理默认情况（传统枚举会警告，强枚举同样会）

```cpp
Color c = Color::Red;
switch (c) {
    case Color::Red: break;
    // 没有处理 Green 和 Blue，如果 c 是其他值，行为未定义（但通常不会）
}
```

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **新代码优先使用 `enum class`** | 除非需要与传统 C 代码交互或隐式转换，否则始终用强枚举 |
| **显式指定底层类型** | 对于需要序列化、跨平台、节省内存的场景，指定如 `: uint8_t` |
| **为位标志重载运算符** | 提供直观的 `|`、`&`、`^`、`~` 等 |
| **提供辅助转换函数** | 如果需要频繁枚举与整数互转，封装为命名函数 |
| **使用 `using enum`（C++20）简化作用域** | 在局部作用域内避免冗长的前缀 |
| **避免使用 `static_cast` 转换非法值** | 确保转换的整数在枚举值范围内，否则可能引发未定义行为 |

```cpp
// 好的示例：安全转换函数
template<typename E>
constexpr auto to_underlying(E e) noexcept {
    return static_cast<std::underlying_type_t<E>>(e);
}
// C++23 标准库提供了 std::to_underlying

int val = to_underlying(Color::Red);
```