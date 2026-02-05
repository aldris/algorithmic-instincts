# Ch0 — DFS / 递归

地基：先学「怎么想」，再学「想什么」。Tree / Graph / DP / Backtracking 都建立在这一章之上。

---

## 识别信号 (Recognition Signals)

| 题目信号 | 想到的方法 |
|----------|------------|
| 同样结构、更小规模（列表→rest，树→左右，n→n-1） | 递归 / DFS |
| 「所有组合」「所有路径」「是否存在路径」 | DFS + 枚举（回溯见 Ch2） |
| 「分两半」「合并结果」「子问题独立」 | 分治 (divide and conquer) |
| 要「层序」或「最短边数」（无权） | BFS |
| 要「任意路径」或「所有路径」 | DFS |

---

## 什么时候想到递归 / DFS

如果题目符合下面任一类，就可以考虑递归（或 DFS）：

- **同样结构、更小规模** — 整个问题的定义和「更小一部分」上的问题一样。例如：「列表求和」= 第一个元素 + 后面部分的求和；「树高」= 1 + max(左子树高, 右子树高)。你把规模缩小（列表→剩余部分，树→左/右子树，n→n-1）然后递归调用自己。
- **枚举所有选择 / 路径** — 必须尝试每一种选项或每一条路径。例如：「所有组合」、「从根到叶的所有路径」、「是否存在路径」。每次调用代表一种选择；对每个选项都递归一次。
- **分治（divide and combine）** — 把输入拆成两（或多）部分，分别递归求解，再合并答案。例如：归并排序（从中间拆开，排序两半，再合并）；二分查找（丢掉一半）。

---

## 递归执行模型 (pre / post / combine)

- **前序 (pre)** — 进入节点时做事（如 append、判断、更新）
- **后序 (post)** — 离开节点时做事（如 pop、还原）
- **合并 (combine)** — 子调用返回后做事（如 max(left, right)、merge）

**最小模板：** `if base: return base_val` → (pre) → `recurse` → (combine) → (post) → `return`

```python
def dfs(node):
    if not node: return 基准值          # base
    # pre: 进节点时
    path.append(node.val)
    left = dfs(node.left)               # recurse
    right = dfs(node.right)
    res = merge(left, right, node.val)   # combine
    path.pop()                         # post: 离开时还原
    return res
```

---

## 递归的三种结构

| 结构 | 特征 | 例子 |
|------|------|------|
| **线性递归** | f(n) 只调 f(n-1) 或 f(rest) | 爬楼梯、列表求和 |
| **树形递归** | 每个节点调左右/多子 | 树高、路径、LCA |
| **分治递归** | 左右独立 + combine，子问题不重叠 | 归并排序、二分 |

---

## 递归契约

每个递归函数都有两部分。这两部分写对，才不会无限递归或答案错。

- **基准情况（base case）** — 什么时候停？通常是：输入为空、只有一个元素、或 n=0。返回一个具体值（如 0、[]、None）。
- **递推关系（recurrence）** — 如何用「更小规模」的答案表示规模 n 的答案？「更小」必须最终到达基准（例如 n-1、列表的 rest、左/右子树）。

**模板：** `if 基准: return 基准值; else: 对更小规模递归; 合并; return`

```python
def sum_list(arr):
    if not arr: return 0           # 基准：空列表 → 和为 0
    return arr[0] + sum_list(arr[1:])  # 对 arr[1:] 递归，再和第一个元素合并
```

---

## 手写模板 (Whiteboard Templates)

**通用递归（白板可默写）：**

```
输入：当前规模（node / n / arr）
base：空/0/1 → return 具体值
recurse：对更小规模调用
状态传递：pass down（path, remain）或 return
合并：max/sum/and 或 combine(left, right)
return
```

**DFS 路径（需还原）：** `append → recurse 子 → pop`

**分治：** `split → left=f(左), right=f(右) → return merge(left, right)`

---

## 状态归属

递归时经常要「往下传信息」、「往上带信息」或「共享一个结果列表」。要明确状态归谁管：

- **往下传（pass down）** — 用参数传下去，如 `path`、`so_far`、`target - sum`。每次调用要么拿到自己的副本，要么你「改完再还原」。适合：「需要知道当前路径」或「需要剩余预算」。
- **全局 / 引用（global / ref）** — 所有调用共读共写的列表或变量（如 `result.append(path[:])`）。适合：「要收集所有路径」或「要全局最大值」。做回溯时，递归返回后一定要还原（如 `path.pop()`）。
- **返回值（return）** — 函数每次返回一个值（如「以当前节点为根的子树的最大深度」）。适合：父节点只需要每个子节点的一个值（如 max、sum、是否存在路径）。

```python
# 往下传：path 是共享的；进子节点前 append，离开后 pop（回溯）
def dfs_path(node, path, target):
    if not node: return
    path.append(node.val)
    if node.val == target: ...  # 使用 path
    dfs_path(node.left, path, target)
    dfs_path(node.right, path, target)
    path.pop()  # 还原，否则兄弟分支会看到不该有的节点

# 返回值：每次调用只返回一个值；无共享可变状态
def max_depth(node):
    if not node: return 0
    return 1 + max(max_depth(node.left), max_depth(node.right))
```

---

## DFS vs BFS — 什么时候用哪个

两者都会遍历到所有可达节点，差别是**访问顺序**，进而**第一次得到的结果**不同。

| | DFS | BFS |
|---|-----|-----|
| **何时用** | 关心「一条路径」、「所有路径」或「是否存在」。用栈（或递归调用栈）即可。 | 关心**层序**或**最短步数**（边权相同）。 |
| **结构** | 栈：后进先出（或递归 = 调用栈）。 | 队列：先进先出。 |
| **第一次访问到某节点时** | 是某条深度优先路径上的。 | 是从起点出发**边数最少**的路径。 |

结论：「无权图最短路径」→ BFS。「找任意路径」或「列出所有路径」→ DFS。

```python
# DFS（显式栈）— 若想写迭代版，用栈代替递归
def dfs_iter(start):
    stack = [start]
    while stack:
        u = stack.pop()
        for v in neighbors(u):
            stack.append(v)

# BFS（队列）— 第一次到达 target 时的距离就是最短边数
def bfs_shortest(start, target):
    from collections import deque
    q = deque([(start, 0)])
    while q:
        u, d = q.popleft()
        if u == target: return d
        for v in neighbors(u):
            q.append((v, d + 1))
```

---

## 分治（Divide and conquer）

把问题拆成**互不重叠**的子问题，分别递归，再合并。子问题之间不能像 DP 那样大量重叠；合并步骤要足够便宜（如 O(n) 的 merge）。

- **何时用：** 「给数组排序」（归并：从中间拆，排序两半，再 merge）。「两个有序数组里找第 k 大」（比较两段中点，丢掉一半）。子问题独立；合并是 O(n) 量级。
- **模板：** 拆 → left = f(左半), right = f(右半) → return combine(left, right)。

```python
def merge_sort(arr):
    if len(arr) <= 1: return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def quick_sort(arr, lo, hi):
    if lo >= hi: return
    p = partition(arr, lo, hi)   # 一次 partition 后 arr[p] 已在最终位置
    quick_sort(arr, lo, p - 1)
    quick_sort(arr, p + 1, hi)
```

---

## 复杂度直觉 (Complexity Instincts)

| 类型 | 时间典型范围 | 空间典型范围 | 一眼看出 |
|------|--------------|--------------|----------|
| **树形递归** | O(节点数) | O(树高)，最坏 O(n) | 每个节点访问一次 |
| **分治** | T(n)=2T(n/2)+O(n) → O(n log n) | O(log n) 栈 | Master：a=2,b=2,k=1 → log_b a=1=k |
| **回溯/枚举** | O(分支^深度)，如 2^n、n! | O(深度) | 每层分支数 × 深度 |
| **BFS/DFS 图** | O(V+E) | O(V) visited + 队列/栈 | 点+边各一遍 |

**Master 定理（简记）：** T(n)=aT(n/b)+O(n^k)。若 log_b a > k → O(n^{log_b a})；若 = k → O(n^k log n)；若 < k → O(n^k)。  
**何时必须改迭代：** 数据规模大、递归深度可能 > ~1000（Python 默认约 1000）时，用显式栈/队列写迭代版。

---

## Python 递归深度限制

- **默认深度** — 约 1000（`sys.getrecursionlimit()`）。
- **何时必须改成迭代** — 树/图节点数可能上万、或链很长时，递归易爆栈；改用显式 `stack`/`deque` 写 DFS/BFS。

---

## 例题 (LeetCode)

- **LC 70 爬楼梯 (Climb Stairs)** — 场景：每次 1 或 2 步，到 n 有几种方式。为何用：同样结构更小规模，f(n)=f(n-1)+f(n-2)。如何用：base 为 n<=1 返回 1；递归 + memo 或递推。
- **LC 200 岛屿数量 (Number of Islands)** — 场景：二维网格，相邻 1 为同一岛，求岛数。为何用：连通块 = 对每个未访问的 1 做一次 DFS 并标记。如何用：二重循环遇到 1 就 DFS 把整岛标成 0 或 visited，计数 +1。
- **LC 102 二叉树的层序遍历 (Binary Tree Level Order)** — 场景：按层输出节点值。为何用：层序 = BFS，第一次到达即该层。如何用：队列，每次处理当前层 len(q) 个节点，把子节点入队，当前层加入结果。

---

## 失败模式 (Failure Modes)

**原内容：**

- **没有基准，或递推不缩小** — 例如 f(n) 永远到不了 n=0。结果：无限递归、栈溢出。
- **改了共享状态但不还原** — 回溯里你先 `path.append(x)`，递归，返回后必须 `path.pop()`。忘了 pop，兄弟分支会看到不属于它们路径上的节点。
- **选错工具：BFS vs DFS** — 要「最短边数」（无权）？用 BFS。要「任意路径」或「所有路径」？用 DFS。用 DFS 求最短步数会错；用 BFS 枚举所有路径可以但通常比 DFS 笨。

**补充：**

- **常见误解** — 「递归一定慢」：有重叠子问题时才慢，加 memo 即 DP；「DFS 和 BFS 结果一样」：只保证都遍历到，顺序和「第一次到达」含义不同。
- **面试易挂点** — 忘记 base case 或递推不单调缩小；共享 path 不 pop；问「最短步数」却写 DFS。
- **典型反例** — 求「无权图最短路径」用 DFS：可能先走到一条很长路径，答案错；应用 BFS 第一次到达即最短。

---

## 跨章节联系 (Cross-Chapter Links)

- **Ch1 Tree** — 递归最干净形态，无 visited，子树 return + 合并。
- **Ch2 Backtracking** — DFS + 显式状态 + **rollback**（push → recurse → pop）。
- **Ch4 Graph** — Tree + **visited** + 环；同一套 DFS/BFS，多「入栈/入队时标记」。
- **Ch6 DP** — 递归 + **memo**（重叠子问题）；递推 = 自下而上填表。
- **Greedy vs DP** — 贪心：局部最优即全局最优；DP：要枚举子问题再合并，有重叠用 memo。
