## `<cstddef>` 头文件详解

`<cstddef>` 是 C++ 标准库中从 C 标准库继承而来的头文件，定义了**常用类型**（如 `size_t`、`ptrdiff_t`、`nullptr_t` 等）和**宏**（如 `NULL`、`offsetof`）。这些构件是语言核心和标准库的基础，用于表示对象大小、指针差值、空指针常量以及结构体成员的偏移量。

---

## 一、类型定义（typedef）

### 1. `std::size_t`

**定义：** 无符号整数类型，是 `sizeof` 操作符和 `alignof` 操作符的结果类型，也是下标操作符的常见类型。

**作用：** 表示对象的大小（字节数）和数组索引。保证能够表达任何对象的尺寸。

**典型别名：** `unsigned long` 或 `unsigned long long`（取决于平台）。

**示例：**
```cpp
#include <cstddef>
std::size_t len = sizeof(int);
std::size_t idx = 0;
```

---

### 2. `std::ptrdiff_t`

**定义：** 有符号整数类型，是两个指针相减的结果类型。

**作用：** 表示指针之间的差值（以元素个数为单位），通常用于指针算术。

**典型别名：** `long` 或 `long long`。

**示例：**
```cpp
int arr[10];
int* p = &arr[0];
int* q = &arr[5];
std::ptrdiff_t diff = q - p;   // 5
```

---

### 3. `std::nullptr_t`（C++11 起）

**定义：** `decltype(nullptr)` 的类型，即空指针常量的类型。

**作用：** 可以隐式转换为任何指针类型或成员指针类型，但不能转换为整数类型（除了 `0` 字面量）。

**示例：**
```cpp
#include <cstddef>
std::nullptr_t np = nullptr;
int* p = np;      // ok
// int i = np;    // 错误
```

---

### 4. `std::max_align_t`（C++11 起）

**定义：** 一种标量类型，其对齐要求不小于任何基础类型（通常为 `long double` 或 `long long` 的对齐）。

**作用：** 用于 `std::aligned_storage` 等需要较大对齐的场景，表示最大的基础对齐值。

**示例：**
```cpp
#include <cstddef>
#include <iostream>
int main() {
    std::cout << alignof(std::max_align_t) << '\n';
}
```

---

## 二、宏定义详解

### 1. `NULL`

**定义：** 实现定义的空指针常量。在 C++ 中通常定义为 `0` 或 `nullptr`（C++11 后鼓励使用 `nullptr` 代替）。

**作用：** 表示一个空指针值。

**注意：** 由于 `NULL` 可能被定义为整数 `0`，在重载时可能会引起歧义，因此推荐使用 `nullptr`。

**示例：**
```cpp
int* p = NULL;       // C 风格
int* q = nullptr;    // 现代 C++ 推荐
```

---

### 2. `offsetof(type, member)`

**定义：** 宏，展开为一个 `size_t` 类型的常量表达式，表示结构体 `type` 中成员 `member` 相对于结构体起始地址的字节偏移量。

**参数：**
- `type`：结构体类型名（或类、联合体）。
- `member`：该类型的成员名。

**返回值：** `size_t` 类型，表示偏移字节数。

**示例：**
```cpp
#include <cstddef>
struct Point { int x; int y; };
std::size_t off = offsetof(Point, y);   // 通常为 4（若 int 占 4 字节）
```

**实现原理：** 在编译期计算成员地址减去结构体首地址。内部可能使用编译器内建函数：`__builtin_offsetof`（GCC/Clang）。

**使用注意事项：** `offsetof` 对标准布局类型（standard-layout）有效，对非标准布局类型的行为未定义。

---

## 三、结构体详解

`<cstddef>` 不定义任何结构体。它提供的 `std::nullptr_t` 是一个类类型（但不能直接实例化使用），`std::max_align_t` 是一个标量类型。

---

## 四、模板声明

`<cstddef>` 不定义任何函数模板。但它定义了与 `std::byte`（C++17）配合的模板操作符重载（`operator>>=`、`operator<<=` 等），这些定义在 `<cstddef>` 中。

### `std::byte`（C++17 起）

- **定义：** 枚举类型：`enum class byte : unsigned char {};`
- **作用：** 表示内存的原始字节（8 位），不表示字符或数值，避免与字符类型混淆。
- **提供的操作：** 按位与、或、异或、取反、移位；以及 `std::to_integer` 函数。

**注意：** `std::byte` 的定义在 `<cstddef>` 中，但相关操作符重载也在该头文件中提供。

**示例：**
```cpp
#include <cstddef>
std::byte b{0x3F};
b = b << 2;
int val = std::to_integer<int>(b);
```

---
