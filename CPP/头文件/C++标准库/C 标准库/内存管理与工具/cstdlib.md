## `<cstdlib>` 头文件详解

`<cstdlib>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**通用工具**，包括：**字符串转换**（`atoi`、`strtol` 等）、**伪随机数生成**（`rand`、`srand`）、**内存管理**（`malloc`、`free` 等）、**程序控制**（`exit`、`abort`、`system`）、**环境访问**（`getenv`）、**排序与搜索**（`qsort`、`bsearch`）以及**多字节字符转换**等。在 C++ 中，大部分功能已被更安全的替代方案取代（如 `std::string` 转换、`std::random_device`、`new`/`delete`），但该头文件仍可用于兼容 C 代码或特定底层操作。

---

## 一、函数详解

### 1. 字符串转换函数

| 函数 | 作用 |
|------|------|
| `int atoi(const char* str)` | 将字符串转换为 `int`（忽略溢出） |
| `long atol(const char* str)` | 转换为 `long` |
| `long long atoll(const char* str)` | 转换为 `long long`（C++11） |
| `double atof(const char* str)` | 转换为 `double` |
| `long strtol(const char* str, char** endptr, int base)` | 按指定进制转换为 `long`，`endptr` 存储第一个无法转换的字符指针 |
| `long long strtoll(...)` | 转换为 `long long`（C++11） |
| `unsigned long strtoul(...)` | 转换为 `unsigned long` |
| `unsigned long long strtoull(...)` | 转换为 `unsigned long long`（C++11） |
| `float strtof(const char* str, char** endptr)` | 转换为 `float`（C++11） |
| `double strtod(...)` | 转换为 `double` |
| `long double strtold(...)` | 转换为 `long double`（C++11） |

**注意：** 这些函数是 C 风格的，建议在 C++ 中使用 `std::stoi`、`std::stol` 等（`<string>`）以获得异常处理和更简单的接口。

---

### 2. 伪随机数生成

| 函数 | 作用 |
|------|------|
| `int rand(void)` | 返回 0 到 `RAND_MAX`（包含）之间的伪随机整数 |
| `void srand(unsigned int seed)` | 设置随机数生成器的种子 |
| `int RAND_MAX` | 宏，表示 `rand()` 返回的最大值 |

**示例：**
```cpp
#include <cstdlib>
#include <ctime>
std::srand(std::time(nullptr));
int r = std::rand();
```

> **警告：** `rand()` 质量较低，C++11 推荐使用 `<random>` 库。

---

### 3. 动态内存管理

| 函数 | 作用 |
|------|------|
| `void* malloc(size_t size)` | 分配 `size` 字节未初始化的内存 |
| `void* calloc(size_t num, size_t size)` | 分配 `num * size` 字节并初始化为 0 |
| `void* realloc(void* ptr, size_t new_size)` | 调整已分配内存的大小 |
| `void free(void* ptr)` | 释放先前由 `malloc`/`calloc`/`realloc` 分配的内存 |

**注意：** 在 C++ 中应优先使用 `new`/`delete` 或智能指针，`malloc`/`free` 不会调用构造函数/析构函数。

---

### 4. 程序控制与环境

| 函数 | 作用 |
|------|------|
| `void abort(void)` | 异常终止程序（不调用析构函数，不刷新 I/O 缓冲区），引发 `SIGABRT` |
| `void exit(int status)` | 正常终止程序，刷新 I/O、调用 `atexit` 注册的函数 |
| `void _Exit(int status)`（C++11） | 立即终止，不调用 `atexit` 函数 |
| `void quick_exit(int status)`（C++11） | 快速终止，仅调用 `at_quick_exit` 注册的函数 |
| `int atexit(void (*func)(void))` | 注册程序正常退出时调用的函数 |
| `int at_quick_exit(void (*func)(void))`（C++11） | 注册快速退出时调用的函数 |
| `int system(const char* command)` | 调用系统 shell 执行命令 |
| `char* getenv(const char* name)` | 获取环境变量值，若不存在返回 `NULL` |

---

### 5. 排序与搜索

| 函数 | 作用 |
|------|------|
| `void qsort(void* base, size_t num, size_t size, int (*compar)(const void*, const void*))` | 快速排序算法 |
| `void* bsearch(const void* key, const void* base, size_t num, size_t size, int (*compar)(const void*, const void*))` | 二分查找 |

**注意：** 这些函数只能用于 C 风格的数据结构，在 C++ 中推荐使用 `std::sort`、`std::binary_search`。

---

### 6. 多字节与宽字符转换（部分）

| 函数 | 作用 |
|------|------|
| `int mblen(const char* s, size_t n)` | 获取多字节字符的字节长度 |
| `size_t mbstowcs(wchar_t* dest, const char* src, size_t len)` | 多字节字符串转换为宽字符串 |
| `size_t wcstombs(char* dest, const wchar_t* src, size_t len)` | 宽字符串转换为多字节字符串 |

---

### 7. 整型除法与余数

| 函数 | 作用 |
|------|------|
| `std::div_t div(int x, int y)` | 计算 `x / y` 和 `x % y`，结果在 `div_t` 结构中 |
| `std::ldiv_t ldiv(long x, long y)` | 同上，返回 `ldiv_t` |
| `std::lldiv_t lldiv(long long x, long long y)` | 同上（C++11） |

---

## 二、结构体详解

### `div_t`、`ldiv_t`、`lldiv_t`

**定义（简略）：**
```cpp
struct div_t {
    int quot;   // 商
    int rem;    // 余数
};
// ldiv_t 类似，成员为 long
// lldiv_t 成员为 long long
```

**作用：** 存储 `div`、`ldiv`、`lldiv` 函数返回的商和余数。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `NULL` | 空指针常量（与 `<cstddef>` 相同） |
| `RAND_MAX` | `rand()` 函数返回的最大值 |
| `EXIT_SUCCESS` | 传递给 `exit` 的成功状态码（通常为 `0`） |
| `EXIT_FAILURE` | 传递给 `exit` 的失败状态码（非零） |
| `MB_CUR_MAX` | 当前 locale 下多字节字符的最大字节数 |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `size_t` | 无符号整数，用于表示大小（也定义在 `<cstddef>`） |
| `wchar_t` | 宽字符类型（语言内建，但该头文件提供相关函数） |
| `div_t`、`ldiv_t`、`lldiv_t` | 上述结构体类型 |

---

## 五、模板声明

`<cstdlib>` 是纯 C 风格头文件，不包含 C++ 模板。

---
