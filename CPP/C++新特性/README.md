# C++ 新特性总览

本目录按 C++ 标准版本（C++11、C++14、C++17、C++20、C++23、C++26）整理了各版本引入的核心语言与标准库新特性。每个特性文档包含语法示例、典型场景、与旧版本的对比及注意事项，适合快速回顾或面试准备。

---

## 📌 版本概览

| 版本 | 发布时间 | 核心亮点 | 学习优先级 |
|------|----------|----------|-------------|
| **C++11** | 2011 | 移动语义、lambda、智能指针、并发库、`auto`、范围 for、`constexpr`、变参模板 | **必须掌握** |
| **C++14** | 2014 | 泛型 lambda、变量模板、二进制字面量、`constexpr` 增强 | **掌握**（在 C++11 基础上完善） |
| **C++17** | 2017 | 结构化绑定、`if`/`switch` 初始化、折叠表达式、文件系统库、`std::optional`/`variant`/`any`、`std::string_view`、并行算法 | **重点掌握**（企业主流） |
| **C++20** | 2020 | 概念（concepts）、协程、范围（ranges）、三路比较（`<=>`）、模块、`std::format`、`std::span`、日历与时区 | **熟悉**（前沿项目逐渐采用） |
| **C++23** | 2023 | `std::print`、`std::expected`、`std::mdspan`、`std::flat_map/set`、`deducing this`、`#embed` | **了解**（新特性预览） |
| **C++26** | 2026（预计） | 编译时反射、契约、`std::simd`、模式匹配、`std::execution`、安全配置 | **了解**（关注未来趋势） |

---

## 📖 学习建议

1. **入门路径**：  
   - 先吃透 **C++11**，它是现代 C++ 的基石。  
   - 再学习 **C++14**（少量补充）和 **C++17**（大量实用特性）。  
   - 最后根据工作需要或兴趣了解 **C++20** 及更高版本。

2. **面试重点**：  
   - 智能指针、移动语义、lambda、`auto`、范围 for、`constexpr`（C++11）  
   - 结构化绑定、`std::optional`、`std::string_view`、文件系统库（C++17）  
   - 概念、协程、范围（C++20，加分项）

3. **实战建议**：  
   - 在代码中主动使用现代特性（如用 `std::unique_ptr` 替代裸指针）。  
   - 阅读知名开源项目（如 LLVM、Boost、Folly）了解最佳实践。  
   - 使用最新编译器（GCC 11+、Clang 14+）体验 C++20/23 特性。

---

## 📂 目录结构

```
C++新特性/
├── README.md
├── C++11/
│   ├── 语言特性.md
│   ├── 库特性.md
│   ├── 移动语义与右值引用.md
│   ├── lambda表达式.md
│   ├── 智能指针.md
│   ├── 并发库（thread, atomic, future等）.md
│   ├── constexpr.md
│   ├── nullptr与强枚举类型.md
│   ├── 初始化列表与变参模板.md
│   └── 其他（auto, 范围for, override, final等）.md
├── C++14/
│   ├── 语言特性.md
│   ├── 库特性.md
│   ├── 泛型lambda.md
│   ├── 变量模板.md
│   ├── 二进制字面量与数字分隔符.md
│   └── constexpr增强.md
├── C++17/
│   ├── 语言特性.md
│   ├── 库特性.md
│   ├── 结构化绑定.md
│   ├── if与switch初始化.md
│   ├── 折叠表达式.md
│   ├── 内联变量.md
│   ├── std::optional, std::variant, std::any.md
│   ├── std::string_view.md
│   ├── 文件系统库.md
│   ├── 并行算法与执行策略.md
│   └── 其他（constexpr lambda, 动态内存分配等）.md
├── C++20/
│   ├── 语言特性.md
│   ├── 库特性.md
│   ├── 概念（concepts）.md
│   ├── 协程（coroutines）.md
│   ├── 范围（ranges）.md
│   ├── 三路比较（<=>）.md
│   ├── 指定初始化器.md
│   ├── constexpr增强（虚函数、dynamic_cast等）.md
│   ├── 模块（modules）.md
│   ├── std::span.md
│   ├── std::format.md
│   ├── std::chrono日历与时区.md
│   ├── 同步输出流.md
│   └── 其他（特性测试宏、likely/unlikely等）.md
├── C++23/
│   ├── 语言特性.md
│   ├── 库特性.md
│   ├── 显式对象参数（deducing this）.md
│   ├── if consteval.md
│   ├── 多维数组支持（std::mdspan）.md
│   ├── std::expected.md
│   ├── std::print与std::println.md
│   ├── std::flat_map / std::flat_set.md
│   ├── 标准库模块（std模块）.md
│   ├── std::generator（范围生成器）.md
│   ├── std::stacktrace.md
│   └── 其他（#embed指令、std::byteswap等）.md
└── C++26/
    ├── 语言特性.md
    ├── 库特性.md
    ├── 反射（compile-time reflection）.md
    ├── 契约（contracts）.md
    ├── 模式匹配（pattern matching）.md
    ├── std::simd（数据并行）.md
    ├── std::execution（异步执行模型）.md
    ├── 安全配置（Safe C++）.md
    ├── 线性代数（std::linalg）.md
    ├── 文本编码（std::text_encoding）.md
    ├── inplace_vector与hive容器.md
    ├── 安全回收（hazard pointers, RCU）.md
    └── 其他（调试支持、契约支持等）.md
```

---

## 🔗 相关资源

- [cppreference.com](https://en.cppreference.com/)（权威参考）  
- [C++ Standards Committee](https://isocpp.org/)  
- 《Effective Modern C++》—— Scott Meyers  
- 《C++17 - The Complete Guide》—— Nicolai M. Josuttis

---

*最后更新：2026年4月*