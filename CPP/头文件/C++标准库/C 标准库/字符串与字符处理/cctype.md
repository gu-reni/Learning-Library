## `<cctype>` 头文件详解

`<cctype>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**字符分类**和**大小写转换**的函数。它根据当前 C 语言环境（locale）的 `LC_CTYPE` 类别对单个字符进行判断和转换。这些函数在处理文本输入、词法分析等场景中非常有用。

---

## 一、函数详解

以下函数均在 `std` 命名空间中，参数为 `int`，但实际应为 `unsigned char` 值或 `EOF`。若传入 `char` 类型，应转换为 `unsigned char` 以避免未定义行为（`char` 可能为负）。

### 1. 字符分类函数

| 函数 | 作用 |
|------|------|
| `isalnum(int ch)` | 检查字符是否为字母或数字 |
| `isalpha(int ch)` | 检查字符是否为字母 |
| `isblank(int ch)` | 检查字符是否为空白（空格或水平制表符） |
| `iscntrl(int ch)` | 检查字符是否为控制字符 |
| `isdigit(int ch)` | 检查字符是否为十进制数字 |
| `isgraph(int ch)` | 检查字符是否为可打印字符（非空格） |
| `islower(int ch)` | 检查字符是否为小写字母 |
| `isprint(int ch)` | 检查字符是否为可打印字符（包括空格） |
| `ispunct(int ch)` | 检查字符是否为标点符号 |
| `isspace(int ch)` | 检查字符是否为空白字符（空格、换行、回车、制表等） |
| `isupper(int ch)` | 检查字符是否为大写字母 |
| `isxdigit(int ch)` | 检查字符是否为十六进制数字 |

**参数：** 要测试的字符，以 `int` 类型传递，必须为 `unsigned char` 值或 `EOF`。

**返回值：** 若字符属于相应类别，返回非零值（通常为 `1`）；否则返回 `0`。

**示例用法：**
```cpp
#include <cctype>
#include <cstdio>

int main() {
    char c = 'A';
    if (std::isupper(c))
        std::printf("%c is uppercase\n", c);
}
```

**实现原理：** 通常通过查表（每个字符的比特位掩码）或调用语言环境函数实现，速度较快。

**线程安全提示：** 这些函数不修改任何外部状态，但若使用全局 locale（未调用 `std::locale::global`），它们依赖于全局的 `LC_CTYPE`。在多线程环境下同时调用 `setlocale` 会产生数据竞争。推荐使用 C++11 的 `std::isalpha`（定义在 `<locale>`）或确保不并发修改全局 locale。

---

### 2. 字符转换函数

| 函数 | 作用 |
|------|------|
| `tolower(int ch)` | 将大写字母转换为小写字母，其他字符不变 |
| `toupper(int ch)` | 将小写字母转换为大写字母，其他字符不变 |

**参数：** 要转换的字符（`int` 类型，应为 `unsigned char` 或 `EOF`）。

**返回值：** 转换后的字符（若适用），否则返回原字符。

**示例：**
```cpp
char c = 'b';
char upper = std::toupper(c);   // 'B'
```

**实现原理：** 同样通过查表或 locale 函数实现。

**线程安全提示：** 同分类函数，依赖全局 locale，多线程调用 `setlocale` 时需要同步。

---

## 二、宏定义详解

`<cctype>` 不定义任何宏。相关常量（如 `EOF`）定义在 `<cstdio>` 中。

---

## 三、类型定义

`<cctype>` 不定义任何类型。

---

## 四、模板声明

`<cctype>` 是纯 C 风格头文件，不包含 C++ 模板。

---
