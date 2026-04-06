# std::optional、std::variant、std::any（C++17）

本文档涵盖 C++17 引入的三种“sum type”风格的工具：`std::optional`（可选值）、`std::variant`（类型安全的联合体）、`std::any`（类型擦除容器）。它们分别解决了“可能没有值”、“多种类型之一”、“存储任意类型”的问题，是现代 C++ 中表达不确定性和多态性的重要组件。

---

## 1. std::optional

**一句话定义**：`std::optional<T>` 是一个包含**要么有值，要么无值**（空状态）的对象容器，用于清晰表达“可能没有有效结果”的语义，避免使用特殊值（如 -1、nullptr）或输出参数。

### 详细语法与用法

```cpp
#include <optional>
#include <string>

// 基本构造
std::optional<int> o1;              // 空 optional
std::optional<int> o2 = std::nullopt; // 显式空
std::optional<int> o3 = 42;         // 包含值 42
std::optional<std::string> o4("hello"); // 包含字符串

// 使用 make_optional
auto o5 = std::make_optional<double>(3.14);

// 访问值
if (o3.has_value()) {               // 检查是否有值
    int val = o3.value();           // 有值返回，无值抛出 std::bad_optional_access
    int val2 = *o3;                 // 直接解引用（无值 UB）
    int val3 = o3.value_or(0);      // 无值时返回默认值
}

// 修改值
o3 = 100;                           // 赋值
o3.reset();                         // 清空
o3.emplace(200);                    // 原位构造

// 比较运算
std::optional<int> a = 10, b = 20, empty;
bool eq = (a == b);                 // false
bool lt = (a < b);                  // true
bool cmp_empty = (empty < a);       // true（空小于任何有值）
```

### 底层原理与实现机制

`std::optional<T>` 的典型实现包含两个部分：
- 一个对齐的存储缓冲区（`alignas(T) unsigned char storage[sizeof(T)]`）。
- 一个布尔标志（`bool has_value`）指示存储是否已初始化。

**关键设计点**：
- 不动态分配内存（`optional` 自身大小约为 `sizeof(T) + 1` 字节，加上对齐填充）。
- 当 `T` 是 trivially_copyable 时，`optional` 也是 trivially_copyable。
- 构造函数和析构函数根据标志位决定是否调用 `T` 的构造/析构。
- `optional` 支持**空基类优化**：如果 `T` 是空类，`optional` 大小可能为 1 字节（标志位）。

### 与相关特性的对比

| 替代方案 | 缺点 | `optional` 优势 |
|----------|------|-----------------|
| 特殊值（如 -1, nullptr） | 语义模糊，容易误用，不适用于所有类型 | 明确表达“无值”，类型安全 |
| 输出参数（`bool get(int& out)`） | 代码冗长，调用方需提前声明变量 | 返回值语义清晰，支持链式调用 |
| `std::pair<bool, T>` | 语义不直接，`first` 含义不明 | 自文档化，成员函数丰富 |
| 指针（`T*`） | 需动态分配或指向静态对象，生命周期复杂 | 值语义，自动管理生命周期 |

### 常见面试问题与解答

**问题 1：`std::optional` 与 `std::unique_ptr` 的区别？**

答：`optional` 存储对象自身（不分配堆内存），`unique_ptr` 存储指针（堆分配）。`optional` 适用于“可能有值，也可能没有”的**值语义**场景，`unique_ptr` 用于动态生命周期管理（多态、大对象等）。`optional` 的拷贝会拷贝对象（若 `T` 可拷贝），`unique_ptr` 不可拷贝，只能移动。

**问题 2：`optional<T>` 的 `operator*()` 和 `value()` 有何区别？**

答：`operator*()` 不检查有效性（UB），性能更高，适合已确认有值的场景；`value()` 会检查，无值时抛出异常，适合需要安全访问的场景。推荐使用 `value_or()` 或先 `if (opt)` 再解引用。

**问题 3：如何实现 `optional<T&>`（引用版本）？**

答：标准库不支持 `optional<T&>`（因为引用无法重置等语义）。C++26 将引入 `std::optional<T&>` 作为 `std::reference_wrapper<T>` 的替代。目前可用 `std::optional<std::reference_wrapper<T>>` 模拟。

### 典型错误与最佳实践

```cpp
// ❌ 错误：对空 optional 解引用
std::optional<int> opt;
int x = *opt;          // UB

// ❌ 错误：从临时对象构造 optional 并保存引用
std::optional<const std::string&> bad = "temp"; // 悬垂引用

// ✅ 最佳实践：使用 value_or 提供默认值
int y = opt.value_or(0);

// ✅ 最佳实践：先检查再使用
if (opt) {
    use(*opt);
}
```

---

## 2. std::variant

**一句话定义**：`std::variant<Types...>` 是一个类型安全的联合体，在任意时刻存储其模板参数列表中**某一个类型**的值，并自动管理生命周期，避免了 C 风格 `union` 的类型不安全问题。

### 详细语法与用法

```cpp
#include <variant>
#include <string>

// 定义 variant：可以存 int、double 或 std::string
std::variant<int, double, std::string> v;

v = 42;                     // 存 int
int i = std::get<int>(v);   // 按类型获取（若当前不是 int，抛异常）
int i2 = std::get<0>(v);    // 按索引获取

v = 3.14;
double d = std::get<double>(v);

v = "hello"s;
std::string s = std::get<std::string>(v);

// 检查当前类型
if (std::holds_alternative<int>(v)) {
    std::cout << "int\n";
}

// 访问器：std::visit（最推荐）
std::visit([](auto&& arg) {
    std::cout << arg << '\n';
}, v);

// 赋值时自动析构原对象，构造新对象
v = 100;    // 先销毁 string，再构造 int

// 异常安全：赋值可能抛出，variant 保证不变（不修改状态）
```

### 底层原理与实现机制

`std::variant` 的核心组件：
- **存储缓冲区**：对齐到最大类型大小，容量为 `max(sizeof(Types)...)`。
- **类型索引**：一个小的整数（通常为 `unsigned char` 或 `size_t`），指示当前激活的类型。
- **析构函数**：根据索引调用当前类型的析构函数。

**重要实现细节**：
- 不允许空状态（除特殊情况如 `std::monostate`），构造时默认初始化为第一个类型的值初始化（要求第一个类型可默认构造）。若第一个类型不能默认构造，可使用 `std::monostate` 作为占位符。
- 拷贝/移动构造和赋值需要根据索引执行相应的拷贝/移动。
- 赋值操作采用“临时变量 + swap”或“先析构再构造”策略以保证强异常安全（`std::variant` 的赋值提供基本异常安全，但部分实现提供强保证）。
- `std::visit` 的实现通常使用**编译期生成的跳转表**（函数指针数组）或递归模板，对每个类型生成一个调用，通过索引调用对应函数。

### 与相关特性的对比

| 特性 | `union`（C 风格） | `std::variant` |
|------|-------------------|----------------|
| 类型安全 | ❌ 需手动跟踪当前类型 | ✅ 自动跟踪，类型检查 |
| 非平凡类型（`std::string`） | ❌ 不允许或需手动构造/析构 | ✅ 自动管理 |
| 访问 | 直接成员访问（危险） | `std::get`、`std::visit`（安全） |
| 空状态 | 允许（无活动成员） | 不允许（需 `monostate`） |

### 常见面试问题与解答

**问题 1：`std::variant` 如何解决 `union` 无法存储非平凡类型的问题？**

答：`union` 不能自动调用非平凡类型的构造/析构函数，且无法知道当前活动成员。`std::variant` 通过存储类型索引和 placement new / 显式析构来自动管理生命周期，在赋值、拷贝、析构时根据索引执行正确的操作。

**问题 2：`std::variant` 能否存储引用类型？**

答：不能直接存储引用（如 `variant<int&>` 非法）。因为引用不可重新绑定，与 variant 语义冲突。可使用 `std::reference_wrapper<T>` 或指针。

**问题 3：`std::visit` 的常见用法及性能？**

答：`visit` 接受一个**泛型 lambda**（或多个重载的函数对象）和 variant，自动调用对应类型的重载。性能通常为 O(1)（跳转表）或 O(log N)（递归分派），但大多数实现使用跳转表实现常数时间。

```cpp
std::visit([](auto&& val) {
    using T = std::decay_t<decltype(val)>;
    if constexpr (std::is_same_v<T, int>) {
        // int 处理
    } else if constexpr (std::is_same_v<T, double>) {
        // double 处理
    }
}, v);
```

### 典型错误与最佳实践

```cpp
// ❌ 错误：使用未初始化的 variant（默认构造为第一个类型的值初始化）
std::variant<int, std::string> v;   // 存 int(0)，不是空

// ✅ 正确：需要空状态时使用 std::monostate
std::variant<std::monostate, int, std::string> v2; // 空状态

// ❌ 错误：std::get 类型不匹配
std::get<double>(v);   // 若当前不是 double，抛出异常

// ✅ 最佳实践：使用 visit 处理所有类型
std::visit([](auto&& arg) { /* 通用处理 */ }, v);

// ✅ 最佳实践：使用 holds_alternative 先检查
if (std::holds_alternative<int>(v)) {
    int i = std::get<int>(v);
}
```

---

## 3. std::any

**一句话定义**：`std::any` 是一个类型安全的容器，可以存储**任意可拷贝构造**的类型的单个值，但存储时丢失具体类型信息，需要通过 `std::any_cast` 恢复。

### 详细语法与用法

```cpp
#include <any>
#include <string>

std::any a;                 // 空 any
a = 42;                     // 存储 int
a = 3.14;                   // 存储 double（销毁之前的 int）
a = std::string("hello");   // 存储 string

// 访问值
if (a.has_value()) {
    // 按类型提取，若不匹配则抛出 std::bad_any_cast
    std::string s = std::any_cast<std::string>(a);
    
    // 指针版本（不抛异常）
    if (int* p = std::any_cast<int>(&a)) {
        // 当前不是 int，p 为 nullptr
    }
}

// 原位构造
a.emplace<std::string>("world");

// 清空
a.reset();

// 获取类型信息
const std::type_info& ti = a.type();   // 若空，为 typeid(void)
```

### 底层原理与实现机制

`std::any` 的典型实现使用**小对象优化（SBO, Small Buffer Optimization）**：
- 存储一个固定大小的缓冲区（通常为 32 字节）和指针对齐。
- 对于小类型（如 `int`、`double`、`char*`），直接存储在缓冲区中。
- 对于大类型，在堆上分配，缓冲区存储指针和大小。
- 内部维护一个虚函数表指针（或函数指针），指向类型相关的操作（拷贝、移动、析构、类型比较等）。

**关键点**：
- 拷贝 `any` 会拷贝存储的值（需要类型可拷贝）。
- `any` 不要求类型可移动？实际上移动会转移资源，但拷贝仍需要拷贝构造。
- `std::any_cast` 使用 RTTI 比较 `typeid` 来检查类型匹配。

### 与相关特性的对比

| 特性 | `std::any` | `void*` | `std::variant` | `std::optional` |
|------|------------|---------|----------------|-----------------|
| 存储类型 | 任意可拷贝类型 | 任意类型（不安全） | 预定义类型列表 | 单一类型或空 |
| 类型安全 | 运行时检查 | 无 | 编译期 + 运行时 | 编译期 |
| 内存分配 | 可能堆分配 | 无 | 无堆分配 | 无堆分配 |
| 适用场景 | 任意类型擦除（如属性系统） | C 兼容接口 | 有限类型集合 | 可有可无 |

### 常见面试问题与解答

**问题 1：`std::any` 与 `std::variant` 的区别及使用场景？**

答：`variant` 要求类型在编译期已知（有限集合），性能高，无堆分配（通常），适用于状态机、解析器等。`any` 可存储任意类型，类型在运行时确定，可能堆分配，适用于动态类型系统（如 JSON 解析、属性包）。优先使用 `variant` 若类型集合固定。

**问题 2：`std::any` 能否存储不可拷贝的类型？**

答：不能。`any` 要求存储类型**可拷贝构造**（因为 `any` 自身可拷贝）。若需要存储不可拷贝类型，可考虑 `std::unique_ptr<void, Deleter>` 或 C++26 的 `std::move_only_function` 等。

**问题 3：`std::any_cast` 指针版本与引用版本的区别？**

答：`any_cast<T>(&a)` 返回 `T*`，若类型不匹配返回 `nullptr`，不抛异常；`any_cast<T>(a)` 返回 `T` 的副本（或引用），不匹配时抛出 `std::bad_any_cast`。推荐使用指针版本进行安全检查。

### 典型错误与最佳实践

```cpp
// ❌ 错误：any_cast 类型不匹配导致异常
std::any a = 42;
double d = std::any_cast<double>(a);  // 抛出 bad_any_cast

// ✅ 正确：先检查类型
if (a.type() == typeid(int)) {
    int i = std::any_cast<int>(a);
}

// ✅ 最佳实践：使用指针版本避免异常
if (int* p = std::any_cast<int>(&a)) {
    use(*p);
}

// ❌ 错误：any 存储大对象频繁拷贝导致性能问题
std::any a = std::vector<int>(1000000);
auto b = a;   // 深拷贝整个 vector

// ✅ 最佳实践：需要时使用移动语义
std::any a = std::vector<int>(1000000);
std::any b = std::move(a);   // 移动，高效
```

---

## 三者综合对比

| 特性 | `std::optional<T>` | `std::variant<Types...>` | `std::any` |
|------|--------------------|--------------------------|------------|
| **包含值数量** | 0 或 1 个 | 恰好 1 个（类型是之一） | 0 或 1 个（任意类型） |
| **可能类型** | 单一类型 `T` | 编译期固定列表 | 运行时任意可拷贝类型 |
| **内存分配** | 无 | 无 | 可能（大对象） |
| **类型安全** | 编译期 | 编译期 + 运行时 | 运行时（通过 RTTI） |
| **访问方式** | `*`, `value()`, `value_or()` | `std::get`, `std::visit` | `std::any_cast` |
| **空状态** | 有（`nullopt`） | 无（需 `monostate`） | 有（`reset()`） |
| **典型用途** | 函数返回值“可能无结果” | 状态机、解析器、多类型返回值 | 动态属性系统、类型擦除 |

**选择指南**：
- 需要表达“可能有值，可能没有” → `optional`
- 需要在一组固定类型中选择一种 → `variant`
- 需要存储完全任意的类型（如脚本语言变量） → `any`

---

## 编译器支持与版本演进

（根据用户要求，本节省略详细版本表，仅简述）

- **C++17**：引入 `std::optional`、`std::variant`、`std::any`。
- **C++20/23**：对三者有小幅改进（如 `optional` 的 `and_then`、`or_else`、`transform` 等链式操作，C++23 加入；`variant` 的 `std::visit` 支持变参等）。

**主流编译器支持**：GCC 7+、Clang 4+、MSVC 2017 15.6+ 基本支持 C++17 版本。

---

**总结**：`optional`、`variant`、`any` 分别填补了 C++ 类型系统中“可空值”、“带标签联合”、“类型擦除”的空白，它们都是现代 C++ 中表达复杂数据流动的安全工具。面试中常考各自的使用场景、性能特性以及与旧有模式的对比。掌握三者是写出健壮、自文档化代码的关键。