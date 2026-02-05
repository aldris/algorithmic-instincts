# Ch3 — 二分查找与双指针 (Binary Search & Two Pointers)

面试最容易在「细节」上挂的一章：边界、等号、空数组。这一章主要是**不变量 (invariant)** 和**边界控制**，和递归是另一块脑区。

---

## 识别信号 (Recognition Signals)

| 题目信号 | 想到的方法 |
|----------|------------|
| 「有序」「排序数组」「找第一个/最后一个」 | 二分查找（下标或答案空间） |
| 「连续子数组/子串」「和/个数满足某条件」 | 滑动窗口 或 前缀和+哈希 |
| 「两数之和」「两下标」 | 有序则相向双指针；无序则哈希 值→下标 |
| 「反转链表」「合并链表」「判环」 | 链表：dummy、prev/curr、快慢指针 |
| 「子数组和为 K」 | 前缀和 + 哈希（prefix→count 或 index） |

---

## 排序 (sorting)

二分和双指针常常建立在**有序**上。先能写「排序」和「划分」，再写「在有序上二分」「在有序上双指针」。

- **何时排序** — 题目说「排序数组」、或需要「顺序」才能用二分/双指针时，先 sort。
- **快速排序划分 (partition)** — 选 pivot，把小于的放左边、大于的放右边；用于 quick sort 或「第 k 大」类题。
- **归并排序 (merge sort)** — 分治：从中间拆开，左右分别排序，再合并；见 Ch0 分治。
- **何时用计数排序 / 桶** — 范围小、整数，可 O(n) 时考虑。

```python
# 快速排序划分：返回 pivot 最终下标
def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo
    for j in range(lo, hi):
        if arr[j] <= pivot:
            arr[i], arr[j] = arr[j], arr[i]
            i += 1
    arr[i], arr[hi] = arr[hi], arr[i]
    return i
```

---

## 什么时候想到二分查找 (binary search)

- **有序（或可排序）** — 数组有序、或排序后问题可解
- **问法** — 「找第一个/最后一个满足条件的」「最小/最大的使得……」「是否存在」
- **O(log n) 提示** — 数据很大、要求对数时间，常是二分

两种用法：**在下标上二分**（在有序数组里找位置）、**在答案值上二分**（二分答案，再 check 是否可行）。

---

## 手写模板 (Whiteboard Templates)

**二分（左闭右开 [L,R)）：**
```
L, R = 0, len(arr)   # R 不取
while L < R:
    mid = (L + R) // 2
    if check(mid): R = mid   # 找第一个满足
    else:          L = mid + 1
return L
```
**找最后一个满足：** mid 取 `(L+R+1)//2`，满足则 L=mid，否则 R=mid-1。

**滑动窗口：** `for R: 扩右、更新窗口 → while 不合法: 缩左 → 更新答案`

**前缀和+K：** `cnt[0]=1`，扫一遍 `prefix+=x`，`ans+=cnt[prefix-k]`，`cnt[prefix]+=1`

---

## 搜索不变量 (search invariant)

二分时，要明确「**循环不变量**」：每一轮后，**答案一定在哪个区间里**。

- **左闭右开 [L, R)** — 常见写法：`while L < R`，`mid = (L+R)//2`，根据 check(mid) 决定 `R = mid` 或 `L = mid + 1`。结束时 L == R，即唯一候选。
- **左闭右闭 [L, R]** — `while L <= R`，`mid = (L+R)//2`，`R = mid - 1` 或 `L = mid + 1`。结束时 L > R，注意最终答案取 L 还是 R。
- **找第一个满足的** — 若 mid 满足，答案可能是 mid 或更左，所以 R = mid（或 L 不动）；若不满足，L = mid + 1。
- **找最后一个满足的** — 若 mid 满足，答案可能是 mid 或更右，所以 L = mid（或 R 不动）；若不满足，R = mid - 1。注意 mid 取 `(L+R+1)//2` 避免死循环。

```python
# 在有序数组里找第一个 >= target 的下标（左闭右开）
def first_ge(arr, target):
    L, R = 0, len(arr)
    while L < R:
        mid = (L + R) // 2
        if arr[mid] < target:
            L = mid + 1
        else:
            R = mid
    return L
```

---

## 在答案空间上二分 (binary search on answer)

有时「答案」是一个数值（如最小容量、最大边长），且「给定一个值，能否达到」可以 O(n) 判断。这时**二分这个数值**，每次 check(mid) 是否可行。

- **单调性** — 若 mid 可行，则比 mid 大/小的一侧也可行（取决于题目），才能二分。
- **模板** — 二分范围 [lo, hi]，每次 mid，check(mid) 决定往左还是往右缩小区间。

---

## 双指针 / 滑动窗口 (two pointers / sliding window)

- **同向双指针 (sliding window)** — 两个指针都往右走；维护一段**连续区间**满足某条件（如和 <= K、最多 K 个不同元素）。右指针扩、左指针收；根据题目维护窗口内的信息（和、计数、哈希表）。
- **相向双指针** — 一头一尾往中间走；用于**有序数组**找两数之和、判断回文等。
- **快慢指针 (fast/slow)** — 链表上找环、找中点、找倒数第 k 个等。

```python
# 滑动窗口：和 >= target 的最短子数组长度
def min_subarray_sum(arr, target):
    L = 0
    cur = 0
    best = float('inf')
    for R in range(len(arr)):
        cur += arr[R]
        while cur >= target:
            best = min(best, R - L + 1)
            cur -= arr[L]
            L += 1
    return best if best != float('inf') else 0
```

---

## 链表 (linked list)

- **何时** — 反转、合并、判环、找交点、dummy 头节点
- **技巧** — **dummy 节点** 简化头节点处理；**快慢指针** 判环、找中点；**prev/curr** 反转、删除

```python
# 反转链表
def reverse_list(head):
    prev = None
    while head:
        nxt = head.next
        head.next = prev
        prev = head
        head = nxt
    return prev
```

---

## 前缀和 + 哈希 (prefix sum + hash)

- **何时** — 「子数组和为 K」「两数之和」「连续子数组满足某条件」
- **思路** — 前缀和 prefix[i] = sum(arr[0..i])；若 prefix[j] - prefix[i] = K，则 arr[i+1..j] 和为 K。用哈希表存「前缀和 → 出现次数或最早下标」，边扫边查。

```python
# 子数组和为 K 的个数
def subarray_sum_k(arr, k):
    from collections import defaultdict
    cnt = defaultdict(int)
    cnt[0] = 1
    prefix = 0
    ans = 0
    for x in arr:
        prefix += x
        ans += cnt[prefix - k]
        cnt[prefix] += 1
    return ans
```

---

## 复杂度直觉 (Complexity Instincts)

| 操作 | 时间 | 空间 | 一眼看出 |
|------|------|------|----------|
| 二分（下标或答案） | O(log n) × check 成本 | O(1) | 每次砍半 |
| 滑动窗口 | O(n) | O(窗口内信息，如 set 大小) | 左右指针各走一遍 |
| 前缀和+哈希 | O(n) | O(n) | 扫一遍 + 查表 O(1) |
| 链表一次遍历 | O(n) | O(1) | 指针走一遍 |

---

## 例题 (LeetCode)

- **LC 704 二分查找 (Binary Search)** — 场景：有序数组找 target。为何用：有序 + 找位置 = 二分。如何用：L,R 维护候选区间，mid 比较 target 决定缩左或缩右。
- **LC 3 无重复字符的最长子串 (Longest Substring Without Repeating)** — 场景：子串内无重复字符，求最长长度。为何用：连续子串 + 满足某条件 = 滑动窗口。如何用：右指针扩，用 set 记窗口内字符，冲突则左指针缩直到无重复。
- **LC 1 两数之和 (Two Sum)** — 场景：找两数之和为 target 的下标。为何用：需要「是否出现过 complement」且要下标 = 哈希表存值→下标。如何用：扫一遍，查 target-num 是否在 map 里，有则返回；否则把 num→i 放入 map。
- **LC 206 反转链表 (Reverse Linked List)** — 场景：原地反转单链表。为何用：链表 + 反转 = prev/curr 指针。如何用：prev=None，每次 curr.next=prev，再 prev,curr=curr,next。

---

## 失败模式 (Failure Modes)

**原内容：**

- **二分边界** — 死循环：找「最后一个满足」时 mid 要取 `(L+R+1)//2`；最终答案是 L 还是 R 要统一。
- **滑动窗口** — 左指针什么时候停、窗口含义（开区间/闭区间）要清楚；先扩右再缩左，别漏状态。
- **空数组 / 单元素** — 先判断 `len(arr) < 2` 等，避免越界。

**补充：**

- **常见误解** — 「二分只能查相等」：可查第一个≥、最后一个≤等；「滑动窗口一定 O(n)」：若窗口内要排序或复杂维护，可能更贵。
- **面试易挂点** — 二分死循环（最后一个满足没写 (L+R+1)//2）；窗口条件写反（该缩左时没缩）；前缀和漏 cnt[0]=1。
- **典型反例** — 找「最后一个≤target」若用 mid=(L+R)//2 且满足时 L=mid：L 可能不变，死循环。

---

## 跨章节联系 (Cross-Chapter Links)

- **Ch0 分治** — 二分 = 分治的一种（只走一边，无合并）；归并排序 = 分治 + 合并。
- **Ch4 数据结构** — 堆维护 Top K；单调栈「下一个更大」；本 chapter 偏线性扫描与二分。
- **Ch6 DP** — 若子数组问题有重叠子问题可转 DP；本 chapter 偏不重叠的「区间/窗口」处理。
