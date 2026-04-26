## `<memory>` 头文件详解

`<memory>` 是 C++ 标准库中提供**智能指针、内存管理工具和分配器**的头文件。它定义了 `std::unique_ptr`、`std::shared_ptr`、`std::weak_ptr` 三种智能指针，以及 `std::allocator`、`std::addressof`、`std::align`、`std::pointer_traits` 等辅助设施。该头文件是 RAII 和现代 C++ 内存安全管理的核心，大幅度减少了手动 `new`/`delete` 带来的资源泄漏风险。

---

## 一、智能指针

### 1. `std::unique_ptr`

**模板参数：**
```cpp
template<class T, class Deleter = std::default_delete<T>>
class unique_ptr;
template<class T, class Deleter>
class unique_ptr<T[], Deleter>;   // 数组偏特化
```

**作用：** 独占所有权的智能指针。同一时刻只能有一个 `unique_ptr` 指向对象。离开作用域时自动销毁对象。

**主要成员函数：**

| 成员 | 说明 |
|------|------|
| `unique_ptr()` | 默认构造空指针 |
| `explicit unique_ptr(pointer p)` | 接管裸指针所有权 |
| `unique_ptr(unique_ptr&& u)` | 移动构造函数（转移所有权） |
| `~unique_ptr()` | 析构，释放所管理的对象 |
| `operator*()` / `operator->()` | 解引用 |
| `pointer get() const` | 返回裸指针 |
| `pointer release()` | 释放所有权，返回裸指针 |
| `void reset(pointer p = pointer())` | 释放当前对象并接管新对象 |
| `void swap(unique_ptr& u)` | 交换内容 |
| `explicit operator bool() const` | 检查是否非空 |

**数组特化额外提供 `operator[]`**：`T& operator[](size_t i) const`。

**辅助函数：**
- `std::make_unique<T>(args...)`（C++14）创建 `unique_ptr`。
- `std::make_unique_for_overwrite`（C++20）创建未初始化的 `unique_ptr`。

**示例：**
```cpp
auto p = std::make_unique<int>(42);
std::unique_ptr<int[]> arr = std::make_unique<int[]>(10);
```

---

### 2. `std::shared_ptr`

**模板参数：**
```cpp
template<class T> class shared_ptr;
```

**作用：** 共享所有权的智能指针。通过引用计数管理对象的生命周期，当最后一个 `shared_ptr` 被销毁时释放对象。

**主要成员函数：**

| 成员 | 说明 |
|------|------|
| `shared_ptr()` | 默认构造空指针 |
| `shared_ptr(Y* ptr)` | 接管裸指针所有权 |
| `shared_ptr(const shared_ptr& r)` | 拷贝构造，增加引用计数 |
| `shared_ptr(shared_ptr&& r)` | 移动构造，不增加引用计数 |
| `~shared_ptr()` | 析构，减少引用计数 |
| `operator*()` / `operator->()` | 解引用 |
| `T* get() const` | 返回裸指针 |
| `long use_count() const` | 返回引用计数 |
| `bool unique() const` | 检查是否独占（C++17 前） |
| `void reset()` / `void reset(Y* ptr)` | 重置 |
| `void swap(shared_ptr& r)` | 交换内容 |

**辅助函数：**
- `std::make_shared<T>(args...)`：分配内存并构造对象，效率更高（一次分配控制块和对象）。
- `std::allocate_shared<T>(alloc, args...)`：使用自定义分配器。
- `std::static_pointer_cast` / `dynamic_pointer_cast` / `const_pointer_cast` / `reinterpret_pointer_cast`：对 `shared_ptr` 进行类型转换。

**示例：**
```cpp
auto sp = std::make_shared<int>(100);
auto sp2 = sp; // 引用计数变为 2
```

---

### 3. `std::weak_ptr`

**模板参数：**
```cpp
template<class T> class weak_ptr;
```

**作用：** 配合 `shared_ptr` 使用，不增加引用计数，用于解决循环引用问题。可以从 `weak_ptr` 构造 `shared_ptr`（若对象仍存在）。

**主要成员函数：**

| 成员 | 说明 |
|------|------|
| `weak_ptr()` | 默认构造空指针 |
| `weak_ptr(const weak_ptr& r)` | 拷贝构造 |
| `weak_ptr(const shared_ptr<T>& r)` | 从 `shared_ptr` 构造 |
| `~weak_ptr()` | 析构 |
| `void swap(weak_ptr& r)` | 交换 |
| `void reset()` | 清空 |
| `long use_count() const` | 返回所管理对象的引用计数 |
| `bool expired() const` | 判断所管理的对象是否已被销毁 |
| `shared_ptr<T> lock() const` | 返回一个指向相同对象的 `shared_ptr`，若已销毁则返回空 |

**示例：**
```cpp
std::weak_ptr<int> wp;
{
    auto sp = std::make_shared<int>(42);
    wp = sp;
    std::cout << wp.use_count(); // 1
}
std::cout << wp.expired();       // true
```

---

## 二、分配器

### 1. `std::allocator<T>`

**模板参数：**
```cpp
template<class T> struct allocator;
```

**作用：** 默认分配器，用于容器和智能指针的内存分配。提供 `allocate`、`deallocate`、`construct`（C++17 前）、`destroy`（C++17 前）等成员。

**主要成员函数（C++17 后）：**

| 成员 | 说明 |
|------|------|
| `T* allocate(size_t n)` | 分配 `n * sizeof(T)` 字节的未初始化内存 |
| `void deallocate(T* p, size_t n)` | 释放内存 |
| `bool operator==(const allocator&)` | 比较相等性 |

**C++20 起增加 `allocate_at_least` 返回 `allocation_result`。**

**示例：**
```cpp
std::allocator<int> alloc;
int* p = alloc.allocate(10);
alloc.deallocate(p, 10);
```

### 2. `std::allocator_traits`

**定义：**
```cpp
template<class Alloc> struct allocator_traits;
```

**作用：** 统一分配器接口，即使分配器未提供某些成员（如 `construct`），`allocator_traits` 也会提供默认版本。

**常见别名与静态函数：**
- `value_type`
- `pointer`, `const_pointer`
- `size_type`, `difference_type`
- `rebind_alloc<U>`
- `static allocate(Alloc& a, size_t n)`
- `static deallocate(Alloc& a, pointer p, size_t n)`
- `static construct(Alloc& a, T* p, Args&&... args)`（C++17 前）
- `static destroy(Alloc& a, T* p)`（C++17 前）

---

## 三、内存工具

### 1. `std::addressof`

**函数原型：**
```cpp
template<class T>
T* addressof(T& arg) noexcept;
```

**作用：** 返回对象的真实地址，即使 `T` 重载了 `operator&` 也能正确获取地址。

**示例：**
```cpp
struct S { int operator&() const { return 0; } };
S s;
S* p = std::addressof(s); // 正确地址，而非 0
```

---

### 2. `std::align`

**函数原型：**
```cpp
void* align(std::size_t alignment, std::size_t size, void*& ptr, std::size_t& space);
```

**作用：** 在给定缓冲区中调整指针对齐。如果可能，将 `ptr` 调整为下一个对齐地址，并减去相应空间；否则返回 `nullptr`。

**返回值：** 成功返回调整后的 `ptr`，失败返回 `nullptr`。

---

### 3. `std::pointer_traits`

**模板参数：**
```cpp
template<class Ptr> struct pointer_traits;
```

**作用：** 提供与指针（智能指针或原始指针）相关的类型信息（如 `element_type`、`difference_type`）和 `pointer_to` 静态函数。类似于 `iterator_traits`。

---

### 4. `std::to_address`（C++20）

**函数原型：**
```cpp
template<class Ptr>
auto to_address(const Ptr& p) noexcept;
template<class T>
constexpr T* to_address(T* p) noexcept;
```

**作用：** 获取指针或智能指针的裸地址，即使 `Ptr` 重载了 `operator->()`。常用于算法实现。

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `std::default_delete<T>` | `unique_ptr` 的默认删除器 |
| `std::default_delete<T[]>` | 数组删除器偏特化 |
| `std::unique_ptr<T>` | 独占智能指针 |
| `std::shared_ptr<T>` | 共享智能指针 |
| `std::weak_ptr<T>` | 弱引用智能指针 |
| `std::allocator<T>` | 默认分配器 |
| `std::allocator_traits<Alloc>` | 分配器特征类 |
| `std::pointer_traits<Ptr>` | 指针特征类 |

---

## 五、模板声明

`<memory>` 中包含大量模板类和函数，上述已覆盖主要部分。此外还有：
- `std::enable_shared_from_this<T>`：允许对象生成指向自身的 `shared_ptr`。
- `std::bad_weak_ptr`：当从已销毁的 `weak_ptr` 构造 `shared_ptr` 时抛出的异常。
- `std::owner_less<T>`：用于按所有者比较 `shared_ptr` 和 `weak_ptr` 的仿函数。

---
