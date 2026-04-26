# C++ 标准库总介绍

以下为 C++ 标准库的完整分类与介绍，按照 **C 标准库**、**STL 核心组件**、**扩展库** 三大模块组织。每个模块下细分子类，并列出对应的头文件及其功能说明。本介绍涵盖 C++98 至 C++20 的所有标准库头文件（部分 C++23 新特性未单独列出），遵循 ISO C++ 标准。

---

## 1. C 标准库

C 标准库头文件以 `<c...>` 形式提供，内容置于 `std` 命名空间，保留与 C 语言兼容的接口。它们是 C++ 标准库的基础，涵盖输入输出、字符串、数学、时间、内存管理等基本功能。

### 输入输出
**<cstdio>**  
功能描述：提供 C 风格输入输出函数，如 `printf`、`scanf`、`fopen`、`fclose`、`fread`、`fwrite` 等。  
典型场景：与 C 代码交互或需要直接控制文件 I/O。  
现代 C++ 对应：`<iostream>`、`<fstream>`、`<iomanip>`。  
头文件依赖：通常独立。

**<cwchar>**  
功能描述：提供宽字符版本的 C 风格 I/O 函数，如 `wprintf`、`wscanf`、`fgetwc`、`fputwc` 等。  
典型场景：处理宽字符流或与宽字符 C 库交互。  
现代 C++ 对应：`<iostream>` 中的 `wcout`、`wcin` 等。  
头文件依赖：通常独立。

### 字符串与字符处理
**<cstring>**  
功能描述：C 风格字符串处理函数，如 `strcpy`、`strlen`、`memcpy`、`memset` 等。  
典型场景：操作 C 风格字符数组或内存块。  
现代 C++ 对应：`<string>` 中的 `std::string` 及相关成员函数。  
头文件依赖：通常独立。

**<cctype>**  
功能描述：字符分类与转换函数，如 `isalpha`、`isdigit`、`toupper`、`tolower` 等。  
典型场景：判断字符类型或进行大小写转换。  
现代 C++ 对应：`<cctype>` 仍可使用，也可使用 C++ 的 `std::isalpha` 等（定义于 `<cctype>`）。  
头文件依赖：通常独立。

**<cwctype>**  
功能描述：宽字符分类与转换函数，如 `iswalpha`、`iswdigit`、`towupper`、`towlower` 等。  
典型场景：处理宽字符的字符分类。  
现代 C++ 对应：`<cwctype>` 仍可使用。  
头文件依赖：通常独立。

**<cuchar>**（C++11）  
功能描述：提供 Unicode 字符转换函数（`c16rtomb`、`c32rtomb`、`mbrtoc16`、`mbrtoc32`）。  
典型场景：在多字节字符与 UTF-16/UTF-32 之间转换。  
现代 C++ 对应：`<string>` 中的 `std::u16string`、`std::u32string`，以及 `<codecvt>`（C++17 前）或 `<cuchar>`。  
头文件依赖：通常独立。

### 数学函数
**<cmath>**  
功能描述：标准数学函数，如 `sin`、`cos`、`sqrt`、`pow`、`exp`、`log` 等，以及浮点分类宏（`FP_NAN` 等）。  
典型场景：科学计算、几何运算、统计分析。  
现代 C++ 对应：`<cmath>` 仍为 C++ 标准数学库。  
头文件依赖：通常独立，但某些系统需链接数学库 `-lm`。

**<cstdlib>**（数学部分）  
功能描述：提供整数绝对值函数 `abs`、`div` 等。  
典型场景：整数运算辅助。  
现代 C++ 对应：`<cstdlib>` 仍可使用，C++11 起 `<cmath>` 也提供整数重载。  
头文件依赖：通常独立。

### 时间与日期
**<ctime>**  
功能描述：C 风格时间处理函数，如 `time`、`localtime`、`strftime`、`difftime` 等，以及 `time_t`、`struct tm` 类型。  
典型场景：获取当前时间、格式化日期字符串。  
现代 C++ 对应：`<chrono>` 提供类型安全的高精度时间库。  
头文件依赖：通常独立。

### 内存管理与工具
**<cstdlib>**（通用部分）  
功能描述：提供内存分配与释放（`malloc`、`calloc`、`realloc`、`free`）、程序终止（`exit`、`abort`）、环境控制（`system`、`getenv`）、随机数（`rand`、`srand`）、字符串转换（`atoi`、`atol`、`strtol`、`strtod`）等。  
典型场景：动态内存管理、进程控制、整数/浮点转换。  
现代 C++ 对应：内存管理用 `new`/`delete` 或智能指针（`<memory>`），随机数用 `<random>`，字符串转换用 `<string>` 中的 `std::stoi` 等。  
头文件依赖：通常独立。

### 错误处理与断言
**<cassert>**  
功能描述：提供 `assert` 宏，在调试时检查条件，条件为假时终止程序并输出错误信息。  
典型场景：开发阶段验证不变量、前置条件。  
现代 C++ 对应：C++ 中仍可使用 `assert`，也可使用 `static_assert` 进行编译期断言。  
头文件依赖：通常独立。

**<cerrno>**  
功能描述：定义全局变量 `errno` 及错误码宏（如 `EDOM`、`ERANGE`、`EINVAL` 等）。  
典型场景：系统调用或库函数失败时获取详细错误原因。  
现代 C++ 对应：C++ 中仍可使用 `errno`，但更推荐通过异常（`std::system_error`）处理错误。  
头文件依赖：通常独立。

### 类型与极限
**<climits>**  
功能描述：定义整数类型取值范围，如 `INT_MAX`、`INT_MIN`、`CHAR_BIT` 等。  
典型场景：确保数值不溢出，跨平台可移植性。  
现代 C++ 对应：`<limits>` 中的 `std::numeric_limits` 模板类提供类型安全的方式。  
头文件依赖：通常独立。

**<cfloat>**  
功能描述：定义浮点类型取值范围和精度，如 `FLT_MAX`、`DBL_EPSILON` 等。  
典型场景：浮点运算的精度控制、比较时的容差设置。  
现代 C++ 对应：`<limits>` 中的 `std::numeric_limits<float>::epsilon()` 等。  
头文件依赖：通常独立。

**<cstdint>**（C++11）  
功能描述：定义定长整数类型，如 `int8_t`、`uint32_t`、`int64_t` 等。  
典型场景：需要固定大小整数的跨平台代码。  
现代 C++ 对应：`<cstdint>` 本身即为现代 C++ 推荐使用。  
头文件依赖：通常独立。

**<cstddef>**  
功能描述：定义 `size_t`、`ptrdiff_t`、`nullptr_t` 和 `offsetof` 宏。  
典型场景：表示大小、指针差、空指针、结构体成员偏移量。  
现代 C++ 对应：`<cstddef>` 仍为基本类型定义头文件。  
头文件依赖：通常独立。

### 宽字符与多字节
**<cwchar>**  
功能描述：已在“输入输出”中列出，同时包含宽字符字符串处理函数（`wcscpy`、`wcslen` 等）。  
典型场景：宽字符字符串操作。  
现代 C++ 对应：`<string>` 中的 `std::wstring`。  
头文件依赖：通常独立。

**<cwctype>**  
功能描述：已在“字符串与字符处理”中列出。  
头文件依赖：通常独立。

### 非局部跳转
**<csetjmp>**  
功能描述：提供 `setjmp` 和 `longjmp` 函数，用于非局部跳转，绕过常规函数调用栈。  
典型场景：错误恢复、在 C 语言中模拟异常（C++ 中应避免使用）。  
现代 C++ 对应：C++ 应使用异常机制（`try`/`catch`）。  
头文件依赖：通常独立。

### 信号处理
**<csignal>**  
功能描述：提供信号处理函数 `signal` 和 `raise`，以及信号宏（`SIGINT`、`SIGTERM`、`SIGSEGV` 等）。  
典型场景：捕捉程序中断、段错误等信号，执行清理操作。  
现代 C++ 对应：`<csignal>` 保留，但现代 C++ 程序更推荐使用信号安全的异步操作。  
头文件依赖：通常独立。

### 可变参数
**<cstdarg>**  
功能描述：提供可变参数处理宏 `va_start`、`va_arg`、`va_end`、`va_copy`（C99）。  
典型场景：实现接受可变参数数量的函数。  
现代 C++ 对应：C++11 起使用可变参数模板（variadic templates）。  
头文件依赖：通常独立。

---

## 2. STL 核心组件

STL（Standard Template Library）是 C++ 标准库中基于泛型编程的核心部分，由容器、迭代器、算法、函数对象、适配器和分配器组成。

### 容器

#### 序列容器
**<vector>**  
功能描述：动态数组，支持随机访问，尾部插入/删除高效。  
典型场景：存储大小动态变化的序列，需要快速随机访问。  
现代 C++ 对应：C++11 起支持移动语义、初始化列表。  
头文件依赖：通常独立。

**<deque>**  
功能描述：双端队列，支持首尾高效插入/删除，随机访问略逊于 vector。  
典型场景：需要从两端操作数据的队列或双端队列。  
头文件依赖：通常独立。

**<list>**  
功能描述：双向链表，任意位置插入/删除高效，不支持随机访问。  
典型场景：频繁在序列中间插入或删除元素。  
头文件依赖：通常独立。

**<forward_list>**（C++11）  
功能描述：单向链表，比 list 更轻量，仅支持向前遍历。  
典型场景：需要单向链表且对内存占用敏感。  
头文件依赖：通常独立。

**<array>**（C++11）  
功能描述：固定大小数组，封装原生数组，提供 STL 接口（如迭代器、`size()`）。  
典型场景：编译期确定大小的数组，需要与 STL 算法兼容。  
头文件依赖：通常独立。

**<span>**（C++20）  
功能描述：连续序列视图，不拥有数据，提供轻量级访问。  
典型场景：函数参数传递，避免拷贝。  
头文件依赖：通常独立。

#### 关联容器（有序）
**<set>**  
功能描述：有序集合（红黑树），元素唯一，自动排序。  
典型场景：需要维护有序唯一元素的集合，快速查找、插入、删除。  
头文件依赖：通常独立。

**<multiset>**  
功能描述：有序多重集合，允许重复元素。  
头文件依赖：通常独立。

**<map>**  
功能描述：有序映射，键值对存储，键唯一，自动按键排序。  
典型场景：需要按键快速查找值的有序字典。  
头文件依赖：通常独立。

**<multimap>**  
功能描述：有序多重映射，允许重复键。  
头文件依赖：通常独立。

#### 无序关联容器（哈希）
**<unordered_set>**（C++11）  
功能描述：哈希无序集合，平均 O(1) 插入、查找、删除。  
典型场景：不需要有序性、追求高效查找的集合。  
头文件依赖：依赖 `<functional>` 提供哈希函数。

**<unordered_multiset>**（C++11）  
功能描述：哈希无序多重集合。  
头文件依赖：同 `<unordered_set>`。

**<unordered_map>**（C++11）  
功能描述：哈希无序映射，平均 O(1) 访问。  
典型场景：高性能键值查找，无需排序。  
头文件依赖：依赖 `<functional>` 提供哈希函数。

**<unordered_multimap>**（C++11）  
功能描述：哈希无序多重映射。  
头文件依赖：同 `<unordered_map>`。

#### 容器适配器
**<stack>**  
功能描述：栈适配器（LIFO），默认基于 `deque`。  
典型场景：后进先出的数据结构。  
头文件依赖：依赖 `<deque>`。

**<queue>**  
功能描述：队列适配器（FIFO），默认基于 `deque`。  
典型场景：先进先出的数据结构。  
头文件依赖：依赖 `<deque>`。

**<priority_queue>**  
功能描述：优先队列适配器，默认基于 `vector`，提供最大堆（或最小堆）语义。  
典型场景：需要按优先级取出元素。  
头文件依赖：依赖 `<vector>` 和 `<functional>`。

### 迭代器
**<iterator>**  
功能描述：定义迭代器类别、辅助函数（`advance`、`distance`、`next`、`prev`）、迭代器适配器（`back_insert_iterator`、`front_insert_iterator`、`insert_iterator`、`istream_iterator`、`ostream_iterator`、`reverse_iterator` 等）。  
典型场景：自定义迭代器、使用插入器、流迭代器。  
现代 C++ 对应：C++11 起 `cbegin`/`cend` 等。  
头文件依赖：通常独立。

### 算法
**<algorithm>**  
功能描述：提供大量泛型算法，如 `sort`、`find`、`for_each`、`transform`、`reverse`、`copy`、`unique`、`binary_search` 等。  
典型场景：对容器或迭代器区间进行排序、查找、变换、拷贝等操作。  
现代 C++ 对应：C++17 引入并行算法（执行策略），C++20 引入 `ranges` 版本。  
头文件依赖：通常独立。

**<numeric>**  
功能描述：数值算法，如 `accumulate`、`inner_product`、`partial_sum`、`adjacent_difference`、`iota`（C++11）。  
典型场景：数值计算、数列生成、内积等。  
头文件依赖：通常独立。

### 函数对象
**<functional>**  
功能描述：提供函数对象包装器 `std::function`、绑定器 `std::bind`、引用包装器 `std::ref`/`std::cref`、预定义函数对象（`plus`、`less` 等）以及 C++11 起的 `std::function`、`std::mem_fn` 等。  
典型场景：将函数、函数对象、成员函数包装为可调用对象，用于回调、算法参数。  
现代 C++ 对应：C++11 起 lambda 表达式成为更便捷的替代。  
头文件依赖：通常独立。

### 适配器
适配器分为容器适配器（已在容器部分列出）、迭代器适配器（包含在 `<iterator>`）、函数适配器（包含在 `<functional>`，如 `std::bind`）。此处不再重复列出具体头文件。

### 分配器
**<memory>**（分配器部分）  
功能描述：提供 `std::allocator`、`std::allocator_traits`、`std::scoped_allocator_adaptor`（C++11）等，用于自定义内存分配策略。  
典型场景：内存池、共享内存、对象池。  
现代 C++ 对应：C++17 简化分配器接口，C++20 引入 `polymorphic_allocator`。  
头文件依赖：通常独立。

---

## 3. 扩展库

扩展库包括 STL 之外的标准库组件，提供字符串、输入输出、并发、数值、时间、文件系统等更高级的功能。

### 字符串与文本
**<string>**  
功能描述：`std::string`、`std::wstring`、`std::u16string`、`std::u32string` 等字符串类。  
典型场景：存储和操作字符序列，支持动态长度、赋值、连接、查找、子串等操作。  
现代 C++ 对应：C++11 支持移动语义、原始字符串字面量；C++17 添加 `std::string_view`。  
头文件依赖：依赖 `<memory>` 分配器。

**<string_view>**（C++17）  
功能描述：轻量级字符串视图，不拥有数据，用于高效传递字符串片段。  
典型场景：函数参数传递，避免拷贝。  
头文件依赖：通常独立。

**<regex>**（C++11）  
功能描述：正则表达式引擎，提供 `std::regex`、`std::smatch`、`std::regex_match`、`std::regex_search`、`std::regex_replace`。  
典型场景：文本匹配、提取、替换。  
头文件依赖：依赖 `<string>`、`<iterator>`。

### 输入输出
**<iostream>**  
功能描述：标准输入输出流对象：`std::cin`、`std::cout`、`std::cerr`、`std::clog`，以及 `std::ios_base`、`std::ios` 等基础类。  
典型场景：控制台输入输出。  
头文件依赖：通常独立，但内部依赖 `<streambuf>`、`<ostream>`、`<istream>` 等。

**<fstream>**  
功能描述：文件流操作：`std::ifstream`、`std::ofstream`、`std::fstream`。  
典型场景：文件读写。  
头文件依赖：依赖 `<iostream>`。

**<sstream>**  
功能描述：字符串流操作：`std::istringstream`、`std::ostringstream`、`std::stringstream`。  
典型场景：内存中的字符串格式化解析。  
头文件依赖：依赖 `<iostream>`。

**<iomanip>**  
功能描述：格式化操纵符：`std::setw`、`std::setprecision`、`std::fixed`、`std::scientific` 等。  
典型场景：控制输出格式（宽度、精度、进制等）。  
头文件依赖：依赖 `<ios>`。

**<syncstream>**（C++20）  
功能描述：同步输出流，用于多线程环境下安全输出。  
典型场景：多线程程序中避免输出交错。  
头文件依赖：依赖 `<ostream>`。

### 并发与并行
**<thread>**（C++11）  
功能描述：`std::thread` 类，管理执行线程。  
典型场景：多线程编程。  
头文件依赖：依赖 `<chrono>` 等。

**<future>**（C++11）  
功能描述：异步任务结果处理：`std::future`、`std::promise`、`std::async`。  
典型场景：异步操作结果获取。  
头文件依赖：依赖 `<thread>`、`<mutex>`、`<condition_variable>` 等。

**<coroutine>**（C++20）  
功能描述：协程支持，提供 `std::coroutine_handle`、`std::suspend_never`、`std::suspend_always` 等。  
典型场景：编写可暂停、恢复的函数。  
头文件依赖：通常独立。

**<mutex>**（C++11）  
功能描述：互斥量（`std::mutex`、`std::recursive_mutex`）、锁（`std::lock_guard`、`std::unique_lock`）等。  
典型场景：线程同步。  
头文件依赖：通常独立。

**<condition_variable>**（C++11）  
功能描述：条件变量 `std::condition_variable`，用于线程间等待通知。  
典型场景：生产者-消费者模型。  
头文件依赖：依赖 `<mutex>`。

**<shared_mutex>**（C++14）  
功能描述：共享互斥量 `std::shared_mutex`，支持共享锁定。  
典型场景：读写锁场景。  
头文件依赖：通常独立。

**<semaphore>**（C++20）  
功能描述：信号量 `std::counting_semaphore`、`std::binary_semaphore`。  
典型场景：计数同步。  
头文件依赖：通常独立。

**<atomic>**（C++11）  
功能描述：原子类型 `std::atomic<T>` 和原子操作，保证无锁编程。  
典型场景：多线程中的共享数据同步，避免互斥锁。  
头文件依赖：通常独立。

### 数值与随机
**<complex>**  
功能描述：复数类型 `std::complex` 及算术运算。  
典型场景：科学计算、信号处理。  
头文件依赖：通常独立。

**<valarray>**  
功能描述：数值数组类 `std::valarray` 及对数组的逐元素运算。  
典型场景：数值计算（性能敏感）。  
头文件依赖：通常独立。

**<random>**（C++11）  
功能描述：随机数生成引擎（如 `std::mt19937`）和分布（如 `std::uniform_int_distribution`）。  
典型场景：产生高质量随机数。  
头文件依赖：通常独立。

**<ratio>**（C++11）  
功能描述：编译期有理数类型 `std::ratio` 及其算术运算，用于 `<chrono>` 中的时间单位。  
典型场景：编译期时间单位定义。  
头文件依赖：通常独立。

**<cfenv>**（C++11）  
功能描述：浮点环境控制函数和宏（`feclearexcept`、`fetestexcept`、`FE_DIVBYZERO` 等）。  
典型场景：浮点异常检测、舍入模式控制。  
头文件依赖：通常独立。

### 时间与日历
**<chrono>**（C++11）  
功能描述：时间点（`std::chrono::time_point`）、时长（`std::chrono::duration`）和系统时钟（`std::chrono::system_clock` 等）。  
典型场景：高精度计时、超时控制、时间差计算。  
头文件依赖：依赖 `<ratio>`。

**<calendar>**（C++20）  
功能描述：日历类型（`std::chrono::year_month_day`、`std::chrono::weekday` 等）。  
典型场景：处理公历日期、周几等。  
头文件依赖：依赖 `<chrono>`。

**<timezone>**（C++20）  
功能描述：时区支持（`std::chrono::zoned_time`）。  
典型场景：跨时区时间转换。  
头文件依赖：依赖 `<chrono>`、`<calendar>`。

**<ctime>**  
功能描述：已在 C 标准库中介绍，此处提供 C 风格时间函数。

### 文件系统
**<filesystem>**（C++17）  
功能描述：文件系统操作类（`std::filesystem::path`、`std::filesystem::directory_entry`）及函数（`exists`、`create_directory`、`copy`、`remove` 等）。  
典型场景：路径操作、文件遍历、元数据查询。  
头文件依赖：通常独立。

### 泛型工具与类型支持
**<utility>**  
功能描述：提供 `std::pair`、`std::make_pair`、`std::move`、`std::forward`、`std::swap`、`std::declval`、`std::in_place` 等。  
典型场景：创建键值对、完美转发、移动语义、编译期类型推导辅助。  
头文件依赖：通常独立。

**<tuple>**（C++11）  
功能描述：固定大小的异构对象集合 `std::tuple`，以及相关函数（`make_tuple`、`tie`、`get`）。  
典型场景：聚合多个不同类型的值，用于多返回值、编译期元编程。  
头文件依赖：依赖 `<utility>`。

**<optional>**（C++17）  
功能描述：`std::optional<T>`，表示可能包含值的对象。  
典型场景：函数返回值可能无效，替代 `nullptr` 或 `-1` 等哨兵值。  
头文件依赖：依赖 `<new>`、`<utility>` 等。

**<variant>**（C++17）  
功能描述：类型安全的联合体 `std::variant<T...>`，存储多种类型中的一种。  
典型场景：替代 C 风格的联合体，支持类型安全和访问。  
头文件依赖：依赖 `<new>`、`<utility>`、`<type_traits>` 等。

**<any>**（C++17）  
功能描述：类型安全的容器 `std::any`，可存储任意可拷贝类型。  
典型场景：需要存储任意类型的值，但无法预知类型。  
头文件依赖：依赖 `<memory>`、`<type_traits>`、`<utility>` 等。

**<type_traits>**（C++11）  
功能描述：编译期类型查询和转换模板，如 `std::is_same`、`std::enable_if`、`std::decay`、`std::is_base_of` 等。  
典型场景：模板元编程、SFINAE。  
头文件依赖：通常独立。

**<initializer_list>**（C++11）  
功能描述：`std::initializer_list`，支持用花括号初始化容器或自定义类型。  
典型场景：实现接受初始化列表的构造函数。  
头文件依赖：通常独立。

**<bit>**（C++20）  
功能描述：位操作函数，如 `std::bit_cast`、`std::byteswap`、`std::popcount` 等。  
典型场景：位运算、字节序转换。  
头文件依赖：通常独立。

**<version>**（C++20）  
功能描述：提供标准库特性宏，用于检测实现支持的版本和功能。  
典型场景：条件编译。  
头文件依赖：通常独立。

### 本地化
**<locale>**  
功能描述：本地化支持，包括 `std::locale`、`std::facet`、`std::use_facet` 等，用于处理区域相关的格式（日期、货币、字符分类）。  
典型场景：国际化应用，处理不同地区的输出格式。  
头文件依赖：通常独立。

**<clocale>**  
功能描述：C 风格本地化函数，如 `setlocale`、`localeconv`。  
典型场景：与 C 代码交互或简单本地化需求。  
现代 C++ 对应：`<locale>` 提供 C++ 风格的本地化设施。  
头文件依赖：通常独立。

### 内存管理
**<new>**  
功能描述：定义 `operator new`/`operator delete` 的重载形式，以及 `std::nothrow_t`、`std::bad_alloc` 等。  
典型场景：自定义内存分配行为、处理内存分配失败。  
头文件依赖：通常独立。

**<memory>**（智能指针部分）  
功能描述：提供智能指针（`std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr`）、未初始化内存算法、`std::addressof`、`std::to_address`（C++20）等。  
典型场景：动态内存管理，避免资源泄漏。  
头文件依赖：依赖 `<cstddef>`、`<utility>` 等。

### 范围库（C++20）
**<ranges>**  
功能描述：范围适配器与视图，提供函数式风格的序列操作（`views::filter`、`views::transform`、`views::take` 等）。  
典型场景：简化对容器的组合操作，替代传统迭代器。  
头文件依赖：依赖 `<iterator>`、`<algorithm>`。

### 格式化库（C++20）
**<format>**  
功能描述：字符串格式化库，提供 `std::format`、`std::formatter`，类似 Python 的 `str.format`。  
典型场景：类型安全、高效地格式化字符串。  
头文件依赖：依赖 `<string>`、`<tuple>` 等。

### 其他语言支持
**<typeinfo>**  
功能描述：运行时类型信息，`std::type_info`、`typeid` 运算符。  
典型场景：多态类型识别。  
头文件依赖：通常独立。

**<exception>**  
功能描述：异常处理基类 `std::exception`，以及 `std::terminate`、`std::uncaught_exception` 等。  
典型场景：自定义异常类。  
头文件依赖：通常独立。

**<stdexcept>**  
功能描述：标准异常类，如 `std::logic_error`、`std::runtime_error` 及其派生类。  
典型场景：抛出标准异常。  
头文件依赖：依赖 `<exception>`。

**<system_error>**（C++11）  
功能描述：系统错误码与异常，`std::error_code`、`std::error_category`、`std::system_error`。  
典型场景：处理操作系统返回的错误码。  
头文件依赖：依赖 `<stdexcept>`、`<cerrno>`。

**<limits>**  
功能描述：数值极限模板 `std::numeric_limits<T>`，提供类型安全的数值属性查询。  
典型场景：获取浮点精度、整数范围等。  
头文件依赖：通常独立。

**<cstddef>**  
功能描述：已在 C 标准库中介绍，此处提供 `size_t`、`ptrdiff_t` 等基本类型定义。

---

## 总括说明

- **分类原则**：本文档按 C++ 标准库的历史与功能逻辑划分为 **C 标准库**、**STL 核心组件**、**扩展库** 三大部分，每部分下按功能细分，形成清晰的目录结构。
- **现代 C++ 演进**：C++11 及后续标准不断扩充标准库，引入移动语义、lambda、智能指针、并发、文件系统等现代特性，使 C++ 标准库更加强大和安全。本文档标注了每个头文件引入的版本（C++11/14/17/20）。
- **使用建议**：
  - 优先使用标准库而非手动实现底层结构。
  - 利用 STL 算法和容器提升开发效率。
  - 根据需求选择合适的组件（如并发用 `std::thread`，文件操作用 `<filesystem>`）。
  - 注意 C 标准库函数在现代 C++ 中通常有更安全的替代（如 `std::string` 代替 `strcpy`）。

C++ 标准库是 C++ 语言的基石，深入理解其结构和用法是成为专业 C++ 开发者的必经之路。本总介绍可作为快速查阅和系统学习的参考手册。