# 0210. Course Schedule II / 课程表 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Topological Sort, BFS, Kahn's Algorithm · 图, 拓扑排序, 广度优先, Kahn 算法
    - **Link**: [LeetCode](https://leetcode.com/problems/course-schedule-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 `numCourses` 门课, `prerequisites[i] = [a, b]` 表示**修 a 前必须先修 b**. 求一个合法的**修课顺序**, 不存在则返 `[]`.

**中文**: 课程依赖图, 求拓扑序; 有环返空数组.

## Key Insights

1. **🔑 问题本质: 有向图拓扑排序 / Topological sort on DAG**

    "修 a 前必须先修 b" → 有向边 `b → a`. 求**所有节点的线性排序**, 使得每条边的源点在目标点**之前**. 这就是**拓扑排序**.

    > 看到"按依赖顺序输出 / 任务调度 / 编译依赖" → 立刻反应拓扑排序.

2. **🔑 两大算法: BFS (Kahn) vs DFS / Topological sort: Kahn vs DFS-postorder**

    | | Kahn's (BFS, Yang 用) | DFS-postorder |
    |---|---|---|
    | 思路 | 反复挑入度 0 的点出队 | DFS 后序遍历 + 反转 |
    | 判环 | 入队总数 < n → 有环 | DFS 中遇到正在访问的节点 → 有环 |
    | 直观 | **更直观, 适合本题** | 适合递归思路熟手 |
    | 输出 | 直接是拓扑序 | 反转后是拓扑序 |

    > **Kahn 更适合"求顺序" 类题** — BFS 出队顺序就是答案. DFS 版的"后序 + 反转" 多一步.

3. **🔑 Kahn 算法三件套 / Three data structures**

    1. **`graph[u]`**: u 的后继列表 (邻接表).
    2. **`inDegree[v]`**: v 的入度计数.
    3. **`queue<int> q`**: 当前所有入度 0 的节点.

    **流程**:

    1. 初始化: 扫边构建 `graph` + `inDegree`. 所有 `inDegree == 0` 的入队.
    2. BFS: 每出队一个 u, 加入 `order`, 把 u 的所有后继 `v` 的入度减 1; 若 `inDegree[v]` 减到 0, 入队.
    3. 终止: 队空时, 若 `order.size() == n` → 拓扑序; 否则有环, 返空.

4. **🔑 判环: 看最终 `order.size()` / Cycle detection via final count**

    若有环, 环上的节点入度永远不会减到 0 (因为环外节点先出队, 但环上节点的入度始终被环内边贡献) → 不会进队 → 不在 `order` 里. 所以 `order.size() < n` 等价"有环".

    > 这是 Kahn 算法最优雅的副产品: **判环 + 求序一体**, 不用单独判环.

5. **`prerequisites[i] = [a, b]` 的边方向 / Edge direction matters**

    LC 约定 `[a, b]` 意为"修 a 前要先修 b" → 边 `b → a`. 所以:

    - `graph[b].push_back(a)` (b 是 a 的前置)
    - `inDegree[a]++` (a 多了一条入边)

    Yang 写 `graph[p[1]].push_back(p[0])` 和 `inDegree[p[0]]++`, 正确. **方向写反就 WA**.

6. **复杂度 O(V + E) / Linear**

    每节点入队/出队各一次, 每条边遍历一次.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
            vector<vector<int>> graph(numCourses);
            vector<int> inDegree(numCourses, 0);
            // 建图: [a, b] = 边 b → a
            for (auto& p : prerequisites) {
                graph[p[1]].push_back(p[0]);
                inDegree[p[0]]++;
            }

            // 初始入度 0 的入队
            queue<int> q;
            for (int i = 0; i < numCourses; i++) {
                if (inDegree[i] == 0) q.push(i);
            }

            vector<int> order;
            while (!q.empty()) {
                int u = q.front(); q.pop();
                order.push_back(u);
                for (int v : graph[u]) {
                    if (--inDegree[v] == 0) q.push(v);             // 入度减到 0 才入队
                }
            }

            // 若所有节点都进了 order, 无环; 否则有环, 返空
            return order.size() == numCourses ? order : vector<int>{};
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def findOrder(self, numCourses: int, prerequisites: list[list[int]]) -> list[int]:
            graph: list[list[int]] = [[] for _ in range(numCourses)]
            in_degree = [0] * numCourses
            for a, b in prerequisites:                              # 修 a 前要 b → 边 b → a
                graph[b].append(a)
                in_degree[a] += 1

            # 初始入度 0 的入队
            q = deque(i for i in range(numCourses) if in_degree[i] == 0)
            order = []
            while q:
                u = q.popleft()
                order.append(u)
                for v in graph[u]:
                    in_degree[v] -= 1
                    if in_degree[v] == 0:
                        q.append(v)
            return order if len(order) == numCourses else []
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} numCourses
     * @param {number[][]} prerequisites
     * @return {number[]}
     */
    var findOrder = function(numCourses, prerequisites) {
        const graph = Array.from({length: numCourses}, () => []);
        const inDegree = new Array(numCourses).fill(0);
        for (const [a, b] of prerequisites) {
            graph[b].push(a);
            inDegree[a]++;
        }

        const q = [];
        for (let i = 0; i < numCourses; i++) {
            if (inDegree[i] === 0) q.push(i);
        }

        const order = [];
        while (q.length) {
            const u = q.shift();
            order.push(u);
            for (const v of graph[u]) {
                if (--inDegree[v] === 0) q.push(v);
            }
        }
        return order.length === numCourses ? order : [];
    };
    ```

## Complexity

- **Time**: O(V + E) — 每节点 / 每边 O(1) 次操作.
- **Space**: O(V + E) — 邻接表 + 队列 + 入度数组.

## 相关题目

- 0207\. Course Schedule (待补) — 只判能否完成 (返 bool), 跟本题相同算法, 省"输出 order" 步
- [0127. Word Ladder](../0127-word-ladder/README.md) — 同款 BFS, 但目标是最短路而非拓扑序
- 0269\. Alien Dictionary (待补) — 字母拓扑排序
- 0310\. Minimum Height Trees (待补) — 拓扑剥皮找树重心
- 0444\. Sequence Reconstruction (待补) — 唯一拓扑序判定
- 1136\. Parallel Courses (待补) — 拓扑排序 + 层数计数
- 2050\. Parallel Courses III (待补) — 拓扑排序 + 最长路径 DP
