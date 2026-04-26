## `<algorithm>` 头文件详解

`<algorithm>` 是 C++ 标准模板库（STL）中定义的头文件，提供了大量**通用算法**。这些算法以迭代器为接口，可应用于任何序列容器（如 `vector`、`list`、`deque`、内置数组等）以及满足迭代器要求的自定义序列。算法涵盖了排序、查找、变换、复制、移除、排列、分区、二分搜索、集合运算等几乎所有常见的数据处理需求，是 STL 极为重要的组成部分。

---

## 一、不修改序列的操作

这些算法不会修改容器中的元素顺序和值。

### 1. 查找

| 函数 | 作用 |
|------|------|
| `std::find(begin, end, value)` | 返回第一个等于 `value` 的迭代器，否则返回 `end` |
| `std::find_if(begin, end, pred)` | 返回第一个使谓词 `pred` 为真的迭代器 |
| `std::find_if_not(begin, end, pred)` (C++11) | 返回第一个使谓词为假的迭代器 |
| `std::find_first_of(begin1, end1, begin2, end2)` | 在序列1中查找序列2中任一元素的第一次出现 |
| `std::find_end(begin1, end1, begin2, end2)` | 查找序列2在序列1中最后一次出现的位置 |
| `std::adjacent_find(begin, end)` | 查找第一对相邻的相等元素 |
| `std::search(begin1, end1, begin2, end2)` | 查找子序列第一次出现的位置 |
| `std::search_n(begin, end, count, value)` | 查找连续 `count` 个 `value` 的第一次出现 |

### 2. 计数

| 函数 | 作用 |
|------|------|
| `std::count(begin, end, value)` | 返回等于 `value` 的元素个数 |
| `std::count_if(begin, end, pred)` | 返回使谓词为真的元素个数 |

### 3. 遍历与检查

| 函数 | 作用 |
|------|------|
| `std::for_each(begin, end, func)` | 对每个元素调用函数 `func` |
| `std::for_each_n(begin, n, func)` (C++17) | 对前 `n` 个元素调用函数 |
| `std::all_of(begin, end, pred)` | 是否所有元素都满足谓词 |
| `std::any_of(begin, end, pred)` | 是否存在至少一个元素满足谓词 |
| `std::none_of(begin, end, pred)` | 是否没有元素满足谓词 |
| `std::equal(begin1, end1, begin2)` | 判断两个区间是否逐元素相等 |
| `std::lexicographical_compare(begin1, end1, begin2, end2)` | 按字典序比较两个区间 |

### 4. 匹配与查找

| 函数 | 作用 |
|------|------|
| `std::mismatch(begin1, end1, begin2)` | 返回第一对不匹配元素的位置 |

---

## 二、修改序列的操作

这些算法会修改容器中的元素（但通常不改变容器的大小，除非使用插入迭代器）。

### 1. 复制

| 函数 | 作用 |
|------|------|
| `std::copy(begin, end, out)` | 将元素复制到输出范围 |
| `std::copy_if(begin, end, out, pred)` (C++11) | 复制满足谓词的元素 |
| `std::copy_n(begin, n, out)` | 复制前 `n` 个元素 |
| `std::copy_backward(begin, end, out_end)` | 反向复制元素（适用于重叠区间） |

### 2. 移动

| 函数 | 作用 |
|------|------|
| `std::move(begin, end, out)` (C++11) | 移动元素 |
| `std::move_backward(begin, end, out_end)` (C++11) | 反向移动元素 |

### 3. 赋值与填充

| 函数 | 作用 |
|------|------|
| `std::fill(begin, end, value)` | 将区间所有元素设为 `value` |
| `std::fill_n(begin, n, value)` | 将前 `n` 个元素设为 `value` |
| `std::generate(begin, end, gen)` | 调用生成器函数为每个元素赋值 |
| `std::generate_n(begin, n, gen)` | 为前 `n` 个元素调用生成器 |

### 4. 替换

| 函数 | 作用 |
|------|------|
| `std::replace(begin, end, old_val, new_val)` | 将等于 `old_val` 的元素替换为 `new_val` |
| `std::replace_if(begin, end, pred, new_val)` | 替换满足谓词的元素 |
| `std::replace_copy(begin, end, out, old_val, new_val)` | 复制并替换 |
| `std::replace_copy_if(begin, end, out, pred, new_val)` | 复制并替换满足谓词的元素 |

### 5. 交换与变换

| 函数 | 作用 |
|------|------|
| `std::swap(a, b)` | 交换两个对象的值 |
| `std::swap_ranges(begin1, end1, begin2)` | 逐个交换两个区间元素 |
| `std::iter_swap(it1, it2)` | 交换两个迭代器指向的元素 |
| `std::transform(begin, end, out, unary_op)` | 对每个元素应用一元运算并输出 |
| `std::transform(begin1, end1, begin2, out, binary_op)` | 对两个序列对应元素应用二元运算并输出 |

### 6. 移除

这些算法并不真正删除元素（容器大小不变），而是将不需要的元素移动到末尾，并返回新的逻辑末尾迭代器。通常配合容器的 `erase` 成员函数实现物理删除。

| 函数 | 作用 |
|------|------|
| `std::remove(begin, end, value)` | 移除等于 `value` 的元素 |
| `std::remove_if(begin, end, pred)` | 移除满足谓词的元素 |
| `std::remove_copy(begin, end, out, value)` | 复制时移除等于 `value` 的元素 |
| `std::remove_copy_if(begin, end, out, pred)` | 复制时移除满足谓词的元素 |
| `std::unique(begin, end)` | 移除相邻重复元素 |
| `std::unique_copy(begin, end, out)` | 复制时移除相邻重复元素 |

### 7. 反转与旋转

| 函数 | 作用 |
|------|------|
| `std::reverse(begin, end)` | 反转区间顺序 |
| `std::reverse_copy(begin, end, out)` | 反转并复制 |
| `std::rotate(begin, middle, end)` | 将 `[middle, end)` 移动到区间开头 |
| `std::rotate_copy(begin, middle, end, out)` | 旋转并复制 |

### 8. 随机打乱

| 函数 | 作用 |
|------|------|
| `std::random_shuffle(begin, end)` | 随机打乱（C++14 起已废弃） |
| `std::shuffle(begin, end, rng)` (C++11) | 用随机数生成器打乱 |

---

## 三、分区操作

| 函数 | 作用 |
|------|------|
| `std::partition(begin, end, pred)` | 将满足谓词的元素放在前面，不满足的放在后面 |
| `std::stable_partition(begin, end, pred)` | 稳定分区（保持相对顺序） |
| `std::partition_copy(begin, end, out_true, out_false, pred)` | 分区并复制到两个输出 |
| `std::is_partitioned(begin, end, pred)` (C++11) | 检查区间是否已分区 |
| `std::partition_point(begin, end, pred)` (C++11) | 在已分区区间中找到分界点 |

---

## 四、排序与相关操作

### 1. 排序

| 函数 | 作用 |
|------|------|
| `std::sort(begin, end)` | 升序排序（平均 O(n log n)） |
| `std::sort(begin, end, comp)` | 自定义比较器排序 |
| `std::stable_sort(begin, end)` | 稳定排序 |
| `std::partial_sort(begin, middle, end)` | 部分排序：将最小的 `(middle-begin)` 个元素放在 `[begin, middle)`，其余无序 |
| `std::partial_sort_copy(srcBegin, srcEnd, destBegin, destEnd)` | 部分排序并将结果复制到目标范围 |
| `std::nth_element(begin, nth, end)` | 将第 `nth` 小的元素放在其正确位置，左侧元素 ≤ 它，右侧 ≥ 它 |

### 2. 堆操作（堆是最大堆）

| 函数 | 作用 |
|------|------|
| `std::make_heap(begin, end)` | 将区间转换为堆 |
| `std::push_heap(begin, end)` | 将最后一个元素加入堆 |
| `std::pop_heap(begin, end)` | 将最大元素移到末尾，并重新调整堆 |
| `std::sort_heap(begin, end)` | 对堆进行排序（排序后不再是堆） |
| `std::is_heap(begin, end)` (C++11) | 检查是否为堆 |
| `std::is_heap_until(begin, end)` (C++11) | 返回第一个破坏堆性质的位置 |

### 3. 二分搜索（要求区间已排序）

| 函数 | 作用 |
|------|------|
| `std::lower_bound(begin, end, value)` | 返回第一个 **不小于** `value` 的位置 |
| `std::upper_bound(begin, end, value)` | 返回第一个 **大于** `value` 的位置 |
| `std::equal_range(begin, end, value)` | 返回 `lower_bound` 和 `upper_bound` 组成的 `pair` |
| `std::binary_search(begin, end, value)` | 返回是否找到等于 `value` 的元素 |

### 4. 合并与集合操作（要求区间已排序）

| 函数 | 作用 |
|------|------|
| `std::merge(begin1, end1, begin2, end2, out)` | 合并两个有序序列 |
| `std::inplace_merge(begin, middle, end)` | 原地合并两个相邻有序段 |
| `std::includes(begin1, end1, begin2, end2)` | 判断序列2是否完全包含在序列1中（有序） |
| `std::set_difference` | 差集 |
| `std::set_intersection` | 交集 |
| `std::set_symmetric_difference` | 对称差集 |
| `std::set_union` | 并集 |

### 5. 排列操作

| 函数 | 作用 |
|------|------|
| `std::next_permutation(begin, end)` | 生成下一个字典序排列（含重复元素） |
| `std::prev_permutation(begin, end)` | 生成上一个字典序排列 |
| `std::is_permutation(begin1, end1, begin2)` (C++11) | 判断一个序列是否为另一个的排列 |

---

## 五、查询最小/最大值

| 函数 | 作用 |
|------|------|
| `std::min(a, b)` | 返回较小值 |
| `std::max(a, b)` | 返回较大值 |
| `std::minmax(a, b)` (C++11) | 返回 `pair`，同时得到最小和最大值 |
| `std::min_element(begin, end)` | 返回最小元素的迭代器 |
| `std::max_element(begin, end)` | 返回最大元素的迭代器 |
| `std::minmax_element(begin, end)` (C++11) | 同时返回最小和最大元素的迭代器 |
| `std::clamp(value, low, high)` (C++17) | 将值限定在 `[low, high]` 内 |

---

## 六、其他实用算法

| 函数 | 作用 |
|------|------|
| `std::iota(begin, end, start)` (C++11) | 用连续递增的 `start, start+1, ...` 填充区间 |
| `std::sample(begin, end, out, n, rng)` (C++17) | 从序列中随机抽取 `n` 个样本 |
| `std::iter_swap` (已包含在交换中) | |

---

## 七、宏与常量

`<algorithm>` 头文件中没有定义任何宏。

---

## 八、类型定义

`<algorithm>` 不定义新类型，但广泛使用迭代器和函数对象类型。

---

## 九、模板声明

`<algorithm>` 包含大量的函数模板，几乎每个算法都是一个函数模板。上述列表中所有函数都是模板。

---

