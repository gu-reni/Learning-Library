## `<cwchar>` 头文件详解

`<cwchar>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**宽字符和多字节字符**的处理功能。它涵盖了宽字符输入输出、字符串操作、转换函数等，是对 `<cstdio>` 和 `<cstring>` 等头文件的宽字符补充。宽字符类型 `wchar_t` 用于表示 Unicode 或本地化的多字节字符，在不同平台下宽度可能不同（Windows 通常为 2 字节 UTF‑16，Linux 通常为 4 字节 UTF‑32）。

---

## 一、函数详解

### 1. 宽字符输入输出

#### `fgetwc` / `getwc` / `getwchar`

**函数原型：**
```cpp
wint_t fgetwc(FILE* stream);
wint_t getwc(FILE* stream);
wint_t getwchar(void);
```

**作用：** 从输入流（`stream` 或标准输入）中读取一个宽字符，并将其转换为 `wint_t` 类型（可保存 `WEOF`）。

**返回值：** 成功返回读取的宽字符（转换为 `wint_t`），失败或文件结束时返回 `WEOF`。

**示例用法：**
```cpp
#include <cwchar>
#include <cstdio>

int main() {
    wint_t ch = getwchar();
    if (ch != WEOF) putwchar(ch);
    return 0;
}
```

**实现原理：** 调用底层文件读取函数，根据当前语言环境（locale）将多字节序列转换为宽字符。

**线程安全提示：** 若无显式同步，对同一流的并发操作不安全。

---

#### `fputwc` / `putwc` / `putwchar`

**函数原型：**
```cpp
wint_t fputwc(wchar_t wc, FILE* stream);
wint_t putwc(wchar_t wc, FILE* stream);
wint_t putwchar(wchar_t wc);
```

**作用：** 将一个宽字符写入输出流。

**返回值：** 成功返回写入的宽字符，失败返回 `WEOF`。

---

#### `ungetwc`

**函数原型：**
```cpp
wint_t ungetwc(wint_t wc, FILE* stream);
```

**作用：** 将一个宽字符“放回”流中，供后续读取。

**返回值：** 成功返回 `wc`，失败返回 `WEOF`。

---

### 2. 宽字符字符串函数

这些函数是 `<cstring>` 中窄字符串函数的宽字符版本，以 `wcs` 前缀代替 `str`。

| 函数 | 作用 |
|------|------|
| `wcslen(const wchar_t* s)` | 返回宽字符串长度 |
| `wcscpy(wchar_t* dest, const wchar_t* src)` | 复制宽字符串 |
| `wcsncpy(wchar_t* dest, const wchar_t* src, size_t n)` | 复制最多 n 个宽字符 |
| `wcscat(wchar_t* dest, const wchar_t* src)` | 连接宽字符串 |
| `wcsncat(wchar_t* dest, const wchar_t* src, size_t n)` | 连接最多 n 个宽字符 |
| `wcscmp(const wchar_t* lhs, const wchar_t* rhs)` | 比较宽字符串 |
| `wcsncmp(const wchar_t* lhs, const wchar_t* rhs, size_t n)` | 比较最多 n 个宽字符 |
| `wcschr(const wchar_t* s, wchar_t c)` | 查找宽字符 |
| `wcsrchr(const wchar_t* s, wchar_t c)` | 反向查找宽字符 |
| `wcspbrk(const wchar_t* s, const wchar_t* accept)` | 查找任意字符 |
| `wcsstr(const wchar_t* haystack, const wchar_t* needle)` | 查找子串 |
| `wcstok(wchar_t* s, const wchar_t* delim, wchar_t** ptr)` | 令牌化宽字符串（线程安全版本） |
| `wmemcpy(wchar_t* dest, const wchar_t* src, size_t n)` | 复制宽字符数组 |
| `wmemmove(wchar_t* dest, const wchar_t* src, size_t n)` | 移动宽字符数组（考虑重叠） |
| `wmemcmp(const wchar_t* lhs, const wchar_t* rhs, size_t n)` | 比较宽字符数组 |
| `wmemset(wchar_t* s, wchar_t c, size_t n)` | 填充宽字符数组 |

**示例：**
```cpp
#include <cwchar>
#include <cstdio>

int main() {
    wchar_t buf[100];
    wcscpy(buf, L"Hello");
    wcscat(buf, L" 宽字符");
    wprintf(L"%ls\n", buf);
    return 0;
}
```

---

### 3. 转换函数

#### `mbsrtowcs` / `wcsrtombs`

**函数原型：**
```cpp
size_t mbsrtowcs(wchar_t* dest, const char** src, size_t len, mbstate_t* ps);
size_t wcsrtombs(char* dest, const wchar_t** src, size_t len, mbstate_t* ps);
```

**作用：** 将多字节字符串（`src`）转换为宽字符串（`dest`），或反之。它们是可重入版本，使用状态对象 `ps`。

**返回值：** 成功转换的字符数（不包括终止符），若出错返回 `(size_t)-1`。

**示例：**
```cpp
#include <cwchar>
#include <cstdio>
#include <clocale>

int main() {
    std::setlocale(LC_ALL, "zh_CN.UTF-8");
    const char* mbstr = "你好";
    wchar_t wbuf[10];
    mbstate_t state = {};
    size_t ret = mbsrtowcs(wbuf, &mbstr, 10, &state);
    if (ret != (size_t)-1)
        wprintf(L"转换得到 %ls\n", wbuf);
    return 0;
}
```

---

#### `wcstol` / `wcstoll` / `wcstoul` / `wcstoull` / `wcstof` / `wcstod` / `wcstold`

**函数原型：**
```cpp
long      wcstol(const wchar_t* nptr, wchar_t** endptr, int base);
long long wcstoll(const wchar_t* nptr, wchar_t** endptr, int base);
unsigned long      wcstoul(...);
unsigned long long wcstoull(...);
float     wcstof(const wchar_t* nptr, wchar_t** endptr);
double    wcstod(...);
long double wcstold(...);
```

**作用：** 将宽字符串转换为数值，类似于 `strtol` 等。

---

#### `swprintf` / `vswprintf`

**函数原型：**
```cpp
int swprintf(wchar_t* s, size_t n, const wchar_t* format, ...);
int vswprintf(wchar_t* s, size_t n, const wchar_t* format, va_list arg);
```

**作用：** 格式化宽字符串输出到缓冲区，与 `sprintf` 类似。

---

#### `wcsftime`

**函数原型：**
```cpp
size_t wcsftime(wchar_t* s, size_t maxsize, const wchar_t* format, const struct tm* timeptr);
```

**作用：** 根据格式 `format` 将 `tm` 结构中的时间信息格式化为宽字符串。

---

## 二、结构体详解

### `struct tm`

`<cwchar>` 引用了 `<ctime>` 中的 `struct tm`，用于时间转换。

### `mbstate_t`

**定义：** 实现定义的不透明类型，用于存储多字节到宽字符转换的状态（例如编码移位状态）。

**作用：** 在可重入的转换函数（如 `mbsrtowcs`、`wcsrtombs`）中保留转换状态，支持多线程安全。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `WEOF` | 与 `EOF` 类似的宽字符文件结束标志，类型为 `wint_t`。 |
| `WCHAR_MIN` / `WCHAR_MAX` | 类型 `wchar_t` 的最小/最大值（定义在 `<climits>`）。 |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `wchar_t` | 内建宽字符类型（C++ 关键字，但 `<cwchar>` 提供相关支持） |
| `wint_t` | 足够存储 `wchar_t` 和 `WEOF` 的整数类型 |
| `wctype_t` | 宽字符分类类型（用于 `iswctype`） |
| `wctrans_t` | 宽字符转换类型（用于 `towctrans`） |
| `mbstate_t` | 转换状态类型 |

---

## 五、模板声明

`<cwchar>` 是纯 C 风格头文件，不包含 C++ 模板。

---
