# C 标准库常用头文件

以下为 ISO C 标准定义的 C 标准库头文件（`.h` 形式），在 C++ 中通常推荐使用对应的 `<c...>` 版本（如 `<cstdio>`），以将名称放入 `std` 命名空间并保持与 C 的兼容性。这些头文件提供最基础的输入输出、字符串处理、数学运算、内存管理、时间日期等功能，是所有 C/C++ 程序的基石。

---

## 输入输出

**<stdio.h>**  
提供标准输入输出函数：`printf`、`scanf`、`fopen`、`fclose`、`fread`、`fwrite`、`fgets`、`fputs`、`fflush` 等，以及文件指针 `FILE` 类型。  
典型场景：控制台交互、文件读写、格式化输出。  
现代 C++ 对应：`<iostream>`（`std::cin`、`std::cout`）、`<fstream>`（`std::ifstream`、`std::ofstream`）、`<iomanip>`（格式化）。  
头文件依赖：通常独立。

---

## 字符串与字符处理

**<string.h>**  
提供 C 风格字符串操作函数：`strlen`、`strcpy`、`strncpy`、`strcat`、`strcmp`、`strstr`、`memcpy`、`memmove`、`memset` 等。  
典型场景：处理以 `'\0'` 结尾的字符数组，内存块操作。  
现代 C++ 对应：`<string>`（`std::string`）提供更安全、更便捷的字符串类，以及 `<algorithm>` 中的通用算法。  
头文件依赖：通常独立。

**<ctype.h>**  
提供字符分类与转换函数：`isalpha`、`isdigit`、`isalnum`、`isspace`、`toupper`、`tolower` 等。  
典型场景：判断字符类型或进行大小写转换。  
现代 C++ 对应：`<cctype>` 中的同名函数，或 C++ 中的 `std::isalpha` 等（定义于 `<cctype>`）。  
头文件依赖：通常独立。

---

## 数学函数

**<math.h>**  
提供数学运算函数：`sin`、`cos`、`tan`、`sqrt`、`pow`、`exp`、`log`、`fabs`、`ceil`、`floor` 等，以及数学常量如 `M_PI`（非标准，但常见）。  
典型场景：科学计算、几何运算、统计分析。  
现代 C++ 对应：`<cmath>` 中的同名函数，C++11 起还有 `std::hypot`、`std::erf` 等更丰富的数学函数。  
头文件依赖：通常需链接数学库 `-lm`（Linux）。

---

## 通用工具

**<stdlib.h>**  
提供内存分配与释放（`malloc`、`calloc`、`realloc`、`free`）、程序终止（`exit`、`abort`）、环境控制（`system`、`getenv`）、随机数（`rand`、`srand`）、字符串转换（`atoi`、`atol`、`strtol`、`strtod`）等。  
典型场景：动态内存管理、进程控制、整数/浮点转换。  
现代 C++ 对应：`<cstdlib>` 中的函数，但内存管理推荐使用 `new`/`delete` 或智能指针（`<memory>`），随机数推荐 `<random>` 库，字符串转换推荐 `<string>` 中的 `std::stoi` 等。  
头文件依赖：通常独立。

---

## 时间与日期

**<time.h>**  
提供时间获取与处理函数：`time`、`clock`、`difftime`、`strftime`、`localtime`、`gmtime`、`mktime` 等，以及 `time_t`、`struct tm` 类型。  
典型场景：获取当前时间、程序运行耗时、格式化日期字符串。  
现代 C++ 对应：`<chrono>` 提供高精度时钟和时长计算，`<ctime>` 保留 C 风格函数，C++20 引入 `<calendar>` 和 `<timezone>` 支持。  
头文件依赖：通常独立。

---

## 错误处理与断言

**<assert.h>**  
提供 `assert` 宏，在调试时检查条件，条件为假时终止程序并输出错误信息。  
典型场景：在开发阶段验证不变量、前置条件。  
现代 C++ 对应：C++ 中仍然可使用 `assert`，也可使用 `static_assert` 进行编译期断言，或自定义异常处理。  
头文件依赖：通常独立。

**<errno.h>**  
定义全局变量 `errno` 及错误码宏（如 `EDOM`、`ERANGE`、`EINVAL` 等）。  
典型场景：系统调用或库函数失败时获取详细错误原因。  
现代 C++ 对应：C++ 中仍可使用 `errno`，但更推荐通过异常（`std::system_error`）或返回错误码进行错误处理。  
头文件依赖：通常独立。

---

## 类型与极限

**<limits.h>**  
定义各种整数类型的取值范围，如 `INT_MAX`、`INT_MIN`、`CHAR_BIT` 等。  
典型场景：确保数值不溢出，跨平台可移植性。  
现代 C++ 对应：`<climits>` 保留同名宏，C++ 还提供 `<limits>` 中的 `std::numeric_limits` 模板类，以类型安全的方式获取极限值。  
头文件依赖：通常独立。

**<float.h>**  
定义浮点类型的取值范围和精度，如 `FLT_MAX`、`DBL_EPSILON` 等。  
典型场景：浮点运算的精度控制、比较时的容差设置。  
现代 C++ 对应：`<cfloat>` 保留同名宏，同样推荐使用 `<limits>` 中的 `std::numeric_limits<float>::epsilon()` 等。  
头文件依赖：通常独立。

**<stddef.h>**  
定义 `size_t`、`ptrdiff_t`、`NULL` 和 `offsetof` 宏。  
典型场景：表示大小、指针差、空指针、结构体成员偏移量。  
现代 C++ 对应：`<cstddef>` 保留这些定义，C++ 中 `size_t` 和 `ptrdiff_t` 位于 `std` 命名空间，`nullptr` 代替 `NULL`。  
头文件依赖：通常独立。

---

## 宽字符与多字节

**<wchar.h>**  
提供宽字符 I/O 和字符串处理函数：`wprintf`、`wscanf`、`fgetwc`、`fputwc`、`wcscpy`、`wcslen` 等，以及 `wchar_t` 类型。  
典型场景：处理 Unicode 或多语言环境下的文本。  
现代 C++ 对应：`<cwchar>` 保留 C 风格函数，C++ 中推荐使用 `std::wstring`（`<string>`）和 `std::wcout`、`std::wifstream` 等。  
头文件依赖：通常依赖 `<stdio.h>` 中定义的类型。

**<wctype.h>**  
提供宽字符分类与转换函数：`iswalpha`、`iswdigit`、`towupper`、`towlower` 等。  
典型场景：对宽字符进行类型判断和大小写转换。  
现代 C++ 对应：`<cwctype>` 保留，C++ 中可通过 `std::ctype` 或 `std::iswalpha`（`<cwctype>`）等使用。  
头文件依赖：通常独立。

---

## 非局部跳转

**<setjmp.h>**  
提供 `setjmp` 和 `longjmp` 函数，用于非局部跳转，绕过常规函数调用栈。  
典型场景：错误恢复、异常处理（在 C 语言中模拟异常）。  
现代 C++ 对应：C++ 中应使用异常机制（`try`/`catch`），`setjmp`/`longjmp` 与 C++ 对象析构不兼容，应避免使用。  
头文件依赖：通常独立。

---

## 信号处理

**<signal.h>**  
提供信号处理函数：`signal` 和 `raise`，以及信号宏（`SIGINT`、`SIGTERM`、`SIGSEGV` 等）。  
典型场景：捕捉程序中断、段错误等信号，执行清理操作。  
现代 C++ 对应：`<csignal>` 保留，但现代 C++ 程序更推荐使用 `std::signal`（需注意限制），或使用 C++ 标准库中的 `<atomic>` 和 `<thread>` 进行线程安全的信号处理，但信号机制在 C++ 中仍保留。  
头文件依赖：通常独立。

---

## 总括说明

- **定位**：C 标准库头文件提供了最底层的、跨平台的 C 语言基础功能，是所有 C/C++ 程序的基础依赖。它们的设计偏重过程式和直接的系统调用，适合对性能要求极高或需要与 C 代码交互的场景。
- **现代 C++ 替代方案**：
  - **输入输出**：`<iostream>`、`<fstream>` 提供类型安全、面向对象的流操作，避免 `printf`/`scanf` 的格式字符串错误。
  - **字符串**：`<string>` 提供 `std::string` 类，自动管理内存，支持运算符重载，比 C 字符串更安全。
  - **内存管理**：推荐使用 `new`/`delete` 或智能指针（`<memory>`），避免 `malloc`/`free` 的繁琐和错误。
  - **时间**：`<chrono>` 提供类型安全、高精度的时间点与时长，支持编译期单位检查。
  - **随机数**：`<random>` 提供高质量的随机数引擎和分布，替代 `rand`。
  - **错误处理**：C++ 异常机制（`try`/`catch`）更清晰、更安全，避免错误码传播的复杂性。
  - **类型极限**：`<limits>` 模板类提供类型安全的极限值查询。
  - **宽字符**：`std::wstring` 和相应的流对象更适合处理多语言文本。
  - **数学函数**：`<cmath>` 在 C++ 中提供重载，支持浮点类型自动匹配，并增加了更多实用函数。
- **与 C++ 标准库的关系**：C++ 标准库中对应头文件（`<c...>`）将 C 库函数放入 `std` 命名空间，并可能提供重载或更安全的版本（如 `<cmath>` 中的重载）。在 C++ 代码中，通常推荐包含 `<c...>` 而非 `<...h>`，以便与 C++ 命名空间和类型系统更好地集成。
- **跨平台性**：所有 C 标准库头文件在所有符合 ISO C 标准的平台上都可用，是跨平台开发的基石。

使用 C 标准库头文件可以编写高度可移植的代码，但需要注意手动管理资源、类型安全等问题。现代 C++ 开发建议优先使用 C++ 标准库提供的类型安全、自动资源管理的替代方案，以降低出错概率并提高代码可读性。