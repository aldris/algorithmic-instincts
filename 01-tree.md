# Ch1 — 树 (Tree)

面试 ROI 最高的一章。Meta/Google 最爱考树的题。树上的递归是最「干净」的形态，也是 DP / Graph / Backtracking 的低噪声训练场。

---

## 什么时候想到树 DFS/BFS

题目给了一棵**二叉树 (binary tree)** 或**多叉树**，且问的是下面一类问题时，就用树的遍历：

- **路径 (path)** — 从根到叶的路径、路径和、路径上是否满足某条件
- **最近公共祖先 (LCA, lowest common ancestor)** — 两个节点在树里的最近公共祖先
- **验证 / 性质** — 是否 BST、是否平衡、是否对称
- **序列化 / 反序列化 (serialize / deserialize)** — 树 ↔ 字符串

树没有「回头路」，所以不需要 `visited`，比图简单。

---

## 子树返回什么 / 信息如何向上合并

递归时，每个节点代表一棵**子树 (subtree)**。要明确两件事：**子节点返回什么**，**当前节点怎么合并**。

- **返回类型** — 一个值（如深度、和、最大值）、布尔（是否满足）、路径列表、计数等
- **合并方式** — 取 max/min、求和、and/or、选一边（如只要左或右）

**模板：** `if not node: return 基准值; left = f(node.left); right = f(node.right); return merge(left, right, node.val)`

```python
# 例：树高 = 1 + max(左高, 右高)
def max_depth(node):
    if not node: return 0
    return 1 + max(max_depth(node.left), max_depth(node.right))

# 例：是否平衡 = 左右都平衡 且 |左高 - 右高| <= 1
def is_balanced(node):
    def height(node):
        if not node: return 0
        L, R = height(node.left), height(node.right)
        if L == -1 or R == -1 or abs(L - R) > 1: return -1
        return 1 + max(L, R)
    return height(node) != -1
```

---

## 路径 (path) / 合法性 (validity) / 全局结果 (global result)

- **路径** — 用「往下传」的 `path`（或 `path_sum`）；进入子节点前 append，离开后 pop（回溯）。到叶节点时根据题目决定是否把 path 记入结果。
- **合法性** — 约束往下传（如「当前路径上的最大值」、「是否已经选了某类节点」）；不满足则剪枝。
- **全局结果** — 若需要收集所有路径或全局最值，用 `result` 列表或 `global_max` 等引用，在递归中更新。

```python
# 例：根到叶的所有路径（路径值为列表）
def all_paths(root):
    result = []
    def dfs(node, path):
        if not node: return
        path.append(node.val)
        if not node.left and not node.right:
            result.append(path[:])
        dfs(node.left, path)
        dfs(node.right, path)
        path.pop()
    dfs(root, [])
    return result
```

---

## 层序 (level order) — 用 BFS

若问「按层输出」「每一层的节点」「每一层的最大值」等，用 **BFS + 按层循环**：每次处理一层的节点数。

```python
def level_order(root):
    if not root: return []
    from collections import deque
    q = deque([root])
    result = []
    while q:
        level_size = len(q)
        level = []
        for _ in range(level_size):
            node = q.popleft()
            level.append(node.val)
            if node.left: q.append(node.left)
            if node.right: q.append(node.right)
        result.append(level)
    return result
```

---

## 例题 (LeetCode)

- **LC 104 二叉树最大深度 (Maximum Depth of Binary Tree)** — 场景：求树高。为何用：子树返回高度，当前节点合并为 1+max(left, right)。如何用：base 为空返回 0；递归左右取 max 加 1。
- **LC 113 路径总和 II (Path Sum II)** — 场景：根到叶的路径上节点和等于 target，求所有这样的路径。为何用：路径 = 往下传 path 和当前和，到叶时若和等于 target 则 path 加入结果。如何用：DFS 带 path 和 cur_sum，进子前 append，出子后 pop。
- **LC 102 二叉树的层序遍历 (Binary Tree Level Order)** — 场景：按层输出。为何用：层序用 BFS，每轮处理当前层节点数。如何用：队列，while 里 for _ in range(len(q)) 取一层，子节点入队。

---

## 失败模式 (failure modes)

- **忘记 pop** — 路径用共享的 `path` 时，递归返回后必须 `path.pop()`，否则兄弟分支会带上不该有的节点。
- **空节点没处理** — `if not node: return 基准值` 要写对；基准值要和合并逻辑一致（如深度为 0，和为 0）。
- **要层序却用 DFS** — 问「每一层」时用 BFS 按层取；DFS 一次走到底，不自然分层。
