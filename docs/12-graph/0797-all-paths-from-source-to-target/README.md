# 0797. All Paths From Source to Target / 所有可能的路径

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, Backtracking · 图, 深度优先, 回溯
    - **Link**: [LeetCode](https://leetcode.com/problems/all-paths-from-source-to-target/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给一个**有向无环图 (DAG)**, `graph[i]` 表示从节点 `i` 能到的节点列表. 起点 `0`, 终点 `n-1`. 返回**所有可能的从 0 到 n-1 的路径**.

**中文**: DAG 上找从 0 到 n-1 的所有路径. `graph[i]` 是 i 的邻接表.

## Key Insights

1. **🔑 DFS + 回溯模板 / DFS + backtracking**

    跟 [§08 回溯](../../08-backtracking/index.md) 系列同款"走 → 记 → 回退" 三步:

    ```
    push 当前节点
    if 到终点: 记录 path 副本
    else: 对每个邻居 dfs
    pop 当前节点 (回退)
    ```

    > 看到"所有路径 / 所有方案" → 立刻反应 DFS + 回溯. 这是图论题里最基础的范式.

2. **🔑 DAG 不需要 visited 数组 / No need for visited in DAG**

    DAG = **有向 + 无环**, 不会回到已访问的节点 → 不会陷入死循环. **省了 visited**.

    > 普通有向图或无向图就必须维护 visited, 否则死循环. **看清"图的性质" 决定要不要 visited**.

3. **状态 (传参): `(图, 当前节点, 终点)` / DFS signature**

    Yang 用 `dfs(graph, x, n)` 三参. `path` 用类成员维护 (push/pop 在 dfs 内). 也可以把 `path` 当参数传, 但成员变量更省栈空间.

4. **🔑 `path` 初始化为 `{0}` / Start with source node**

    起点 0 不在循环里加, 需要**初始就放进 path**. 然后 dfs 内每次进入 push 的是"下一个" 节点. Yang 用 `vector<int> path = {0}` 在成员声明里初始化.

    若 path 初始为空, 入口要单独 `path.push_back(0); dfs(...); path.pop_back();` 包一层.

5. **`res.push_back(path)` 是**值拷贝**, 不是引用 / Push a copy**

    C++ `vector` 默认值语义, `push_back(path)` 拷贝整个 path 进 res. 后续 path 继续变化不影响 res 里的副本. ✓

    若用 Python list 要 `res.append(path[:])` 或 `list(path)` 显式拷贝, 不能直接 `res.append(path)` (那是引用, 会被后续 pop 改).

6. **复杂度 O(2^n × n) / Complexity**

    DAG 最多 `2^n` 条路径 (完全 DAG), 每条路径长度 O(n) 拷贝. 总 O(2^n × n). LC 数据 n ≤ 15, 完全够.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> res;
        vector<int> path = {0};                                    // 起点 0 预填

        void dfs(const vector<vector<int>>& graph, int x, int n) {
            if (x == n) {
                res.push_back(path);                               // 值拷贝
                return;
            }
            for (int i : graph[x]) {
                path.push_back(i);
                dfs(graph, i, n);
                path.pop_back();                                   // 回退
            }
        }

        vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
            dfs(graph, 0, graph.size() - 1);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def allPathsSourceTarget(self, graph: list[list[int]]) -> list[list[int]]:
            n = len(graph) - 1
            res = []
            path = [0]                                             # 起点预填

            def dfs(x: int) -> None:
                if x == n:
                    res.append(path[:])                            # ⚠ 必须切片拷贝, 不能直接 append(path)
                    return
                for nxt in graph[x]:
                    path.append(nxt)
                    dfs(nxt)
                    path.pop()                                     # 回退

            dfs(0)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} graph
     * @return {number[][]}
     */
    var allPathsSourceTarget = function(graph) {
        const n = graph.length - 1;
        const res = [];
        const path = [0];

        const dfs = (x) => {
            if (x === n) {
                res.push([...path]);                               // spread 拷贝, 类似 Python path[:]
                return;
            }
            for (const nxt of graph[x]) {
                path.push(nxt);
                dfs(nxt);
                path.pop();
            }
        };

        dfs(0);
        return res;
    };
    ```

## Complexity

- **Time**: O(2^n × n).
- **Space**: O(n) — 递归栈 + path.

## 相关题目

- [§08 回溯算法](../../08-backtracking/index.md) — DFS + 回溯模板的大本营
- 0207\. Course Schedule (待补) — DAG + 拓扑排序判环
- 0210\. Course Schedule II (待补) — 拓扑排序输出顺序
- 0133\. Clone Graph (待补) — 图的 DFS / BFS 深拷贝
- 0200\. Number of Islands (待补) — 二维网格 DFS / BFS
- 0399\. Evaluate Division (待补) — 带权图 + DFS
