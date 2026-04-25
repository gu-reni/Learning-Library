## `<cassert>` 头文件详解

`<cassert>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**编译时和运行时的断言功能**。它主要定义了 `assert` 宏，用于在调试阶段检查程序中的逻辑条件是否成立。如果断言失败，程序会输出错误信息并终止执行，从而帮助开发者快速定位逻辑错误。该头文件在 Release 编译（定义了 `NDEBUG` 宏）时可以被完全移除，不产生任何运行时开销。

---

## 一、宏定义详解

### 1. `assert`

**宏原型：**
```cpp
assert(expression);
```

**作用：** 计算表达式 `expression` 的值。如果该值为假（即 `0`），则 `assert` 会将一条诊断信息（包含文件名、行号、失败的表达式）写入标准错误输出，并调用 `std::abort()` 终止程序执行。如果表达式为真，则不做任何操作。

**参数：**
- `expression`：任何可以隐式转换为 `bool` 的标量表达式。如果结果为 `0` 或 `false`，触发断言失败。

**示例用法：**
```cpp
#include <cassert>

int divide(int a, int b) {
    assert(b != 0);   // 在 debug 构建中检查除数不能为 0
    return a / b;
}

int main() {
    divide(10, 0);    // 当 NDEBUG 未定义时，程序会在此终止
    return 0;
}
```

**输出示例（Linux / GCC）：**
```
test: assert_test.cpp:5: int divide(int, int): Assertion `b != 0' failed.
Aborted (core dumped)
```

**实现原理：** `assert` 通常被实现为条件宏：
```cpp
#ifdef NDEBUG
#define assert(condition) ((void)0)
#else
#define assert(condition) \
    ((condition) ? (void)0 : \
     __assert_fail(#condition, __FILE__, __LINE__, __func__))
#endif
```
- 当未定义 `NDEBUG` 时（通常是 Debug 构建），`assert` 展开为对一个内部函数 `__assert_fail` 的调用，该函数输出诊断信息并调用 `abort()`。
- 当定义了 `NDEBUG` 时（通常是 Release 构建），`assert` 被替换为一条无作用的 `((void)0)` 语句，不会产生任何运行时开销。

**线程安全提示：** `assert` 宏本身只做条件判断和函数调用，在表达式求值过程中不会修改任何状态。如果表达式本身是线程安全的（例如只读取共享变量），则 `assert` 也是线程安全的。但需要注意：`assert` 可能调用 `abort()`，这会终止整个进程，影响所有线程。

---

### 2. `static_assert`（C++11 起）

**注意：** `static_assert` 实际上是 C++ 语言的内置关键字，并非 `<cassert>` 中定义的宏。它用于编译时断言，如果条件为假，则产生编译错误，并可选地输出错误信息。

**语法：**
```cpp
static_assert(bool_constexpr, message);   // C++11 语法
static_assert(bool_constexpr);            // C++17 起，message 可选
```

**作用：** 在编译时检查某个常量表达式是否为真。常用于模板编程、类型特性检查等场景。

**示例：**
```cpp
static_assert(sizeof(int) >= 4, "int is too small");
template<typename T>
void process(T value) {
    static_assert(std::is_integral_v<T>, "T must be integral type");
}
```

**与 `assert` 的区别：**
- `assert` 是运行时检查，仅在 Debug 构建中有效。
- `static_assert` 是编译时检查，无论构建类型都有效，且不产生运行时代码。

---

## 二、函数详解

`<cassert>` 本身不直接声明任何可供用户调用的函数。它内部使用了标准库的 `abort()` 函数（声明在 `<cstdlib>` 中）来终止程序。因此用户通常不会从 `<cassert>` 中获得新的函数声明。

---

## 三、结构体详解

`<cassert>` 不定义任何结构体。

---

## 四、类型定义

`<cassert>` 不定义任何类型。

---

## 五、宏与常量

| 宏名 | 作用 |
|------|------|
| `assert(expr)` | 断言宏。如果表达式为假，输出诊断信息并调用 `abort()`。 |
| `NDEBUG` | 不是 `<cassert>` 定义的宏，而是用户自定义的宏。在包含 `<cassert>` **之前**定义 `NDEBUG`，可以禁用 `assert` 宏。通常由编译器在 Release 模式下自动定义。 |

**使用 `NDEBUG` 禁用断言：**
```cpp
#define NDEBUG          // 必须在包含 <cassert> 之前定义
#include <cassert>

int main() {
    assert(1 == 2);     // 没有任何输出，程序继续运行
    return 0;
}
```

---

## 六、模板声明

`<cassert>` 是 C 风格头文件，不包含任何 C++ 模板。但在 C++ 中，它也提供了 `static_assert` 关键字（语言内置），不需要通过头文件。

---
