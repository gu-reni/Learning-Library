# C++ 标准库总介绍

以下为 ISO C++ 标准定义的标准库主要组件（包含 STL 及扩展），提供容器、算法、迭代器、字符串、输入输出、并发、数值、通用工具等基础设施。C++ 标准库以头文件形式提供，所有组件均位于 `std` 命名空间中，支持类型安全、资源自动管理、泛型编程和高性能抽象。

---

## 容器

**<vector>**  
动态数组，支持随机访问，尾部插入/删除高效。  
典型场景：存储大小动态变化的序列，需要快速随机访问。  
现代 C++ 特性：C++11 支持初始化列表、移动语义；C++17 引入 `emplace_back` 返回引用。  
头文件依赖：通常独立。

**<deque>**  
双端队列，支持首尾高效插入/删除，随机访问略逊于 vector。  
典型场景：需要从两端操作数据的队列或双端队列。  
头文件依赖：通常独立。

**<list>**  
双向链表，任意位置插入/删除高效，不支持随机访问。  
典型场景：频繁在序列中间插入或删除元素。  
头文件依赖：通常独立。

**<forward_list>**  
单向链表，比 list 更轻量，仅支持向前遍历。  
典型场景：需要单向链表且对内存占用敏感。  
头文件依赖：通常独立。

**<array>**  
固定大小数组，封装原生数组，提供 STL 接口（如迭代器、`size()`）。  
典型场景：编译期确定大小的数组，需要与 STL 算法兼容。  
头文件依赖：通常独立。

**<set> / <multiset>**  
有序集合（红黑树），元素唯一（或允许重复），自动排序。  
典型场景：需要维护有序唯一元素的集合，快速查找、插入、删除。  
头文件依赖：通常独立。

**<map> / <multimap>**  
有序映射（红黑树），键值对存储，键唯一（或允许重复），自动按键排序。  
典型场景：需要按键快速查找值的有序字典。  
头文件依赖：通常独立。

**<unordered_set> / <unordered_multiset>**  
哈希无序集合，平均 O(1) 插入、查找、删除。  
典型场景：不需要有序性、追求高效查找的集合。  
头文件依赖：依赖 `<functional>` 提供哈希函数。

**<unordered_map> / <unordered_multimap>**  
哈希无序映射，键值对存储，平均 O(1) 访问。  
典型场景：高性能键值查找，无需排序。  
头文件依赖：依赖 `<functional>` 提供哈希函数。

**<queue>**  
队列适配器（默认基于 deque），提供 FIFO 语义。  
典型场景：先进先出的数据结构。  
头文件依赖：依赖 `<deque>`。

**<stack>**  
栈适配器（默认基于 deque），提供 LIFO 语义。  
典型场景：后进先出的数据结构。  
头文件依赖：依赖 `<deque>`。

**<priority_queue>**  
优先队列适配器（默认基于 vector），提供最大堆（或最小堆）语义。  
典型场景：需要按优先级取出元素。  
头文件依赖：依赖 `<vector>` 和 `<functional>`。

---

## 算法

**<algorithm>**  
提供大量泛型算法，如 `sort`、`find`、`for_each`、`transform`、`reverse`、`copy`、`unique`、`binary_search` 等。  
典型场景：对容器或迭代器区间进行排序、查找、变换、拷贝等操作。  
现代 C++ 特性：C++17 引入并行算法（与执行策略搭配），C++20 增加了 `ranges` 版本。  
头文件依赖：通常独立，但某些算法可能依赖 `<iterator>` 中的概念。

**<numeric>**  
提供数值算法，如 `accumulate`、`inner_product`、`partial_sum`、`adjacent_difference`、`iota`（C++11）等。  
典型场景：数值计算、数列生成、内积等。  
头文件依赖：通常独立。

---

## 迭代器

**<iterator>**  
定义迭代器类别（`input_iterator_tag`、`output_iterator_tag` 等）、迭代器辅助函数（`advance`、`distance`、`next`、`prev`）和迭代器适配器（`back_insert_iterator`、`front_insert_iterator`、`insert_iterator`、`istream_iterator`、`ostream_iterator` 等）。  
典型场景：自定义迭代器、使用插入器、流迭代器。  
头文件依赖：通常被其他头文件间接包含。

---

## 函数对象

**<functional>**  
提供函数对象包装器 `std::function`、绑定器 `std::bind`、引用包装器 `std::ref`/`std::cref`、预定义函数对象（`plus`、`less` 等）以及 C++11 起的 `std::function` 和 `std::mem_fn`。  
典型场景：将函数、函数对象、成员函数包装为可调用对象，用于回调、算法参数。  
现代 C++ 特性：C++11 起 `std::function` 可存储任何可调用对象；C++14 起 `std::bind` 可用通用 lambda 替代；C++20 引入 `std::bind_front`。  
头文件依赖：通常独立。

---

## 字符串

**<string>**  
提供 `std::string`（多字节字符）和 `std::wstring`（宽字符），以及 `std::u16string`、`std::u32string`（C++11）。  
典型场景：存储和操作字符序列，支持动态长度、赋值、连接、查找、子串等操作。  
现代 C++ 特性：C++11 支持移动语义、原始字符串字面量；C++17 添加 `std::string_view`（轻量级字符串视图）。  
头文件依赖：依赖 `<memory>` 分配器。

---

## 输入输出

**<iostream>**  
提供标准输入输出流对象：`std::cin`、`std::cout`、`std::cerr`、`std::clog`，以及 `std::ios_base`、`std::ios` 等基础类。  
典型场景：控制台输入输出。  
头文件依赖：通常独立，但内部依赖 `<streambuf>`、`<ostream>`、`<istream>` 等。

**<fstream>**  
提供文件流操作：`std::ifstream`、`std::ofstream`、`std::fstream`。  
典型场景：文件读写。  
头文件依赖：依赖 `<iostream>`。

**<sstream>**  
提供字符串流操作：`std::istringstream`、`std::ostringstream`、`std::stringstream`。  
典型场景：内存中的字符串格式化解析。  
头文件依赖：依赖 `<iostream>`。

**<iomanip>**  
提供格式化操纵符：`std::setw`、`std::setprecision`、`std::fixed`、`std::scientific` 等。  
典型场景：控制输出格式（宽度、精度、进制等）。  
头文件依赖：依赖 `<ios>`。

---

## 数值库

**<complex>**  
提供复数类型 `std::complex` 及算术运算。  
典型场景：科学计算、信号处理。  
头文件依赖：通常独立。

**<valarray>**  
提供数值数组类 `std::valarray` 及对数组的逐元素运算。  
典型场景：数值计算（性能敏感）。  
头文件依赖：通常独立。

**<random>**（C++11）  
提供随机数生成引擎（如 `std::mt19937`）和分布（如 `std::uniform_int_distribution`）。  
典型场景：产生高质量随机数。  
头文件依赖：通常独立。

**<ratio>**（C++11）  
提供编译期有理数类型 `std::ratio` 及其算术运算，用于 `<chrono>` 中的时间单位。  
典型场景：编译期时间单位定义。  
头文件依赖：通常独立。

---

## 并发支持

**<thread>**（C++11）  
提供 `std::thread` 类，管理执行线程。  
典型场景：多线程编程。  
头文件依赖：依赖 `<chrono>` 等。

**<mutex>**（C++11）  
提供互斥量（`std::mutex`、`std::recursive_mutex`）、锁（`std::lock_guard`、`std::unique_lock`）等。  
典型场景：线程同步。  
头文件依赖：通常独立。

**<condition_variable>**（C++11）  
提供条件变量 `std::condition_variable`，用于线程间等待通知。  
典型场景：生产者-消费者模型。  
头文件依赖：依赖 `<mutex>`。

**<future>**（C++11）  
提供异步任务结果处理：`std::future`、`std::promise`、`std::async`。  
典型场景：异步操作结果获取。  
头文件依赖：依赖 `<thread>`、`<mutex>`、`<condition_variable>` 等。

**<atomic>**（C++11）  
提供原子类型 `std::atomic<T>` 和原子操作，保证无锁编程。  
典型场景：多线程中的共享数据同步，避免互斥锁。  
头文件依赖：通常独立。

---

## 通用工具

**<utility>**  
提供 `std::pair`、`std::make_pair`、`std::move`、`std::forward`、`std::swap`、`std::declval`、`std::in_place` 等。  
典型场景：创建键值对、完美转发、移动语义、编译期类型推导辅助。  
头文件依赖：通常独立。

**<tuple>**（C++11）  
提供固定大小的异构对象集合 `std::tuple`，以及相关函数（`make_tuple`、`tie`、`get`）。  
典型场景：聚合多个不同类型的值，用于多返回值、编译期元编程。  
头文件依赖：依赖 `<utility>`。

**<memory>**  
提供智能指针（`std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr`）、分配器（`std::allocator`）、未初始化内存算法、`std::addressof`、`std::to_address`（C++20）等。  
典型场景：动态内存管理，避免资源泄漏。  
头文件依赖：依赖 `<cstddef>`、`<utility>` 等。

**<new>**  
定义 `operator new`/`operator delete` 的重载形式，以及 `std::nothrow_t`、`std::bad_alloc` 等。  
典型场景：自定义内存分配行为。  
头文件依赖：通常独立。

**<type_traits>**（C++11）  
提供编译期类型查询和转换模板，如 `std::is_same`、`std::enable_if`、`std::decay`、`std::is_base_of` 等。  
典型场景：模板元编程、SFINAE。  
头文件依赖：通常独立。

**<initializer_list>**（C++11）  
提供 `std::initializer_list`，支持用花括号初始化容器或自定义类型。  
典型场景：实现接受初始化列表的构造函数。  
头文件依赖：通常独立。

**<optional>**（C++17）  
提供 `std::optional<T>`，表示可能包含值的对象。  
典型场景：函数返回值可能无效，替代 `nullptr` 或 `-1` 等哨兵值。  
头文件依赖：依赖 `<new>`、`<utility>` 等。

**<variant>**（C++17）  
提供类型安全的联合体 `std::variant<T...>`，存储多种类型中的一种。  
典型场景：替代 C 风格的联合体，支持类型安全和访问。  
头文件依赖：依赖 `<new>`、`<utility>`、`<type_traits>` 等。

**<any>**（C++17）  
提供类型安全的容器 `std::any`，可存储任意可拷贝类型。  
典型场景：需要存储任意类型的值，但无法预知类型。  
头文件依赖：依赖 `<memory>`、`<type_traits>`、`<utility>` 等。

---

## 时间与日历

**<chrono>**（C++11）  
提供时间点（`std::chrono::time_point`）、时长（`std::chrono::duration`）和系统时钟（`std::chrono::system_clock` 等）。  
典型场景：高精度计时、超时控制、时间差计算。  
头文件依赖：依赖 `<ratio>`。

**<ctime>**  
C 风格时间函数（`time`、`localtime` 等），但通常推荐使用 `<chrono>`。  
头文件依赖：通常独立。

**<calendar>**（C++20）  
提供日历类型（`std::chrono::year_month_day`、`std::chrono::weekday` 等）。  
典型场景：处理公历日期、周几等。  
头文件依赖：依赖 `<chrono>`。

**<timezone>**（C++20）  
提供时区支持（`std::chrono::zoned_time`）。  
典型场景：跨时区时间转换。  
头文件依赖：依赖 `<chrono>`、`<calendar>`。

---

## 文件系统

**<filesystem>**（C++17）  
提供文件系统操作类（`std::filesystem::path`、`std::filesystem::directory_entry`）及函数（`exists`、`create_directory`、`copy`、`remove` 等）。  
典型场景：路径操作、文件遍历、元数据查询。  
头文件依赖：通常独立，但需注意平台差异。

---

## 正则表达式

**<regex>**（C++11）  
提供正则表达式引擎：`std::regex`、`std::smatch`、`std::regex_match`、`std::regex_search`、`std::regex_replace`。  
典型场景：文本匹配、提取、替换。  
头文件依赖：依赖 `<string>`、`<iterator>`。

---

## 其他

**<cstdlib>**  
C 标准库通用工具（`atoi`、`malloc`、`rand` 等），C++ 中推荐使用 `<cstdlib>` 并放在 `std` 命名空间，但通常建议用 C++ 替代（如 `<random>`、`new`/`delete`）。  
头文件依赖：通常独立。

**<climits>**、**<cfloat>**  
C 风格整数/浮点极限宏，C++ 中推荐使用 `<limits>` 中的 `std::numeric_limits`。  
头文件依赖：通常独立。

---

## 总括说明

- **定位**：C++ 标准库是现代 C++ 程序的基础，提供类型安全、零开销抽象和资源自动管理。它分为 STL（容器、算法、迭代器、函数对象、适配器、分配器）和扩展部分（字符串、I/O、并发、数值、时间、文件系统等）。STL 部分以泛型编程为核心，强调算法与容器的分离；扩展部分提供更丰富的应用层支持。
- **现代 C++ 演进**：
  - **C++11**：引入了移动语义、智能指针、线程库、`<random>`、`<chrono>`、正则表达式、哈希容器等，极大地丰富了标准库。
  - **C++14**：完善了泛型 lambda、`make_unique`、`std::integer_sequence` 等。
  - **C++17**：增加了 `std::optional`、`std::variant`、`std::any`、`std::filesystem`、并行算法、`std::string_view` 等。
  - **C++20**：引入了协程、`std::ranges`（范围视图）、`std::format`（格式化库）、`<calendar>`、`<timezone>`、`std::span`、三路比较运算符（`<=>`）等。
- **头文件组织**：C++ 标准库头文件通常不带 `.h` 后缀，如 `<vector>`、`<algorithm>`。与 C 库对应的头文件以 `c` 开头（如 `<cstdio>`），内容放入 `std` 命名空间，并可能提供重载或更安全的版本。
- **性能特点**：C++ 标准库在满足类型安全的前提下追求零开销抽象，大多数容器和算法与手写 C 代码性能相当，同时提供了更强的表达能力和安全性。
- **使用建议**：优先使用标准库而非自行实现底层数据结构或算法；利用 RAII 原则管理资源；在可能的情况下使用现代 C++ 特性提升代码可读性和安全性。

C++ 标准库是 C++ 语言的重要组成部分，熟练掌握其各个组件是编写高质量 C++ 程序的关键。