# Ch5 — 图 (Graph)

图 = 树 + **多父/多子 + 环**。所以需要 **visited** 和「什么时候标记 visited」。树的 DFS/BFS 模板可以搬过来，多一步「避免重复访问」和「判环」。

---

## 识别信号 (Recognition Signals)

| 题目信号 | 想到的方法 |
|----------|------------|
| 「节点」「边」「邻接」 | 建图（邻接表/矩阵），DFS/BFS |
| 「连通分量」「岛屿」「能到吗」 | 对每个未访问节点 DFS/BFS，计数或收集 |
| 「最短路径」（边数） | BFS，入队时标记，第一次到达即最短 |
| 「顺序」「依赖」「先修」 | 拓扑排序（入度 0 入队，出队即顺序） |
| 「最短路径」（带权、非负） | Dijkstra（堆 + 松弛） |
| 「负权边」 | Bellman-Ford（松弛 V-1 轮） |
| 「最小代价连通所有节点」 | MST：Kruskal（边排序+Union-Find）或 Prim（堆） |
| 「能否二着色」「分组使组内无边」 | 二分图：BFS/DFS 染色 |
| 「动态连通」「合并集合」 | Union-Find |

---

## 什么时候想到图

- 题目里有**节点 (node)** 和**边 (edge)**、**依赖关系**、**连通分量**、**最短路径**（边数或带权）。
- 建图方式：**邻接表 (adjacency list)** — 每个节点一个列表存邻居；或 **邻接矩阵**。有向/无向、是否带权根据题意。

---

## 手写模板 (Whiteboard Templates)

**DFS（入栈时标记）：** `visited.add(start); stack=[start]` → `while stack: u=pop; for v in adj[u]: if v not in visited: visited.add(v); stack.append(v)`  
**BFS 最短路：** `q=deque([(start,0)]); visited={start}` → 出队 (u,d)，若 u==target  return d；邻居未访问则 visited.add、入队 (v,d+1)。  
**拓扑排序：** 算入度 → 入度 0 入队 → 出队 u，order.append(u)，邻居入度-1，若 0 则入队；若 len(order)<n 则有环。  
**Dijkstra：** dist[s]=0，堆 (0,s) → 取 (d,u)，若 d!=dist[u] continue；松弛邻居，若更小则更新 dist 并入堆。  
**二分图：** 对每个未访问 BFS，color[u]=0/1，邻居染 1-color[u]，若已染且同色 return False。

---

## 访问标记与环 (visited and cycles)

- **何时标记 visited** — **入队/入栈时标记**（推荐）：一放进队列/栈就标记，这样同一节点不会重复进。若在「弹出时」才标记，同一层可能把同一节点加进去多次。
- **有向图判环** — DFS + 三色：未访问 0、访问中 1、已结束 2。若 DFS 到「访问中」的节点则有环；回溯时标 2。
- **无向图判环** — DFS 时传「父节点」，邻居里除了父以外若还有已访问的则有环。

```python
# 有向图 DFS，入栈时标记
def dfs(start, adj):
    visited = set()
    stack = [start]
    visited.add(start)
    while stack:
        u = stack.pop()
        for v in adj[u]:
            if v not in visited:
                visited.add(v)
                stack.append(v)
```

---

## 连通分量 (connected components)

- **无向图** — 对每个未访问节点做一次 DFS/BFS，每次做完就是一个连通分量。数分量个数或把每个分量里的节点记下来。
- **有向图** — 强连通分量 (SCC) 用 Tarjan 或 Kosaraju，见下。

---

## BFS 最短路径（边权为 1）

无权图上「最短边数」= BFS 第一次到达该节点时的层数。用队列，存 (节点, 距离)，第一次扩展到 target 即答案。

```python
def bfs_shortest(adj, start, target):
    from collections import deque
    q = deque([(start, 0)])
    visited = {start}
    while q:
        u, d = q.popleft()
        if u == target: return d
        for v in adj[u]:
            if v not in visited:
                visited.add(v)
                q.append((v, d + 1))
    return -1
```

---

## 拓扑排序 (Topological Sort)

- **何时用** — 有向无环图 (DAG)、「顺序」「依赖」「先修」等。
- **做法** — (1) 算每个点入度；(2) 入度为 0 的入队；(3) 每次出队一个，把它连出去的边删掉（邻居入度 -1），若邻居入度变 0 则入队；(4) 出队顺序即拓扑序。若最后出队数量 < 节点数，说明有环。

```python
def topological_sort(adj, n):
    indeg = [0] * n
    for u in range(n):
        for v in adj[u]:
            indeg[v] += 1
    from collections import deque
    q = deque([i for i in range(n) if indeg[i] == 0])
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in adj[u]:
            indeg[v] -= 1
            if indeg[v] == 0:
                q.append(v)
    return order if len(order) == n else []  # 若不足 n 则有环
```

---

## 带权最短路径：Dijkstra

- **何时用** — 边权非负，求单源最短路径。
- **做法** — 用最小堆，存 (距离, 节点)。每次取当前距离最小的节点，若已处理过则跳过；否则标记已处理，并松弛其邻居（若 d[u] + w < d[v] 则更新 d[v] 并入堆）。

```python
import heapq
def dijkstra(adj, start, n):
    # adj: list of list of (neighbor, weight)
    dist = [float('inf')] * n
    dist[start] = 0
    heap = [(0, start)]
    while heap:
        d, u = heapq.heappop(heap)
        if d != dist[u]: continue
        for v, w in adj[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(heap, (dist[v], v))
    return dist
```

---

## 负权边：Bellman-Ford

- **何时用** — 有负权边；或需要判「负环」。
- **做法** — 松弛所有边，重复 V-1 轮；若第 V 轮还能松弛则有负环。

---

## 最小生成树 (MST, Minimum Spanning Tree)

- **何时用** — 无向带权图，「用最小代价连通所有节点」。
- **Kruskal** — 边按权排序，从小到大加边，用 Union-Find 保证不成环。加满 n-1 条边即 MST。
- **Prim** — 类似 Dijkstra，堆里存 (到已选集合的最短边权, 节点)，每次取最小边对应的节点加入集合，并更新其邻居到集合的距离。

---

## 二分图 (Bipartite)

- **何时用** — 「能否二着色」「分成两组使组内无边」。
- **做法** — BFS/DFS 染色，相邻节点不同色；若发现相邻同色则不是二分图。

```python
def is_bipartite(adj, n):
    color = {}  # 0 / 1
    for start in range(n):
        if start in color: continue
        from collections import deque
        q = deque([start])
        color[start] = 0
        while q:
            u = q.popleft()
            for v in adj[u]:
                if v not in color:
                    color[v] = 1 - color[u]
                    q.append(v)
                elif color[v] == color[u]:
                    return False
    return True
```

---

## 强连通分量 (SCC, Strongly Connected Components)

- **何时用** — 有向图、「强连通」「缩点」。
- **Tarjan / Kosaraju** — 模板题；面试较少手写，知道「有向图缩成 DAG」即可。

---

## Union-Find（并查集）

- **何时用** — 动态加边、多次问「两点是否连通」「合并两个集合」；或 Kruskal 里判环。
- **实现** — parent 数组 + 按秩合并或路径压缩；find 时路径压缩。

---

## 复杂度直觉 (Complexity Instincts)

| 操作 | 时间 | 空间 | 一眼看出 |
|------|------|------|----------|
| DFS/BFS 遍历 | O(V+E) | O(V) visited + 栈/队列 | 点边各访问一次 |
| 拓扑排序 | O(V+E) | O(V) 入度+队列 | 建度 O(E)，出队 V 次 |
| Dijkstra | O((V+E) log V) 二叉堆 | O(V) dist+堆 | 每边松弛一次，堆操作 log V |
| Bellman-Ford | O(VE) | O(V) | V-1 轮每轮扫 E 条边 |
| 二分图染色 | O(V+E) | O(V) color | 每点每边一次 |

---

## 例题 (LeetCode)

- **LC 200 岛屿数量 (Number of Islands)** — 场景：二维网格，相邻 1 为岛，求岛数。为何用：连通分量 = 对每个未访问的 1 做 DFS/BFS 并标记。如何用：二重循环遇 1 则 DFS 整岛标 0，计数 +1。
- **LC 207 课程表 (Course Schedule)** — 场景：选课有先修关系，问能否全选完。为何用：依赖关系 = 有向图，能否全选 = 是否有环 = 拓扑排序。如何用：建图、算入度、入度 0 入队，出队时邻居入度 -1，若出队数 < n 则有环。
- **LC 743 网络延迟时间 (Network Delay Time)** — 场景：有向加权图，从源点到所有点的最短路径，求最远的那个距离。为何用：带权最短路径 = Dijkstra。如何用：堆存 (距离, 节点)，每次取最小，松弛邻居并入堆。
- **LC 785 判断二分图 (Is Graph Bipartite?)** — 场景：无向图能否二着色使相邻不同色。为何用：二分图 = 无奇环 = BFS/DFS 染色。如何用：对每个未访问节点 BFS，相邻染另一色，若发现相邻同色则返回 False。

---

## 失败模式 (Failure Modes)

**原内容：**

- **visited 时机错** — 入队/入栈时就要标记，别等弹出再标，否则重复进队。
- **有向/无向搞混** — 建图时无向要加两条边。
- **Dijkstra 用于负权** — 负权要用 Bellman-Ford；Dijkstra 在负权下可能错。

**补充：**

- **常见误解** — 「BFS 和 DFS 结果一样」：遍历到的点一样，顺序和「第一次到达」含义不同；「拓扑排序唯一」：不唯一，任意合法顺序均可。
- **面试易挂点** — visited 在「弹出时」才标导致同一节点多次入队；有向图建反（先修 A→B 是 B 依赖 A，边应是 A→B 还是 B→A 要看清）；Dijkstra 堆里重复入队没判断 d!=dist[u]。
- **典型反例** — 无权最短路用 DFS：第一次到达不保证最短；应用 BFS。

---

## 跨章节联系 (Cross-Chapter Links)

- **Ch0 DFS/BFS** — 图 = 树上的 DFS/BFS + **visited** + 环；递归契约、状态归属同 Ch0。
- **Ch1 Tree** — 树 = 无环无多父的图；树不需要 visited。
- **Ch4 数据结构** — Dijkstra 用堆；拓扑用队列；Kruskal 用 Union-Find。
- **Ch6 DP** — 最短路有重叠子问题时可 DP（如 DAG 上单源最短路）；一般图用 BFS/Dijkstra。
