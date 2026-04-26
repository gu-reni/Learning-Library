## `<cstring>` 头文件详解

`<cstring>` 是 C++ 标准库中从 C 标准库继承而来的头文件，提供了**C 风格字符串**（以 `'\0'` 结尾的字符数组）和**原始内存块**的处理函数。它包含了字符串操作（长度、复制、连接、比较、查找）、内存操作（设置、复制、移动、比较）等。虽然 C++ 推荐使用 `std::string`，但在底层操作、性能敏感场景或与 C 代码交互时仍然广泛使用。

---

## 一、函数详解

以下函数均在 `std` 命名空间中。大多数函数接受 `char*` 类型参数，且假定字符串以空字符结尾。

### 1. 字符串长度

#### `std::strlen`

**函数原型：**
```cpp
size_t strlen(const char* str);
```

**作用：** 返回字符串 `str` 的长度，即从起始到第一个 `'\0'`（不包含）的字符数。

**参数：** 指向以空字符结尾的字符串的指针。

**返回值：** 字符串的长度（类型 `size_t`）。

**示例用法：**
```cpp
const char* s = "Hello";
size_t len = std::strlen(s);  // 5
```

**实现原理：** 利用处理器的字节扫描指令（如 x86 的 `repne scasb`）或逐字节循环查找 `'\0'`。

**线程安全提示：** 只读操作，线程安全。但传入的指针必须有效。

---

### 2. 字符串复制

#### `std::strcpy`

**函数原型：**
```cpp
char* strcpy(char* dest, const char* src);
```

**作用：** 将源字符串 `src`（包括终止符 `'\0'`）复制到目标缓冲区 `dest` 中。要求 `dest` 有足够的空间容纳 `src`，否则导致缓冲区溢出（未定义行为）。

**参数：**
- `dest`：指向目标缓冲区的指针。
- `src`：指向源字符串的指针。

**返回值：** 返回 `dest`。

**实现原理：** 逐字符复制，直到遇到 `'\0'`。

**线程安全提示：** 若两个线程操作不同内存区域则安全；若操作重叠区域或同一区域则需同步。通常不推荐在复杂多线程环境中依赖该函数的结果顺序。

> **安全警告：** 此函数极易导致缓冲区溢出，应优先使用 `strncpy` 或 C++ 的 `std::string`。

---

#### `std::strncpy`

**函数原型：**
```cpp
char* strncpy(char* dest, const char* src, size_t count);
```

**作用：** 从 `src` 最多复制 `count` 个字符到 `dest`。若 `src` 长度小于 `count`，则剩余位置用 `'\0'` 填充；若 `src` 长度 ≥ `count`，则不会在 `dest` 末尾自动添加 `'\0'`（可能导致未终止的字符串）。

**返回值：** 返回 `dest`。

**注意事项：** 结果可能不以 `'\0'` 结尾，需要手动添加或使用 `strlcpy`（非标准）。

---

#### `std::memcpy`

**函数原型：**
```cpp
void* memcpy(void* dest, const void* src, size_t count);
```

**作用：** 从 `src` 复制 `count` 个字节到 `dest`。源和目标内存区域不能重叠。

**返回值：** 返回 `dest`。

**实现原理：** 通常使用字对齐的高效复制（如 `rep movsb`）。

---

#### `std::memmove`

**函数原型：**
```cpp
void* memmove(void* dest, const void* src, size_t count);
```

**作用：** 与 `memcpy` 类似，但允许源和目标区域重叠。使用临时缓冲区或反向复制保证正确性。

**返回值：** 返回 `dest`。

---

### 3. 字符串连接

#### `std::strcat`

**函数原型：**
```cpp
char* strcat(char* dest, const char* src);
```

**作用：** 将 `src` 追加到 `dest` 的末尾（覆盖 `dest` 的 `'\0'` 并添加终止符）。要求 `dest` 有足够空间容纳结果。

**返回值：** 返回 `dest`。

**安全警告：** 同样存在缓冲区溢出风险，推荐使用 `strncat`。

---

#### `std::strncat`

**函数原型：**
```cpp
char* strncat(char* dest, const char* src, size_t count);
```

**作用：** 从 `src` 最多追加 `count` 个字符到 `dest` 末尾，然后添加 `'\0'`。`dest` 的总长度应至少为 `strlen(dest) + min(count, strlen(src)) + 1`。

---

### 4. 字符串比较

#### `std::strcmp`

**函数原型：**
```cpp
int strcmp(const char* lhs, const char* rhs);
```

**作用：** 按字典序比较两个字符串。返回负值（`lhs < rhs`）、0（相等）或正值（`lhs > rhs`）。

**实现原理：** 逐字符比较直到不等或遇到 `'\0'`。

---

#### `std::strncmp`

**函数原型：**
```cpp
int strncmp(const char* lhs, const char* rhs, size_t count);
```

**作用：** 比较前 `count` 个字符（或直到遇到 `'\0'`），按字典序。

---

### 5. 字符串查找

#### `std::strchr`

**函数原型：**
```cpp
char* strchr(const char* str, int ch);
```

**作用：** 在字符串 `str` 中查找字符 `ch`（转换为 `char`）第一次出现的位置。包含终止符 `'\0'`（可查找 `'\0'`）。

**返回值：** 指向第一个匹配字符的指针，未找到返回 `NULL`。

---

#### `std::strrchr`

**函数原型：**
```cpp
char* strrchr(const char* str, int ch);
```

**作用：** 查找字符 `ch` 最后一次出现的位置。

---

#### `std::strstr`

**函数原型：**
```cpp
char* strstr(const char* haystack, const char* needle);
```

**作用：** 在 `haystack` 中查找子串 `needle` 第一次出现的位置。

**返回值：** 指向匹配子串首字符的指针，未找到返回 `NULL`。

---

#### `std::strspn` / `std::strcspn`

**函数原型：**
```cpp
size_t strspn(const char* dest, const char* src);
size_t strcspn(const char* dest, const char* src);
```

**作用：**
- `strspn`：返回 `dest` 起始连续匹配 `src` 中任意字符的最大长度。
- `strcspn`：返回 `dest` 起始连续不匹配 `src` 中任意字符的最大长度。

---

#### `std::strpbrk`

**函数原型：**
```cpp
char* strpbrk(const char* dest, const char* breakset);
```

**作用：** 在 `dest` 中查找第一个属于 `breakset` 中任意字符的位置。

---

#### `std::strtok`

**函数原型：**
```cpp
char* strtok(char* str, const char* delim);
```

**作用：** 将字符串分隔为令牌（token）。首次调用传入目标字符串，后续调用传入 `NULL`。使用静态内部指针，**非线程安全**。

**返回值：** 当前令牌的起始指针，没有更多令牌时返回 `NULL`。

**线程安全提示：** 使用静态缓冲区，不可重入。多线程环境下应使用 `strtok_r`（POSIX）或 C++ 的 `std::stringstream` / `std::regex`。

---

### 6. 内存操作

#### `std::memset`

**函数原型：**
```cpp
void* memset(void* ptr, int value, size_t num);
```

**作用：** 将 `ptr` 指向的内存区域的前 `num` 个字节设置为 `value`（转换为 `unsigned char`）。

---

#### `std::memcmp`

**函数原型：**
```cpp
int memcmp(const void* lhs, const void* rhs, size_t count);
```

**作用：** 比较两个内存块的前 `count` 个字节。

**返回值：** 类似 `strcmp`，按无符号字符比较。

---

#### `std::memchr`

**函数原型：**
```cpp
void* memchr(const void* ptr, int ch, size_t count);
```

**作用：** 在内存块中查找第一次出现 `ch`（转换为 `unsigned char`）的位置。

---

## 二、结构体详解

`<cstring>` 不定义任何结构体。

---

## 三、宏定义详解

`<cstring>` 不定义任何宏。相关常量（如 `NULL`）定义在 `<cstddef>` 中。

---

## 四、类型定义

`<cstring>` 不定义新类型，但使用 `size_t`（来自 `<cstddef>`）。

---

## 五、模板声明

`<cstring>` 是纯 C 风格头文件，不包含 C++ 模板。

---
