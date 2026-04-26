## `<ctime>` 头文件详解

`<ctime>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**时间与日期处理**的函数、类型和宏。它支持获得当前日历时间、将时间分解为年/月/日/时/分/秒等字段，以及格式化输出时间字符串。该头文件是 C++ 中处理系统时间的基础接口。

---

## 一、函数详解

### 1. 时间获取函数

#### `std::time`

**函数原型：**
```cpp
std::time_t time(std::time_t* arg);
```

**作用：** 返回当前系统日历时间（自 1970‑01‑01 00:00:00 UTC 起的秒数）。若 `arg` 不为 `NULL`，也会将结果写入 `arg` 指向的变量。

**返回值：** 成功返回 `std::time_t` 值，失败返回 `(std::time_t)-1`。

**示例：**
```cpp
#include <ctime>
#include <cstdio>
int main() {
    std::time_t now = std::time(nullptr);
    std::printf("Seconds since epoch: %ld\n", now);
}
```

---

#### `std::clock`

**函数原型：**
```cpp
std::clock_t clock();
```

**作用：** 返回程序从启动开始消耗的处理器时间（近似值），常用于性能测量。

**返回值：** 成功返回 `std::clock_t` 值，失败返回 `(std::clock_t)-1`。除以 `CLOCKS_PER_SEC` 可换算为秒。

**示例：**
```cpp
std::clock_t start = std::clock();
// do work...
std::clock_t end = std::clock();
double cpu_sec = static_cast<double>(end - start) / CLOCKS_PER_SEC;
```

---

### 2. 时间转换函数

#### `std::gmtime` / `std::localtime`

**函数原型：**
```cpp
std::tm* gmtime(const std::time_t* timep);
std::tm* localtime(const std::time_t* timep);
```

**作用：** 将 `std::time_t` 日历时间分解为 `std::tm` 结构体。`gmtime` 返回 UTC 时间，`localtime` 返回本地时间。

**返回值：** 指向静态 `struct tm` 对象的指针（**非线程安全**）。失败返回 `NULL`。

**线程安全提示：** 使用静态缓冲区，**不可重入**。多线程环境下推荐使用 `gmtime_r` 或 `localtime_r`（POSIX）。

---

#### `std::mktime`

**函数原型：**
```cpp
std::time_t mktime(std::tm* timeptr);
```

**作用：** 将本地 `std::tm` 结构体转换为 `std::time_t` 日历时间，并自动调整 `tm_wday`、`tm_yday` 以及其他可能越界的字段。

**返回值：** 成功返回 `std::time_t`，失败返回 `(std::time_t)-1`。

---

### 3. 时间格式化函数

#### `std::asctime`

**函数原型：**
```cpp
char* asctime(const std::tm* timeptr);
```

**作用：** 将 `tm` 结构体转换为固定格式的字符串（例如 `"Wed Jun 30 21:49:08 1993\n"`）。

**返回值：** 指向静态字符数组的指针（**非线程安全**）。失败可能返回 `NULL`。

**线程安全提示：** 使用静态缓冲区，不可重入。建议使用 `std::strftime` 代替。

---

#### `std::ctime`

**函数原型：**
```cpp
char* ctime(const std::time_t* timep);
```

**作用：** 将 `time_t` 日历时间转换为字符串，等价于 `asctime(localtime(timep))`。

**返回值：** 指向静态字符数组的指针（**非线程安全**）。失败返回 `NULL`。

**线程安全提示：** 同 `asctime`，不可重入。

---

#### `std::strftime`

**函数原型：**
```cpp
size_t strftime(char* s, size_t max, const char* format, const std::tm* timeptr);
```

**作用：** 根据格式字符串 `format` 将 `tm` 结构体格式化为自定义字符串，写入 `s`（长度限制 `max`）。

**返回值：** 返回写入 `s` 的字符数（不含终止符），若超过 `max` 则返回 `0`。

**示例：**
```cpp
char buf[100];
std::time_t t = std::time(nullptr);
std::tm* tm = std::localtime(&t);
std::strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", tm);
```

**支持的常用格式说明符：**

| 格式符 | 含义                | 示例        |
|--------|---------------------|-------------|
| `%Y`   | 四位年份            | 2024        |
| `%m`   | 月份（01‑12）       | 04          |
| `%d`   | 日（01‑31）         | 26          |
| `%H`   | 小时（00‑23）       | 21          |
| `%M`   | 分钟（00‑59）       | 05          |
| `%S`   | 秒（00‑59）         | 08          |
| `%A`   | 完整星期名称        | Sunday      |
| `%a`   | 缩写星期名称        | Sun         |
| `%B`   | 完整月份名称        | April       |
| `%b`   | 缩写月份名称        | Apr         |
| `%p`   | AM/PM 标记          | PM          |
| `%Z`   | 时区名称            | CST         |

---

### 4. 其他函数（C++11 起）

#### `std::difftime`

**函数原型：**
```cpp
double difftime(std::time_t end, std::time_t begin);
```

**作用：** 返回 `end - begin` 的秒数（`double` 类型），用以计算时间差。

**示例：**
```cpp
std::time_t start = std::time(nullptr);
// ...
std::time_t end = std::time(nullptr);
double diff = std::difftime(end, start);
```

---

## 二、结构体详解

### `struct tm`

**定义：**
```cpp
struct tm {
    int tm_sec;    // 秒 [0,60]（含闰秒）
    int tm_min;    // 分 [0,59]
    int tm_hour;   // 时 [0,23]
    int tm_mday;   // 日 [1,31]
    int tm_mon;    // 月 [0,11]（0 = 一月）
    int tm_year;   // 年（1900 年起，如 124 表示 2024 年）
    int tm_wday;   // 星期 [0,6]（0 = 星期日）
    int tm_yday;   // 年中的天数 [0,365]
    int tm_isdst;  // 夏令时标志：>0、0、<0
};
```

**作用：** 存储分解后的日历时间（本地或 UTC）。许多字段自动通过 `mktime` 归一化。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `CLOCKS_PER_SEC` | 每秒的 `clock_t` 计数值，用于将 `clock()` 结果转换为秒 |
| `NULL` | 空指针常量（与 `<cstddef>` 相同） |
| `TIME_UTC`（C++11） | 用于 `timespec_get` 的时间基准（UTC） |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `std::time_t` | 算术类型，表示日历时间（通常为整型） |
| `std::clock_t` | 算术类型，表示处理器时间 |
| `struct tm` | 分解时间结构体 |
| `std::size_t` | 无符号整数类型（用于 `strftime` 的缓冲区长度） |

---

## 五、模板声明

`<ctime>` 是纯 C 风格头文件，不包含 C++ 模板。

---
