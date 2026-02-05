# Ch4 — 数据结构 (Data Structures)

两个角度：(1) **什么时候用哪种 DS**（决策）；(2) **如何实现**常见 DS（面试有时会考「实现一个堆 / LRU / Trie」）。

---

## 识别信号 (Recognition Signals)

| 题目信号 | 想到的方法 |
|----------|------------|
| 「第 K 大/小」「合并 K 个有序」「实时中位数」 | 堆（小根堆维护 K 大，大根堆维护 K 小） |
| 「下一个更大/更小元素」「直方图最大矩形」 | 单调栈（维护单调性，弹栈时得到结果） |
| 「前缀匹配」「是否以某串开头」「单词搜索」 | Trie（children + is_end） |
| 「LRU」「get/put 容量限制」「淘汰最久未用」 | 哈希表 + 双向链表（O(1) 查 + O(1) 更新顺序） |
| 「用栈实现队列」「用队列实现栈」 | 两个栈 / 队列倒腾 |

---

## 堆 (Heap)

- **何时用** — 第 K 大/小、合并 K 个有序链表、实时求中位数、按频率取前 K 个等。需要「动态取最值」时用堆。
- **实现（数组堆）** — 用数组存完全二叉树：父 `(i-1)//2`，左子 `2*i+1`，右子 `2*i+2`。**上浮 (siftUp)**：比父小就交换；**下沉 (siftDown)**：比子大就和更小的那个交换。插入时放末尾再 siftUp；弹出时根和末尾交换，弹出末尾，再对根 siftDown。

```python
# 小根堆（最小在根）
class MinHeap:
    def __init__(self):
        self.arr = []
    def push(self, x):
        self.arr.append(x)
        self._sift_up(len(self.arr) - 1)
    def pop(self):
        if not self.arr: return None
        self.arr[0], self.arr[-1] = self.arr[-1], self.arr[0]
        out = self.arr.pop()
        if self.arr:
            self._sift_down(0)
        return out
    def _sift_up(self, i):
        while i > 0:
            p = (i - 1) // 2
            if self.arr[i] >= self.arr[p]: break
            self.arr[i], self.arr[p] = self.arr[p], self.arr[i]
            i = p
    def _sift_down(self, i):
        n = len(self.arr)
        while True:
            small = i
            L, R = 2*i+1, 2*i+2
            if L < n and self.arr[L] < self.arr[small]: small = L
            if R < n and self.arr[R] < self.arr[small]: small = R
            if small == i: break
            self.arr[i], self.arr[small] = self.arr[small], self.arr[i]
            i = small
```

---

## 手写模板 (Whiteboard Templates)

**堆：** 父 `(i-1)//2`，左 `2*i+1`，右 `2*i+2`；push→末尾+siftUp；pop→根尾交换+pop+siftDown(0)。  
**单调栈（下一个更大）：** for i: while stack and arr[stack[-1]]<arr[i]: 弹栈并记录 result[j]=arr[i]；stack.append(i)。  
**Trie：** 节点 children+is_end；insert 沿路径建节点最后 is_end=True；search 走到底看 is_end；startsWith 走完前缀即可。  
**LRU：** map key→node；双向链表 head↔tail；get 时 _remove+_add(移到头)；put 时若存在先 _remove，_add 新节点，若 len>cap 则删 tail.prev。

---

## 单调栈 (Monotonic Stack)

- **何时用** — 「下一个更大/更小元素」「每个位置向左/右第一个比它大/小的位置」「直方图最大矩形」等。需要**一边扫一边维护「候选」序列**时，且候选有单调性时用单调栈。
- **实现** — 栈里存下标（或值）；维护栈内单调递增/递减。新元素进来时，不断弹栈直到满足单调性，弹的时候就可以得到「栈顶元素的下一个更大/更小就是当前元素」。

```python
# 每个位置的下一个更大元素（没有则为 -1）
def next_greater_element(arr):
    n = len(arr)
    result = [-1] * n
    stack = []
    for i in range(n):
        while stack and arr[stack[-1]] < arr[i]:
            j = stack.pop()
            result[j] = arr[i]
        stack.append(i)
    return result
```

---

## 字典树 (Trie)

- **何时用** — 前缀匹配、「是否以某串开头」、单词搜索、自动补全等。
- **实现** — 节点：`children`（dict 或 26 长数组）+ `is_end`（是否为一个单词的结尾）。insert：按字符往下走，没有就建子节点，最后标 is_end。search：按字符往下走，最后看 is_end。startsWith：按字符往下走，能走完前缀即可。

```python
class Trie:
    def __init__(self):
        self.children = {}
        self.is_end = False
    def insert(self, word):
        node = self
        for c in word:
            if c not in node.children:
                node.children[c] = Trie()
            node = node.children[c]
        node.is_end = True
    def search(self, word):
        node = self._go(word)
        return node is not None and node.is_end
    def startsWith(self, prefix):
        return self._go(prefix) is not None
    def _go(self, s):
        node = self
        for c in s:
            if c not in node.children: return None
            node = node.children[c]
        return node
```

---

## LRU 缓存 (LRU Cache)

- **何时用** — 题目直接要求「实现 LRU」或「get/put，容量限制，淘汰最久未用」。
- **实现** — **哈希表 (key → 节点)** + **双向链表 (按访问顺序)**。get：若存在则把该节点移到链表头，返回值。put：若存在则更新并移到头；若不存在则新建节点放头，若超过容量则删链表尾节点，并在哈希表里删掉对应 key。

```python
class LRUNode:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = {}
        self.head = LRUNode(0, 0)
        self.tail = LRUNode(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head
    def _add(self, node):
        node.next = self.head.next
        node.prev = self.head
        self.head.next.prev = node
        self.head.next = node
    def _remove(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev
    def get(self, key):
        if key not in self.cache: return -1
        node = self.cache[key]
        self._remove(node)
        self._add(node)
        return node.val
    def put(self, key, value):
        if key in self.cache:
            self._remove(self.cache[key])
        node = LRUNode(key, value)
        self._add(node)
        self.cache[key] = node
        if len(self.cache) > self.cap:
            old = self.tail.prev
            self._remove(old)
            del self.cache[old.key]
```

---

## 其他（用栈实现队列等）

- **用两个栈实现队列** — 一个栈 A 负责入队，一个栈 B 负责出队；出队时若 B 空则把 A 全部弹到 B，再弹 B 顶。
- **用队列实现栈** — 每次 push 后把队列里前面的元素依次出队再入队，使刚进的到队头；pop 就取队头。

---

## 复杂度直觉 (Complexity Instincts)

| DS | 操作 | 时间 | 空间 | 一眼看出 |
|----|------|------|------|----------|
| 堆 | push/pop | O(log n) | O(n) | 树高 log n |
| 单调栈 | 每元素入栈出栈各至多一次 | O(n) | O(n) | 扫一遍 |
| Trie | insert/search/startsWith | O(串长) | O(节点数) | 走一条路径 |
| LRU | get/put | O(1) | O(cap) | 哈希+链表头尾操作 |

---

## 例题 (LeetCode)

- **LC 215 数组中的第 K 个最大元素 (Kth Largest)** — 场景：未排序数组找第 k 大。为何用：动态取当前最大/最小 = 堆。如何用：维护大小为 k 的小根堆，比堆顶大就替换堆顶，最后堆顶即第 k 大。
- **LC 146 LRU 缓存 (LRU Cache)** — 场景：get/put，容量限制，淘汰最久未用。为何用：需要 O(1) 查 + O(1) 更新顺序 = 哈希表 + 双向链表。如何用：map 存 key→节点；get 时把节点移到链表头；put 时若满则删尾，新节点插头。
- **LC 208 实现 Trie (Implement Trie)** — 场景：插入单词、查单词是否存在、查前缀是否存在。为何用：前缀匹配 = Trie。如何用：节点 children + is_end；insert 沿路径建节点最后标 is_end；search 走到底看 is_end；startsWith 走完前缀即可。
- **LC 496 下一个更大元素 I (Next Greater Element)** — 场景：对 nums1 中每个元素，在 nums2 里找其右侧第一个更大的数。为何用：找「下一个更大」= 单调栈。如何用：从左到右扫 nums2，维护单调递减栈，弹栈时栈顶的「下一个更大」就是当前元素。

---

## 失败模式 (Failure Modes)

**原内容：**

- **堆** — 上浮/下沉时比较的是值，但交换的是下标对应的元素；父子下标别写错。
- **单调栈** — 栈里存下标还是值要统一；弹栈时先算完结果再弹。
- **Trie** — 区分 search（必须到单词结尾）和 startsWith（前缀即可）；空串要单独处理。
- **LRU** — 更新已存在的 key 时也要把节点移到头；容量满时删的是尾节点对应的 key。

**补充：**

- **常见误解** — 「堆顶就是最大值」：小根堆顶是最小；第 k 大用大小为 k 的小根堆。「LRU 用队列」：队列无法 O(1) 把中间元素移到头，需双向链表。
- **面试易挂点** — 堆父子下标写反；LRU 更新已存在 key 时忘记先 _remove 再 _add；Trie 的 search 和 startsWith 搞混。
- **典型反例** — LRU 若只在 put 新 key 时删尾：已存在的 key 被 put 更新时若没移到头，淘汰顺序会错。

---

## 跨章节联系 (Cross-Chapter Links)

- **Ch3 二分/双指针** — 第 K 大也可用快排划分 O(n)；堆适合动态流数据。前缀和+哈希在 Ch3。
- **Ch5 Graph** — Dijkstra 用堆；拓扑排序用队列/度。Union-Find 是另一种 DS。
- **Ch6 DP** — 某些「选择」问题用 DP；本 chapter 偏「维护动态集合」的 DS。
