# std::string_view（C++17，C++20 增强）

**一句话定义**：`std::string_view` 是一个非拥有的、只读的字符串视图，它指向一段连续的字符序列（可以是 `std::string`、字符数组或字符串字面量），提供轻量级的字符串访问接口，避免不必要的拷贝，显著提升只读字符串操作的性能。

---

## 详细语法与用法

### 基本构造与访问

```cpp
#include <string_view>
#include <string>

// 从字符串字面量构造（不拷贝）
std::string_view sv1 = "hello world";      // 指向静态存储区的字符数组
std::string_view sv2("hello", 5);          // 指向 "hello"，长度 5

// 从 std::string 构造（不拷贝）
std::string str = "hello world";
std::string_view sv3 = str;                // 指向 str 的内部缓冲区

// 空视图
std::string_view sv4;                      // 默认构造，data() 可能为 nullptr，size() 为 0

// 访问字符
char ch = sv3[0];                          // 不检查边界，越界 UB
char ch2 = sv3.at(0);                      // 检查边界，越界抛出 std::out_of_range
char front = sv3.front();                  // 第一个字符
char back = sv3.back();                    // 最后一个字符
const char* data = sv3.data();             // 返回指针（不保证以 '\0' 结尾）
size_t len = sv3.size();                   // 长度
bool empty = sv3.empty();                  // 是否为空
```

### 修改视图（不修改底层数据）

```cpp
std::string_view sv = "hello world";
sv.remove_prefix(6);        // sv 变为 "world"
sv.remove_suffix(2);        // sv 变为 "wor"（移除最后 2 个字符）
sv = sv.substr(1, 2);       // sv 变为 "or"（substr 返回新的 string_view）
```

### 查找与比较

```cpp
std::string_view sv = "hello world";
size_t pos = sv.find("world");         // 6
size_t rpos = sv.rfind('o');           // 7（从右查找）
size_t first = sv.find_first_of("aeiou"); // 1（e）
size_t last = sv.find_last_of("aeiou");   // 7（o）
bool starts = sv.starts_with("hel");   // C++20，true
bool ends = sv.ends_with("rld");       // C++20，true
int cmp = sv.compare("hello world");   // 0（相等）
```

### 与 `std::string` 的互操作

```cpp
std::string_view sv = "hello";
// 隐式转换：string_view -> string（拷贝）
std::string str(sv);                   // 拷贝构造
std::string str2 = std::string(sv);    // 显式转换

// 从 string_view 构造 string（需要拷贝）
std::string str3(sv.data(), sv.size());

// 接收 string_view 参数的函数可以接受 string 和字面量
void process(std::string_view name) {
    std::cout << name << std::endl;
}
std::string s = "Alice";
process(s);          // OK，隐式构造 string_view
process("Bob");      // OK
```

### C++20 增强：`constexpr` 支持与 `starts_with`/`ends_with`

```cpp
// C++20 起 string_view 的几乎所有操作都是 constexpr
constexpr std::string_view sv = "constexpr string";
constexpr char first = sv.front();     // 'c'
constexpr auto sub = sv.substr(0, 9);  // "constexpr"
static_assert(sub == "constexpr");

// starts_with / ends_with
constexpr bool start = sv.starts_with("const");   // true
constexpr bool end = sv.ends_with("ing");         // true
```

### 哈希支持

```cpp
#include <unordered_set>
#include <string_view>

std::unordered_set<std::string_view> views;  // C++20 起合法，但需注意生命周期
views.insert("hello");
views.insert("world");
```

> 注意：`std::hash<std::string_view>` 在 C++20 之前可能不可用，C++20 标准库提供。

---

## 底层原理与实现机制

### 1. 内存布局与大小

`std::string_view` 的典型实现只包含两个成员：

```cpp
class string_view {
    const char* data_;   // 指向字符序列的起始
    size_t size_;        // 字符序列的长度
};
```

在 64 位系统上，`sizeof(std::string_view) == 16`（两个 8 字节成员）。对比 `std::string` 通常为 32 字节（或更多，因包含指针、大小、容量、分配器等），`string_view` 极为轻量。

### 2. 非拥有语义

`string_view` **不管理**底层字符序列的生命周期。它仅是一个**观察者**，类似指针。这既是其性能优势的来源，也是使用中最大的风险源。拷贝 `string_view` 只复制指针和长度（O(1)），不分配内存，不复制字符数据。

### 3. 不保证以空字符结尾

`string_view::data()` 返回的指针**不保证**指向以 `'\0'` 结尾的字符串。因此，不能直接将其传递给期望 C 风格字符串（如 `printf("%s", sv.data())`）的函数，除非你明确知道底层存储包含空字符。

```cpp
std::string_view sv = "hello";        // 字面量隐含 '\0'，安全
std::string s = "world";
std::string_view sv2(s.data(), 3);    // 指向 "wor"，不保证后面有 '\0'
// printf("%s", sv2.data());          // 危险，可能越界
```

### 4. 构造与转换的开销

- 从 `const char*` 构造：需调用 `strlen` 计算长度（O(n)），除非提供长度参数。
- 从 `std::string` 构造：O(1)，只复制指针和大小（字符串长度已知）。
- 从 `std::string_view` 拷贝：O(1)。

因此，从 C 字符串字面量构造 `string_view` 有 O(n) 的开销（计算长度），但通常可接受；如果频繁使用，可考虑显式传递长度。

### 5. constexpr 支持（C++20）

C++20 中 `string_view` 的几乎所有成员函数都是 `constexpr`，允许在编译期进行字符串操作，适用于编译期哈希、静态断言等场景。

---

## 与相关特性的对比

### 1. `std::string_view` vs `const std::string&`

| 方面 | `const std::string&` | `std::string_view` |
|------|----------------------|---------------------|
| 是否拥有数据 | 否（引用） | 否（视图） |
| 可从 `const char*` 构造 | 需要临时 `std::string`（分配内存） | 直接，无分配 |
| 可从 `std::string` 构造 | 直接绑定 | 直接绑定 |
| 可从子串构造 | 需创建临时 `std::string` | 直接，O(1) |
| 需空字符结尾 | 是（`c_str()` 保证） | 否 |
| 隐式转换开销 | 可能分配内存 | 无分配 |
| 适用场景 | API 需要修改字符串或传递给期望 `std::string` 的函数 | 只读字符串参数，避免不必要的拷贝 |

**经验法则**：函数参数如果是只读字符串，优先使用 `std::string_view` 而不是 `const std::string&`。

### 2. `std::string_view` vs `const char*`

| 方面 | `const char*` | `std::string_view` |
|------|---------------|---------------------|
| 知道长度 | 需 `strlen`（O(n)） | 直接 `size()`（O(1)） |
| 支持字符串操作 | 需 C 函数（`strcmp`、`strstr` 等） | 成员函数（`find`、`substr` 等） |
| 空字符结尾要求 | 是 | 否 |
| 可存储子串 | 需指针 + 长度分开管理 | 一体化 |
| 安全性 | 易错（越界风险） | 更安全（带长度） |

### 3. `std::string_view` vs `std::span<char>`

`std::span`（C++20）是连续序列的视图，可泛化到任何类型。`std::string_view` 专门为字符序列优化，提供字符串特有方法（查找、比较、前缀/后缀检查等）。两者不冲突，`string_view` 可看作 `span<const char>` 的特化版本，但增加了字符串语义。

---

## 常见面试问题与解答

### 问题 1：`std::string_view` 有哪些潜在的危险？如何安全使用？

**答**：最大危险是**悬垂引用**。`string_view` 不拥有数据，若原始数据被销毁，视图变为悬垂，访问导致未定义行为。

**常见悬垂场景**：
- 返回局部 `std::string` 的 `string_view`
- 存储 `string_view` 在容器中，而原始 `string` 被销毁
- 从临时 `std::string` 构造 `string_view`

**安全使用原则**：
1. 确保 `string_view` 的生命周期不超过底层字符串的生命周期。
2. 将 `string_view` 用作函数参数（传入数据生命周期由调用方保证）。
3. 需要保存字符串副本时，显式转换为 `std::string`。
4. 避免从临时 `std::string` 构造 `string_view` 并保存。

```cpp
// 危险
std::string_view get_view() {
    std::string s = "hello";
    return s;   // 隐式转换为 string_view，s 销毁，悬垂
}

// 安全
std::string get_string() {
    std::string s = "hello";
    return s;   // 返回 string
}
```

### 问题 2：`std::string_view` 的 `data()` 函数不保证以 `'\0'` 结尾，如何安全地传递给需要 C 字符串的 API？

**答**：需要确保底层数据确实有空字符。以下方法可安全传递：

1. 使用 `std::string` 作为中转（拷贝）：
   ```cpp
   std::string_view sv = ...;
   std::string temp(sv);
   c_api(temp.c_str());
   ```

2. 若确定底层存储有空字符（如从字面量构造），可直接传递 `sv.data()`。
3. 编写安全包装函数，检查 `sv.size()` 与 `strlen(sv.data())` 是否一致，不一致则拷贝。

**推荐**：大多数现代 C API 接受长度参数，优先使用 `printf("%.*s", (int)sv.size(), sv.data())` 或 `fwrite` 等带长度的接口。

### 问题 3：`std::string_view` 作为函数参数时，与 `const std::string&` 相比有何性能差异？何时选择哪个？

**答**：

| 调用场景 | `const std::string&` | `std::string_view` |
|----------|----------------------|---------------------|
| 传入 `std::string` 变量 | 直接绑定（无拷贝） | 直接构造（无拷贝） |
| 传入字符串字面量 | 构造临时 `std::string`（分配内存、拷贝） | 直接构造（无分配） |
| 传入 `const char*` | 同上（分配） | 直接构造（需计算长度 O(n)） |
| 传入子串（如 `s.substr(5,3)`） | 创建临时 `std::string`（分配） | 创建临时 `string_view`（无分配） |

**结论**：`string_view` 在传入字面量、C 字符串或子串时避免内存分配，性能更优。但需注意从 `const char*` 构造时需 O(n) 计算长度（仅一次）。**默认优先使用 `string_view`**，除非：
- 函数内部需要调用需要 `std::string` 的 API（如 `c_str()` 且需要空结尾）。
- 函数需要存储参数的副本（此时用 `std::string` 参数更明确）。

### 问题 4：`std::string_view` 能否作为 `std::unordered_set` / `std::map` 的键？需要注意什么？

**答**：可以，但需极度小心生命周期。`unordered_set<string_view>` 存储的是视图，不拷贝字符串数据。若插入的字符串是临时对象或后续被销毁，集合中的视图悬垂。

**安全做法**：
- 使用 `std::string` 作为键，或
- 确保所有插入的视图指向静态存储或长期存在的字符串。
- C++20 起可定义 `std::unordered_set<std::string_view>` 配合自定义哈希，但仍需保证生命周期。

**示例（危险）**：
```cpp
std::unordered_set<std::string_view> set;
std::string s = "hello";
set.insert(s);           // 视图指向 s 的内部，s 销毁后悬垂
```

**安全示例**：
```cpp
std::unordered_set<std::string> safe_set;   // 推荐
```

---

## 典型错误与最佳实践

### ❌ 典型错误

#### 错误 1：返回局部 `std::string` 的 `string_view`

```cpp
std::string_view bad() {
    std::string s = "temporary";
    return s;           // 悬垂引用
}
```

#### 错误 2：将 `string_view` 存储到容器，原始字符串被销毁

```cpp
std::vector<std::string_view> views;
views.push_back(get_string().c_str());   // get_string 返回临时 string，悬垂
```

#### 错误 3：假设 `data()` 以 `'\0'` 结尾

```cpp
std::string s = "hello world";
std::string_view sv(s.data(), 5);        // 指向 "hello"
std::cout << sv.data();                  // 可能输出 "hello world"（巧合有 '\0'）或乱码
```

#### 错误 4：使用 `string_view` 修改底层数据（不能修改）

```cpp
std::string s = "hello";
std::string_view sv = s;
sv[0] = 'H';            // 错误：string_view 的 operator[] 返回 const char&
```

#### 错误 5：从临时 `std::string` 构造 `string_view` 并保存

```cpp
auto sv = std::string("temp");   // 临时 string 销毁，sv 悬垂
```

#### 错误 6：在 `constexpr` 上下文中使用非 `constexpr` 操作（C++20 前）

C++17 中很多 `string_view` 操作不是 `constexpr`，C++20 才完全支持。

---

### ✅ 最佳实践

| 实践 | 说明 |
|------|------|
| **只读字符串参数优先用 `string_view`** | 避免 `const string&` 带来的隐式分配 |
| **绝不返回 `string_view` 指向临时数据** | 返回 `std::string` 或确保数据生命周期足够长 |
| **将 `string_view` 视为“指针 + 长度”** | 谨慎管理生命周期，不拥有数据 |
| **需要修改或存储字符串时用 `std::string`** | 显式转换：`std::string(sv)` |
| **传递给 C API 时若需要 `\0` 结尾，先拷贝** | 使用 `std::string(sv).c_str()` 或带长度接口 |
| **使用 `sv = sv.substr(pos, len)` 来裁剪视图** | 不拷贝数据，O(1) |
| **优先使用 `sv.starts_with` / `ends_with`（C++20）** | 比手动 `compare` 更清晰 |
| **在编译期字符串处理中使用 `constexpr std::string_view`（C++20）** | 编译期哈希、正则等 |

```cpp
// 好的示例：函数参数
void log(std::string_view message) {
    std::cout << "[LOG] " << message << std::endl;
}
log("Application started");      // 无分配
std::string msg = "User logged in";
log(msg);                        // 无分配

// 好的示例：返回局部字符串（安全）
std::string make_string() {
    return "hello";              // 返回 string
}
std::string_view sv = make_string(); // 危险！仍会悬垂，因为 make_string 返回临时
// 正确：std::string s = make_string();
```

---

## 编译器支持与版本演进

（根据用户要求，本节省略详细版本表，仅简述）

- **C++17**：引入 `std::string_view`，基本成员函数支持。
- **C++20**：大多数成员函数变为 `constexpr`（包括构造、`substr`、`compare`、`find` 等），增加 `starts_with`/`ends_with`，提供 `std::hash` 特化。
- **C++23**：可能增加 `contains` 等成员（C++23 已加入 `std::string`，但 `string_view` 通常同步）。

**主流编译器支持**：GCC 7+、Clang 4+、MSVC 2017 15.6+ 基本支持 C++17 版本。C++20 增强需要 GCC 10+、Clang 10+、MSVC 2019 16.10+。

---

**总结**：`std::string_view` 是 C++17 中重要的性能优化工具，适用于所有只读字符串操作的场景。其非拥有语义既带来了零拷贝的性能优势，也引入了生命周期管理的责任。面试中常考其与 `const string&` 的区别、悬垂引用的风险以及正确使用场景。掌握 `string_view` 是编写现代高效 C++ 代码的必备技能。