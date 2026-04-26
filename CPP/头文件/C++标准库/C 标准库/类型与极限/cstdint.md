## `<cstdint>` 头文件详解

`<cstdint>` 是 C++11 标准引入的头文件，定义了**固定宽度**的整数类型（如 `int8_t`、`uint32_t`），以及给定宽度的最小/最大类型、最快速类型等。它还提供了一系列宏，用于表示这些类型的最小值和最大值。该头文件可提高代码的可移植性，因为它允许程序员显式指定整数的大小和行为。

---

## 一、类型定义（typedef）

以下类型定义在命名空间 `std` 中。

### 1. 精确宽度整数类型（exact-width）

| 类型 | 作用 |
|------|------|
| `int8_t` | 有符号 8 位整数 |
| `int16_t` | 有符号 16 位整数 |
| `int32_t` | 有符号 32 位整数 |
| `int64_t` | 有符号 64 位整数 |
| `uint8_t` | 无符号 8 位整数 |
| `uint16_t` | 无符号 16 位整数 |
| `uint32_t` | 无符号 32 位整数 |
| `uint64_t` | 无符号 64 位整数 |

**注意：** 这些类型仅当系统提供正好该宽度的整数类型时才会定义，否则不存在。

### 2. 至少具有指定宽度的整数类型（minimum-width）

| 类型 | 作用 |
|------|------|
| `int_least8_t` | 至少 8 位的有符号整数类型 |
| `int_least16_t` | 至少 16 位的有符号整数类型 |
| `int_least32_t` | 至少 32 位的有符号整数类型 |
| `int_least64_t` | 至少 64 位的有符号整数类型 |
| `uint_least8_t` | 至少 8 位的无符号整数类型 |
| `uint_least16_t` | 至少 16 位的无符号整数类型 |
| `uint_least32_t` | 至少 32 位的无符号整数类型 |
| `uint_least64_t` | 至少 64 位的无符号整数类型 |

这些类型总是存在的，因为系统至少支持某种宽度。

### 3. 足够快的最小宽度整数类型（fastest minimum-width）

| 类型 | 作用 |
|------|------|
| `int_fast8_t` | 至少 8 位且运算速度最快的有符号整数类型 |
| `int_fast16_t` | 至少 16 位且运算速度最快的有符号整数类型 |
| `int_fast32_t` | 至少 32 位且运算速度最快的有符号整数类型 |
| `int_fast64_t` | 至少 64 位且运算速度最快的有符号整数类型 |
| `uint_fast8_t` | 至少 8 位且运算速度最快的无符号整数类型 |
| `uint_fast16_t` | 至少 16 位且运算速度最快的无符号整数类型 |
| `uint_fast32_t` | 至少 32 位且运算速度最快的无符号整数类型 |
| `uint_fast64_t` | 至少 64 位且运算速度最快的无符号整数类型 |

适用于对性能有要求，但对精确大小要求不严格的场景。

### 4. 能够保存指针值的整数类型

| 类型 | 作用 |
|------|------|
| `intptr_t` | 能够保存指针的有符号整数类型 |
| `uintptr_t` | 能够保存指针的无符号整数类型 |

### 5. 最大宽度整数类型

| 类型 | 作用 |
|------|------|
| `intmax_t` | 受支持的最大宽度有符号整数类型 |
| `uintmax_t` | 受支持的最大宽度无符号整数类型 |

---

## 二、宏定义详解

### 1. 精确宽度类型的极限值

| 宏名 | 作用 |
|------|------|
| `INT8_MIN` | `int8_t` 的最小值 |
| `INT8_MAX` | `int8_t` 的最大值 |
| `UINT8_MAX` | `uint8_t` 的最大值 |
| `INT16_MIN` | `int16_t` 的最小值 |
| `INT16_MAX` | `int16_t` 的最大值 |
| `UINT16_MAX` | `uint16_t` 的最大值 |
| `INT32_MIN` | `int32_t` 的最小值 |
| `INT32_MAX` | `int32_t` 的最大值 |
| `UINT32_MAX` | `uint32_t` 的最大值 |
| `INT64_MIN` | `int64_t` 的最小值 |
| `INT64_MAX` | `int64_t` 的最大值 |
| `UINT64_MAX` | `uint64_t` 的最大值 |

### 2. 最小宽度类型的极限值

| 宏名 | 作用 |
|------|------|
| `INT_LEAST8_MIN` | `int_least8_t` 的最小值 |
| `INT_LEAST8_MAX` | `int_least8_t` 的最大值 |
| `UINT_LEAST8_MAX` | `uint_least8_t` 的最大值 |
| `INT_LEAST16_MIN` | `int_least16_t` 的最小值 |
| ... | ... |

类似地，对于 `INT_LEAST32`、`INT_LEAST64`、`UINT_LEASTxx` 也有对应的宏。

### 3. 快速类型的极限值

| 宏名 | 作用 |
|------|------|
| `INT_FAST8_MIN` | `int_fast8_t` 的最小值 |
| `INT_FAST8_MAX` | `int_fast8_t` 的最大值 |
| `UINT_FAST8_MAX` | `uint_fast8_t` 的最大值 |
| `INT_FAST16_MIN` | `int_fast16_t` 的最小值 |
| ... | ... |

### 4. 指针类型的极限值

| 宏名 | 作用 |
|------|------|
| `INTPTR_MIN` | `intptr_t` 的最小值 |
| `INTPTR_MAX` | `intptr_t` 的最大值 |
| `UINTPTR_MAX` | `uintptr_t` 的最大值 |

### 5. 最大宽度类型的极限值

| 宏名 | 作用 |
|------|------|
| `INTMAX_MIN` | `intmax_t` 的最小值 |
| `INTMAX_MAX` | `intmax_t` 的最大值 |
| `UINTMAX_MAX` | `uintmax_t` 的最大值 |

### 6. 其他宏

| 宏名 | 作用 |
|------|------|
| `PTRDIFF_MIN` | `ptrdiff_t` 的最小值（定义在 `<cstddef>`，但在 `<cstdint>` 中也可能提供） |
| `PTRDIFF_MAX` | `ptrdiff_t` 的最大值 |
| `SIZE_MAX` | `size_t` 的最大值（定义在 `<cstddef>`） |
| `SIG_ATOMIC_MIN` | `sig_atomic_t` 的最小值（定义在 `<csignal>`） |
| `SIG_ATOMIC_MAX` | `sig_atomic_t` 的最大值 |
| `WCHAR_MIN` | `wchar_t` 的最小值（定义在 `<cwchar>`） |
| `WCHAR_MAX` | `wchar_t` 的最大值 |
| `WINT_MIN` | `wint_t` 的最小值（定义在 `<cwchar>`） |
| `WINT_MAX` | `wint_t` 的最大值 |

**注意：** 这些宏的值是实现定义的，例如 `SIZE_MAX` 通常为 `4294967295`（32 位系统）或 `18446744073709551615`（64 位系统）。

---

## 三、模板声明

`<cstdint>` 不包含任何 C++ 模板。

---
