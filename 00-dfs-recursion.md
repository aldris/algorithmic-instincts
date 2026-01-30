# Ch0 — DFS / 递归

地基：先学「怎么想」，再学「想什么」。Tree / Graph / DP / Backtracking 都建立在这一章之上。

---

## 什么时候想到递归 / DFS

如果题目符合下面任一类，就可以考虑递归（或 DFS）：

- **同样结构、更小规模** — 整个问题的定义和「更小一部分」上的问题一样。例如：「列表求和」= 第一个元素 + 后面部分的求和；「树高」= 1 + max(左子树高, 右子树高)。你把规模缩小（列表→剩余部分，树→左/右子树，n→n-1）然后递归调用自己。
- **枚举所有选择 / 路径** — 必须尝试每一种选项或每一条路径。例如：「所有组合」、「从根到叶的所有路径」、「是否存在路径」。每次调用代表一种选择；对每个选项都递归一次。
- **分治（divide and combine）** — 把输入拆成两（或多）部分，分别递归求解，再合并答案。例如：归并排序（从中间拆开，排序两半，再合并）；二分查找（丢掉一半）。

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

## 例题 (LeetCode)

- **LC 70 爬楼梯 (Climb Stairs)** — 场景：每次 1 或 2 步，到 n 有几种方式。为何用：同样结构更小规模，f(n)=f(n-1)+f(n-2)。如何用：base 为 n<=1 返回 1；递归 + memo 或递推。
- **LC 200 岛屿数量 (Number of Islands)** — 场景：二维网格，相邻 1 为同一岛，求岛数。为何用：连通块 = 对每个未访问的 1 做一次 DFS 并标记。如何用：二重循环遇到 1 就 DFS 把整岛标成 0 或 visited，计数 +1。
- **LC 102 二叉树的层序遍历 (Binary Tree Level Order)** — 场景：按层输出节点值。为何用：层序 = BFS，第一次到达即该层。如何用：队列，每次处理当前层 len(q) 个节点，把子节点入队，当前层加入结果。

---

## 失败模式

- **没有基准，或递推不缩小** — 例如 f(n) 永远到不了 n=0。结果：无限递归、栈溢出。
- **改了共享状态但不还原** — 回溯里你先 `path.append(x)`，递归，返回后必须 `path.pop()`。忘了 pop，兄弟分支会看到不属于它们路径上的节点。
- **选错工具：BFS vs DFS** — 要「最短边数」（无权）？用 BFS。要「任意路径」或「所有路径」？用 DFS。用 DFS 求最短步数会错；用 BFS 枚举所有路径可以但通常比 DFS 笨。
