# 0685. Redundant Connection II / 冗余连接 II

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Graph, Union-Find, Tree, Directed · 图, 并查集, 树, 有向图
    - **Link**: [LeetCode](https://leetcode.com/problems/redundant-connection-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给一张 n 节点的**有向图**, 它本来是棵**有根树** (每个节点恰有一个父节点, 根没有父), **多加了一条有向边**. 找出可以删除以恢复有根树的边, 多解返回**输入最后**.

**中文**: 有根树 + 1 条多余有向边, 找能删的那条 (恢复有根树), 多解返回最后.

## Key Insights

1. **🔑 跟 [0684 无向](../0684-redundant-connection/README.md) 的本质差别: 有向图 + 多余边可能两种"作恶" 方式 / Two distinct violations in directed case**

    无向图中, 多余边只能造**环**. 有向 + 树结构有两种异常:

    - **情况 A**: 某节点出现**两个父节点** (入度 2) → 树性质破坏, 但可能没环.
    - **情况 B**: 多余边造成**环** (跟 0684 同).
    - **情况 C**: 两种同时 — 某节点入度 2, 同时该节点参与一个环.

    > **必须分情况** 讨论. 这是 0685 比 0684 难的核心.

2. **🔑 三步走 / Three-step plan**

    1. **扫一遍找入度 2 的节点**: 记录指向它的两条边 `cand1` (先出现) 和 `cand2` (后出现).
    2. **若找到 (情况 A 或 C)**:
        - 试删 `cand2`, 剩下能成树吗? 能 → 删 `cand2`.
        - 不能 → 删 `cand1`.
    3. **若没找到 (情况 B, 纯环)**: 跟 0684 一字不差, UF 找第一条成环边返回.

    > **题目"多解返回最后" 隐含**: 同 (cand1, cand2) 时优先返回 cand2 (输入更晚). 上面"先试 cand2" 就是这个规则的实现.

3. **🔑 `isTreeWithoutEdge` 用 UF 判定 / Tree check via UF**

    "**剩下 n-1 条边能否构成有根树**" 等价于"无环" (因为去掉的是入度 2 节点的一条父边, 入度问题自然消解). 用 UF 跑剩余 n-1 条边: 若任意一次 `unite` 返回 false (成环) → 不是树, 返 false.

    > **UF 判无环** 跟 0684 同思想, 包装成一个辅助函数即可.

4. **入度 2 检测: 用 `inDegreeFrom[v] = i` 存指向 v 的边索引 / Track in-degree via index map**

    Yang 用 `vector<int> inDegreeFrom(n + 1, -1)` 记录"指向 v 的第一条边在 edges 中的索引". 遇到第二条指向 v 的边时, 就发现了入度 2 的节点, 记下 (cand1, cand2) 并 break.

    > **第一次遇到记下索引, 第二次撞到就出手** — 流式处理入度的标准写法.

5. **`unite` 用 `swap` 让 rx 永远是较高的根 / Cleaner union by rank via swap**

    Yang 的 UF 写法 (跟 [0684](../0684-redundant-connection/README.md) 略变):

    ```cpp
    if (rank_[rx] < rank_[ry]) swap(rx, ry);   // 保证 rx 高
    parent[ry] = rx;                            // 矮的挂高的
    if (rank_[rx] == rank_[ry]) rank_[rx]++;   // 平手时高度 +1
    ```

    比三分支 if/else 短一行. 等价, 选哪种都行.

6. **复杂度 O(N × α(N)) ≈ O(N) / Near-linear**

    最多两次 UF 扫描 (`isTreeWithoutEdge` 调用一次 + 最坏 UF 主跑), 每次 O(N × α).

## Solution

=== "C++"
    ```cpp
    class UnionFind {
        vector<int> parent, rank_;
    public:
        UnionFind(int n) : parent(n), rank_(n) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }
        bool unite(int x, int y) {
            int rx = find(x), ry = find(y);
            if (rx == ry) return false;                                         // 成环信号
            if (rank_[rx] < rank_[ry]) swap(rx, ry);                            // 保 rx 更高
            parent[ry] = rx;
            if (rank_[rx] == rank_[ry]) rank_[rx]++;
            return true;
        }
    };

    class Solution {
    public:
        vector<int> findRedundantDirectedConnection(vector<vector<int>>& edges) {
            int n = edges.size();
            vector<int> inDegreeFrom(n + 1, -1);                                // v → 指向它的边索引
            int cand1 = -1, cand2 = -1;

            // 第一步: 扫描找入度 2 的节点
            for (int i = 0; i < n; i++) {
                int v = edges[i][1];
                if (inDegreeFrom[v] == -1) {
                    inDegreeFrom[v] = i;
                } else {
                    cand1 = inDegreeFrom[v];                                    // 先出现的
                    cand2 = i;                                                  // 后出现的
                    break;
                }
            }

            // 第二步: 有入度 2 的节点 → 试删 cand2, 否则 cand1
            if (cand2 != -1) {
                return isTreeWithoutEdge(edges, cand2, n) ? edges[cand2] : edges[cand1];
            }

            // 第三步: 没入度 2 (纯环情况), 跟 0684 一字不差
            UnionFind uf(n + 1);
            for (auto& e : edges) {
                if (!uf.unite(e[0], e[1])) return e;
            }
            return {};
        }

    private:
        bool isTreeWithoutEdge(vector<vector<int>>& edges, int skipIdx, int n) {
            UnionFind uf(n + 1);
            for (int i = 0; i < n; i++) {
                if (i == skipIdx) continue;
                if (!uf.unite(edges[i][0], edges[i][1])) return false;          // 成环 → 不是树
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class UnionFind:
        def __init__(self, n: int):
            self.parent = list(range(n))
            self.rank = [0] * n

        def find(self, x: int) -> int:
            if self.parent[x] != x:
                self.parent[x] = self.find(self.parent[x])
            return self.parent[x]

        def unite(self, x: int, y: int) -> bool:
            rx, ry = self.find(x), self.find(y)
            if rx == ry:
                return False
            if self.rank[rx] < self.rank[ry]:
                rx, ry = ry, rx
            self.parent[ry] = rx
            if self.rank[rx] == self.rank[ry]:
                self.rank[rx] += 1
            return True


    class Solution:
        def findRedundantDirectedConnection(self, edges: list[list[int]]) -> list[int]:
            n = len(edges)
            in_from = [-1] * (n + 1)
            cand1 = cand2 = -1

            # 第一步: 找入度 2 的节点
            for i, (_, v) in enumerate(edges):
                if in_from[v] == -1:
                    in_from[v] = i
                else:
                    cand1, cand2 = in_from[v], i
                    break

            def is_tree_without(skip: int) -> bool:
                uf = UnionFind(n + 1)
                for i, (u, v) in enumerate(edges):
                    if i == skip:
                        continue
                    if not uf.unite(u, v):
                        return False
                return True

            # 第二步
            if cand2 != -1:
                return edges[cand2] if is_tree_without(cand2) else edges[cand1]

            # 第三步: 纯环
            uf = UnionFind(n + 1)
            for u, v in edges:
                if not uf.unite(u, v):
                    return [u, v]
            return []
    ```

=== "JavaScript"
    ```javascript
    class UnionFind {
        constructor(n) {
            this.parent = Array.from({length: n}, (_, i) => i);
            this.rank = new Array(n).fill(0);
        }
        find(x) {
            if (this.parent[x] !== x) this.parent[x] = this.find(this.parent[x]);
            return this.parent[x];
        }
        unite(x, y) {
            let rx = this.find(x), ry = this.find(y);
            if (rx === ry) return false;
            if (this.rank[rx] < this.rank[ry]) [rx, ry] = [ry, rx];
            this.parent[ry] = rx;
            if (this.rank[rx] === this.rank[ry]) this.rank[rx]++;
            return true;
        }
    }

    /**
     * @param {number[][]} edges
     * @return {number[]}
     */
    var findRedundantDirectedConnection = function(edges) {
        const n = edges.length;
        const inFrom = new Array(n + 1).fill(-1);
        let cand1 = -1, cand2 = -1;
        for (let i = 0; i < n; i++) {
            const v = edges[i][1];
            if (inFrom[v] === -1) inFrom[v] = i;
            else { cand1 = inFrom[v]; cand2 = i; break; }
        }
        const isTreeWithout = (skip) => {
            const uf = new UnionFind(n + 1);
            for (let i = 0; i < n; i++) {
                if (i === skip) continue;
                if (!uf.unite(edges[i][0], edges[i][1])) return false;
            }
            return true;
        };
        if (cand2 !== -1) {
            return isTreeWithout(cand2) ? edges[cand2] : edges[cand1];
        }
        const uf = new UnionFind(n + 1);
        for (const e of edges) {
            if (!uf.unite(e[0], e[1])) return e;
        }
        return [];
    };
    ```

## Complexity

- **Time**: O(N × α(N)) ≈ O(N) — UF 最坏跑两次.
- **Space**: O(N).

## 相关题目

- [0684. Redundant Connection](../0684-redundant-connection/README.md) — 无向图版, 本题简化
- [1971. Find if Path Exists in Graph](../1971-find-if-path-exists-in-graph/README.md) — UF 判连通母题
- 0207\. Course Schedule (待补) — 有向图 + 拓扑排序判环
- 0210\. Course Schedule II (待补) — 输出拓扑序
- 0261\. Graph Valid Tree (待补) — 判定无向图是否为树
- 1192\. Critical Connections in a Network (待补) — Tarjan 找桥
