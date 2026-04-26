## `<cuchar>` 头文件详解

`<cuchar>` 是 C++11 标准引入的头文件，提供了**UTF-16 和 UTF-32 字符**的处理功能，以及用于与多字节字符串相互转换的**C 风格转换函数**。它补充了 `<cwchar>` 中宽字符（`wchar_t`）的局限性，使程序能够直接处理固定宽度的 Unicode 编码单元（`char16_t` 和 `char32_t`）。该头文件是编写跨平台 Unicode 文本处理程序的重要工具。

---

## 一、函数详解

### 1. UTF-16 相关函数

#### `std::mbrtoc16`

**函数原型：**
```cpp
size_t mbrtoc16(char16_t* pc16, const char* s, size_t n, mbstate_t* ps);
```

**作用：** 将多字节序列（常为 UTF‑8）转换为单个 UTF‑16 编码单元（一个 `char16_t`）。一次可能消耗多个输入字节（若多字节字符对应代理对，则可能需要多次调用才能输出完整的两个 `char16_t`）。

**参数：**
- `pc16`：输出参数，存储转换得到的 `char16_t`，可为 `NULL`（仅用于计算字节数）。
- `s`：输入多字节字符串指针。
- `n`：最多读取 `s` 的字节数。
- `ps`：指向转换状态的指针（`mbstate_t`），用于处理移位状态或代理对。

**返回值：**
- `0`：遇到空字符（`'\0'`），输出也为 `0`（若 `pc16` 非空）。
- `1...n`：成功转换的字节数，并输出一个 `char16_t`。
- `-2`：剩余输入字节不足以构成完整的多字节字符，未输出。
- `-3`：下一个字符对应的 UTF‑16 编码需要两个代理项（例如 Emoji），当前输出了高代理，下次调用输出低代理。
- `-1`：编码错误，`errno` 设为 `EILSEQ`。

**示例用法：**
```cpp
#include <cuchar>
#include <cstdio>

int main() {
    const char* u8_str = u8"Hello 你好";
    mbstate_t state{};
    char16_t c16;
    int n = std::mbrtoc16(&c16, u8_str, 5, &state);
    if (n > 0) std::printf("First UTF-16 code unit: 0x%X\n", c16);
}
```

**实现原理：** 依赖于 `LC_CTYPE` 区域设置（locale），将输入多字节字符串解码为 Unicode 码点，再根据 UTF-16 规则编码为 `char16_t`。

**线程安全提示：** `mbstate_t` 应作为每个线程的独立状态对象，避免共享。若使用全局状态（不传递 `ps` 而用静态内部状态），则函数不可重入。

---

#### `std::c16rtomb`

**函数原型：**
```cpp
size_t c16rtomb(char* s, char16_t c16, mbstate_t* ps);
```

**作用：** 将单个 UTF‑16 编码单元（可能为代理对的一部分）转换为多字节序列（常为 UTF‑8）。

**参数：**
- `s`：输出多字节字符串缓冲区（至少 `MB_CUR_MAX` 字节），可为 `NULL`。
- `c16`：要转换的 UTF‑16 字符。
- `ps`：转换状态指针。

**返回值：**
- 成功：写入 `s` 的字节数（不包括结尾 `'\0'`）。
- 失败：`(size_t)-1`。

**示例：**
```cpp
char mb[8];
mbstate_t state{};
size_t len = std::c16rtomb(mb, u'€', &state);
```

---

### 2. UTF-32 相关函数

#### `std::mbrtoc32`

**函数原型：**
```cpp
size_t mbrtoc32(char32_t* pc32, const char* s, size_t n, mbstate_t* ps);
```

**作用：** 将多字节序列转换为单个 UTF‑32 编码单元（`char32_t`）。

**参数与返回值：** 类似 `mbrtoc16`，但输出始终是一个完整的 Unicode 码点（代理对不适用）。

#### `std::c32rtomb`

**函数原型：**
```cpp
size_t c32rtomb(char* s, char32_t c32, mbstate_t* ps);
```

**作用：** 将单个 UTF‑32 码点转换为多字节序列。

---

## 二、宏定义详解

`<cuchar>` 不定义任何宏。

---

## 三、类型定义

| 类型 | 说明 |
|------|------|
| `char16_t` | 用于 UTF‑16 编码单元的整数类型（C++11 内建） |
| `char32_t` | 用于 UTF‑32 编码单元的整数类型（C++11 内建） |
| `mbstate_t` | 多字节转换状态类型（在 `<cwchar>` 中定义，此头文件引用） |
| `size_t` | 无符号整数类型（来自 `<cstddef>`） |

---

## 四、结构体详解

`<cuchar>` 不定义任何结构体。

---

## 五、模板声明

`<cuchar>` 是纯 C 风格头文件，不包含 C++ 模板。

---
