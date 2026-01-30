# Ch2 — 回溯 (Backtracking)

控制「指数级」枚举。回溯 = DFS + **显式状态管理**：选一个分支、递归、再**撤销**选择，试下一个分支。树那一章是「隐式状态」（树结构决定分支）；回溯里你要自己维护「当前选了哪些」，并在返回时还原。

---

## 什么时候想到回溯 (backtracking)

题目要求**枚举**下面一类东西，且每一步有**多种选择**、选完要**试另一种**时，用回溯：

- 「所有组合 (all combinations)」「所有排列 (all permutations)」
- 「所有满足条件的子集」「所有路径」「所有划分」
- 「是否存在一组选择使得……」也可以先想回溯（再考虑能否剪枝或转 DP）

**信号：** 选/不选、选哪一个、顺序；选完要「撤销」再试下一个。

---

## 决策树 (decision tree)

把回溯过程想成一棵**决策树**：每个节点 = 当前状态，每条边 = 一次选择，叶子 = 得到一个解或死路。

- **分支** — 当前可做的选择（选哪个数、放哪个位置、选/不选当前元素）
- **剪枝 (prune)** — 若当前分支已经不可能得到合法解，直接 return，不再递归
- **叶子** — 要么得到一个解（记入 result），要么不合法（忽略）

---

## 状态回滚 (state rollback)

**模板：** 做选择 → 递归 → 撤销选择。用同一个列表（如 `path`、`used`）时，递归返回后必须还原，否则兄弟分支会带着你的选择。

```python
# 例：全排列 (permutations)
def permute(nums):
    result = []
    used = [False] * len(nums)
    def backtrack(path):
        if len(path) == len(nums):
            result.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]: continue
            used[i] = True
            path.append(nums[i])
            backtrack(path)
            path.pop()
            used[i] = False
    backtrack([])
    return result
```

---

## 剪枝 (pruning)

- **合法性剪枝** — 选了这个数就违反约束（如重复、超过和），直接 continue 或 return
- **重复剪枝** — 若数组有重复且不允许重复组合/排列，先排序，再在循环里跳过「与上一个相同且上一个没选」的情况
- **界剪枝** — 剩余元素全选也不够、或已经超过目标，提前 return

```python
# 例：组合和 (combination sum)，每个数可用一次，无重复组合
def combination_sum2(candidates, target):
    candidates.sort()
    result = []
    def backtrack(start, path, remain):
        if remain == 0:
            result.append(path[:])
            return
        if remain < 0:
            return
        for i in range(start, len(candidates)):
            if i > start and candidates[i] == candidates[i-1]:
                continue  # 重复剪枝
            path.append(candidates[i])
            backtrack(i + 1, path, remain - candidates[i])
            path.pop()
    backtrack(0, [], target)
    return result
```

---

## 例题 (LeetCode)

- **LC 46 全排列 (Permutations)** — 场景：无重复数组，求所有排列。为何用：每一步选一个未选的数，选完要试其他选择，需撤销。如何用：used 数组 + path，for 里选未选的、标记、递归、撤销。
- **LC 39 组合总和 (Combination Sum)** — 场景：同一数可重复选，和等于 target 的所有组合。为何用：枚举选哪个数、选几个，选完递归并撤销。如何用：回溯时传 start 避免重复组合；可重复则递归仍从 i 开始。
- **LC 78 子集 (Subsets)** — 场景：求所有子集。为何用：每个元素选或不选，枚举所有 2^n 种。如何用：回溯，每层先记当前 path 为一种子集，再 for 从 start 到末尾选一个加入、递归、撤销。

---

## 失败模式 (failure modes)

- **忘记撤销** — `path.append(x)` 后递归，返回后必须 `path.pop()`；`used[i] = True` 后必须 `used[i] = False`。否则兄弟分支会带着你的选择。
- **重复结果** — 求「组合」时用 `start` 避免回头选；有重复元素时排序 + 「相同数只选第一个未被选的」避免重复组合。
- **剪枝写错** — 剪枝条件要和题意一致；别把合法分支剪掉。
