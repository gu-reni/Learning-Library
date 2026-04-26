## `<cstdio>` 头文件详解

`<cstdio>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**标准输入输出（I/O）**的函数、类型和宏。它涵盖了文件操作、标准输入输出流（`stdin`、`stdout`、`stderr`）、格式化输入输出、字符 I/O、行 I/O 以及文件定位等功能。虽然 C++ 提供了更安全的 `<iostream>` 库，但 `<cstdio>` 在底层文件操作、性能敏感场景以及与 C 代码交互时仍然广泛使用。

---

## 一、函数详解

### 1. 文件操作函数

| 函数 | 作用 |
|------|------|
| `std::fopen` | 打开文件并返回文件指针 |
| `std::fclose` | 关闭文件 |
| `std::freopen` | 重定向已打开的文件指针到新文件 |
| `std::fflush` | 刷新输出缓冲区 |

---

#### `std::fopen`

**函数原型：**
```cpp
std::FILE* fopen(const char* filename, const char* mode);
```

**作用：** 打开由 `filename` 指定的文件，并以 `mode` 指定的模式打开。

**参数：**
- `filename`：文件路径。
- `mode`：打开模式字符串，常用值：
  - `"r"`：只读（文件必须存在）
  - `"w"`：只写（文件不存在则创建，存在则清空）
  - `"a"`：追加（文件不存在则创建）
  - `"rb"` / `"wb"` / `"ab"`：二进制模式的读/写/追加
  - `"r+"`：读写（文件必须存在）
  - `"w+"`：读写（文件不存在则创建，存在则清空）
  - `"a+"`：读写追加（文件不存在则创建）

**返回值：** 成功返回 `std::FILE*` 指针，失败返回 `NULL`。

**示例：**
```cpp
#include <cstdio>
int main() {
    std::FILE* fp = std::fopen("test.txt", "w");
    if (fp) {
        std::fprintf(fp, "Hello\n");
        std::fclose(fp);
    }
}
```

**实现原理：** 调用底层操作系统 API（如 POSIX `open`）打开文件，并关联一个 `FILE` 结构体用于缓冲。

**线程安全提示：** 对同一文件指针的并发操作不安全，需要外部同步。

---

#### `std::fclose`

**函数原型：**
```cpp
int fclose(std::FILE* stream);
```

**作用：** 关闭文件流，刷新缓冲区并释放关联的资源。

**返回值：** 成功返回 `0`，失败返回 `EOF`。

---

#### `std::fflush`

**函数原型：**
```cpp
int fflush(std::FILE* stream);
```

**作用：** 将输出流的缓冲区数据写入文件。若 `stream` 为 `NULL`，则刷新所有输出流。

**返回值：** 成功返回 `0`，失败返回 `EOF`。

---

### 2. 格式化输入输出

| 函数 | 作用 |
|------|------|
| `std::printf` / `std::fprintf` / `std::sprintf` / `std::snprintf` | 格式化输出 |
| `std::scanf` / `std::fscanf` / `std::sscanf` | 格式化输入 |
| `std::vprintf` / `std::vfprintf` / `std::vsprintf` / `std::vsnprintf` | 可变参数版本 |

---

#### `std::printf` 族

**函数原型：**
```cpp
int printf(const char* format, ...);
int fprintf(std::FILE* stream, const char* format, ...);
int sprintf(char* buffer, const char* format, ...);
int snprintf(char* buffer, size_t bufsz, const char* format, ...);
```

**作用：** 根据格式字符串将数据输出到标准输出、文件或字符串缓冲区。`snprintf` 限制写入长度，防止溢出。

**返回值：** 成功返回写入的字符数（不含终止符），失败返回负值。

**常用格式说明符：**

| 格式符 | 含义 |
|--------|------|
| `%d` / `%i` | 有符号十进制整数 |
| `%u` | 无符号十进制整数 |
| `%x` / `%X` | 十六进制整数 |
| `%o` | 八进制整数 |
| `%f` / `%lf` | 浮点数（十进制） |
| `%e` / `%E` | 科学计数法 |
| `%g` / `%G` | 自动选择 `%f` 或 `%e` |
| `%c` | 单个字符 |
| `%s` | 字符串 |
| `%p` | 指针 |
| `%%` | 百分号本身 |

---

#### `std::scanf` 族

**函数原型：**
```cpp
int scanf(const char* format, ...);
int fscanf(std::FILE* stream, const char* format, ...);
int sscanf(const char* buffer, const char* format, ...);
```

**作用：** 从标准输入、文件或字符串中根据格式读取数据。

**返回值：** 成功匹配并赋值的参数个数，失败返回 `EOF`。

**注意事项：** 这些函数不安全（不支持长度限制），在 C++ 中推荐使用 `std::cin` 或 `std::getline`。

---

### 3. 字符输入输出

| 函数 | 作用 |
|------|------|
| `std::fgetc` / `std::getc` / `std::getchar` | 读取一个字符 |
| `std::fputc` / `std::putc` / `std::putchar` | 写入一个字符 |
| `std::ungetc` | 将一个字符放回流中 |
| `std::fgets` / `std::gets`（已废弃） | 读取一行字符串 |
| `std::fputs` / `std::puts` | 写入字符串 |

---

#### `std::fgets`

**函数原型：**
```cpp
char* fgets(char* str, int count, std::FILE* stream);
```

**作用：** 从 `stream` 读取最多 `count-1` 个字符，直到换行符或文件结束，并添加终止符 `\0`。

**返回值：** 成功返回 `str`，失败或文件结束返回 `NULL`。

**示例：**
```cpp
char line[256];
if (std::fgets(line, sizeof(line), stdin)) {
    // 处理 line
}
```

---

### 4. 文件定位与错误处理

| 函数 | 作用 |
|------|------|
| `std::fseek` | 移动文件位置指示器 |
| `std::ftell` | 获取当前文件位置 |
| `std::rewind` | 将文件位置重置到开头 |
| `std::feof` | 检查文件结束标志 |
| `std::ferror` | 检查文件错误标志 |
| `std::perror` | 打印错误信息（基于 `errno`） |
| `std::clearerr` | 清除错误和 EOF 标志 |

---

## 二、结构体详解

### `std::FILE`

**定义：** 不透明结构体类型，表示一个文件流。其具体成员由实现定义，不应直接访问。

**作用：** 通过 `fopen` 等函数获得，用于所有后续文件 I/O 操作。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `EOF` | 整数常量（通常 `-1`），表示文件结束或错误 |
| `NULL` | 空指针常量 |
| `BUFSIZ` | `setbuf` 使用的缓冲区大小 |
| `FILENAME_MAX` | 足够存储最长文件名的数组大小 |
| `FOPEN_MAX` | 同时打开的文件最大数量（至少 8） |
| `TMP_MAX` | `tmpnam` 生成唯一文件名的最大次数 |
| `L_tmpnam` | `tmpnam` 所需缓冲区大小 |
| `SEEK_SET` | `fseek` 从文件开头定位 |
| `SEEK_CUR` | `fseek` 从当前位置定位 |
| `SEEK_END` | `fseek` 从文件末尾定位 |
| `_IOFBF` | 全缓冲模式 |
| `_IOLBF` | 行缓冲模式 |
| `_IONBF` | 无缓冲模式 |
| `stdin` / `stdout` / `stderr` | 标准输入、输出、错误流（类型 `FILE*`） |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `std::FILE` | 不完整的文件流类型 |
| `std::fpos_t` | 用于记录文件位置的类型 |
| `std::size_t` | 无符号整数（来自 `<cstddef>`） |

---

## 五、模板声明

`<cstdio>` 是纯 C 头文件，不包含 C++ 模板。

---
