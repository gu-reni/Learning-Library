## `<cmath>` 头文件详解

`<cmath>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**数学函数**、**浮点数分类**和**类型泛型宏**。它涵盖了基本算术运算（如 `sqrt`、`pow`）、三角函数、双曲函数、指数对数函数、舍入函数、误差函数、伽马函数等。C++11/17/20 还增加了一批浮点环境控制和特殊函数（部分在 `<cfenv>` 或 `<cmath>` 中提供）。本文档以 C++17 标准为基础。

---

## 一、函数详解

以下函数均在 `std` 命名空间中，并且支持 `float`、`double`、`long double` 重载（C++11 起也有泛型版本）。

### 1. 基本算术运算

| 函数 | 作用 |
|------|------|
| `std::abs(int)` / `std::labs(long)` / `std::llabs(long long)` / `std::fabs(double)` | 绝对值（整数或浮点数） |
| `std::fmod(double x, double y)` | 浮点数取余（`x - trunc(x/y)*y`） |
| `std::remainder(double x, double y)` | IEC 60559 余数（舍入到最近） |
| `std::remquo(double x, double y, int* quo)` | 同 `remainder`，并将商存入 `quo` |
| `std::fma(double x, double y, double z)` | 乘加：`x*y+z`，一次舍入 |
| `std::fmax(double x, double y)` | 返回较大值（含 NaN 处理） |
| `std::fmin(double x, double y)` | 返回较小值 |
| `std::fdim(double x, double y)` | 正差：`max(x-y, 0.0)` |

---

### 2. 指数、对数和幂函数

| 函数 | 作用 |
|------|------|
| `std::exp(double x)` | 自然指数 `e^x` |
| `std::exp2(double x)` | 2 的 x 次幂 |
| `std::expm1(double x)` | `e^x - 1`（低精度时更准确） |
| `std::log(double x)` | 自然对数 `ln(x)` |
| `std::log10(double x)` | 常用对数 `log10(x)` |
| `std::log2(double x)` | 以 2 为底的对数 |
| `std::log1p(double x)` | `ln(1 + x)`（对接近 0 的 x 更精确） |
| `std::pow(double base, double exp)` | 幂函数 `base^exp` |
| `std::sqrt(double x)` | 平方根 |
| `std::cbrt(double x)` | 立方根（C++11） |
| `std::hypot(double x, double y)` | 欧几里得距离 `sqrt(x² + y²)`（避免溢出） |

---

### 3. 三角函数

| 函数 | 作用                          |
|------|-------------------------------|
| `std::sin(double x)` | 正弦（弧度）                 |
| `std::cos(double x)` | 余弦                         |
| `std::tan(double x)` | 正切                         |
| `std::asin(double x)` | 反正弦，返回值 `[-π/2, π/2]` |
| `std::acos(double x)` | 反余弦，返回值 `[0, π]`     |
| `std::atan(double x)` | 反正切，返回值 `(-π/2, π/2)` |
| `std::atan2(double y, double x)` | 从坐标 `(x,y)` 计算反正切（四象限），返回值 `(-π, π]` |

---

### 4. 双曲函数

| 函数 | 作用 |
|------|------|
| `std::sinh(double x)` | 双曲正弦 |
| `std::cosh(double x)` | 双曲余弦 |
| `std::tanh(double x)` | 双曲正切 |
| `std::asinh(double x)` | 反双曲正弦（C++11） |
| `std::acosh(double x)` | 反双曲余弦（C++11，要求 `x>=1`） |
| `std::atanh(double x)` | 反双曲正切（C++11，要求 `|x|<1`） |

---

### 5. 舍入与取整

| 函数 | 作用 |
|------|------|
| `std::ceil(double x)` | 向上取整（返回 ≥ x 的最小整数） |
| `std::floor(double x)` | 向下取整（返回 ≤ x 的最大整数） |
| `std::trunc(double x)` | 向零取整（丢弃小数部分） |
| `std::round(double x)` | 四舍五入到最近整数 |
| `std::lround(double x)` | 返回 `long` 类型（四舍五入） |
| `std::llround(double x)` | 返回 `long long` 类型 |
| `std::nearbyint(double x)` | 按当前舍入方向取整 |
| `std::rint(double x)` | 同 `nearbyint`，可能触发浮点异常 |
| `std::frexp(double x, int* exp)` | 返回 `[0.5, 1)` 尾数，并将指数存入 `exp` |
| `std::ldexp(double x, int exp)` | 与 `frexp` 逆运算：`x * 2^exp` |
| `std::modf(double x, double* intpart)` | 拆分整数部分和小数部分 |
| `std::scalbn(double x, int n)` / `scalbln` | 乘 `FLT_RADIX` 的 n 次幂 |

---

### 6. 误差函数与伽马函数

| 函数 | 作用 |
|------|------|
| `std::erf(double x)` | 误差函数 `2/√π ∫₀ˣ e^{-t²} dt` |
| `std::erfc(double x)` | 互补误差函数 `1 - erf(x)` |
| `std::tgamma(double x)` | 伽马函数 `Γ(x)` |
| `std::lgamma(double x)` | 伽马函数的自然对数 `ln|Γ(x)|`，并通过 `signgam` 全局变量返回符号 |

---

### 7. 浮点比较与分类（宏/函数）

| 函数（C++11） | 作用 |
|----------------|------|
| `std::fpclassify(double x)` | 分类为 `FP_NORMAL`、`FP_SUBNORMAL`、`FP_ZERO`、`FP_INFINITE`、`FP_NAN` |
| `std::isfinite(double x)` | 是否为有限值 |
| `std::isinf(double x)` | 是否为无穷大 |
| `std::isnan(double x)` | 是否为 NaN |
| `std::isnormal(double x)` | 是否为正常值 |
| `std::signbit(double x)` | 符号是否为负（包括 `-0.0`） |
| `std::isgreater` 等 | 安全比较，不会因 NaN 导致异常 |

---

## 二、结构体详解

`<cmath>` 本身不定义结构体。它使用标准库中的 `double`、`float`、`long double` 等基本类型。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `HUGE_VAL` / `HUGE_VALF` / `HUGE_VALL` | 当函数结果溢出时返回的值（正无穷） |
| `INFINITY` | 正无穷常量（C++11，如果实现支持） |
| `NAN` | 静默 NaN（C++11，如果支持） |
| `MATH_ERRNO` / `MATH_ERREXCEPT` | 用于 `std::math_errhandling` 的常量 |
| `FP_FAST_FMA` / `FP_FAST_FMAF` / `FP_FAST_FMAL` | 表示 `fma` 函数是否快速 |
| `FP_ILOGB0` | `ilogb(0)` 返回的常量 |
| `FP_ILOGBNAN` | `ilogb(NaN)` 返回的常量 |
| `FP_INFINITE` / `FP_NAN` 等 | `fpclassify` 的返回值 |
| `math_errhandling` | 表示错误处理方式的宏（可用 `MATH_ERRNO` 或 `MATH_ERREXCEPT`） |

另外，许多实现宏（如 `M_PI`）**不是标准 C++**，但在 `<cmath>` 中可能可用（需要定义 `_USE_MATH_DEFINES`，属于扩展）。标准要求以 `std::numbers::pi`（C++20 `<numbers>`）替代。

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `double_t` | 至少与 `double` 一样宽的类型，用于 `float_t` 和 `double_t`（C++11，通常在 `<math.h>` 中） |
| `float_t` | 至少与 `float` 一样宽的类型 |

（注：`float_t` 和 `double_t` 在实际中很少直接使用。）

---

## 五、模板声明

C++11 起，`<cmath>` 提供了大量**类型泛型**模板函数，例如：
```cpp
template<class T> T abs(T x);                // 整数重载也在 <cstdlib>
template<class T> T sqrt(T x);
template<class T> T sin(T x);
// 等等
```
但这些模板主要与算术类型一起使用，并且通常由标准库提供重载而非用户自定义模板。

---
