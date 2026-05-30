# 1971. Find if Path Exists in Graph / 寻找图中是否存在路径

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Graph, Union-Find, BFS, DFS · 图, 并查集, 广度优先, 深度优先
    - **Link**: [LeetCode](https://leetcode.com/problems/find-if-path-exists-in-graph/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 无向图 `n` 个节点, `edges` 是边列表. 判断从 `source` 到 `destination` 是否**存在路径**.

**中文**: 无向图判断两点是否连通.

## Key Insights

1. **🔑 "判定连通" → 并查集 (Union-Find) 完美场景 / Connectivity query = Union-Find**

    并查集就是为"判定连通块" 设计的. 流程:

    1. **建并查集**: 每条边 `(u, v)` → `unite(u, v)`.
    2. **查询**: `find(source) == find(destination)`?

    比 BFS / DFS 更优雅 — **多次查询** 时一次建表后每次查询近 O(1).

    > 看到"是否连通 / 是否一组" → 立刻反应 UF. 看到"找路径 / 最短路径" → BFS/DFS. **问什么决定用什么**.

2. **🔑 Union-Find 三大 API / Three core operations**

    | API | 含义 | 复杂度 |
    |---|---|---|
    | **`find(x)`** | 返回 x 所在集合的"代表元" (根) | 近 O(1) 摊销 (带路径压缩) |
    | **`unite(x, y)`** | 把 x, y 所在集合合并 | 近 O(1) |
    | **`connected(x, y)`** | 判断 x, y 是否同集合 = `find(x) == find(y)` | 近 O(1) |

3. **🔑 两大优化: 路径压缩 + 按秩合并 / Path compression + union by rank**

    朴素 UF 每次 `find` O(树高). 加两个优化几乎 O(1):

    - **路径压缩 (Path Compression)**: `find` 递归时把路径上**所有节点直接挂到根**. Yang 写法:

        ```cpp
        if (parent[x] != x) parent[x] = find(parent[x]);   // 递归 + 顺手把 x 挂到根
        ```

    - **按秩合并 (Union by Rank)**: 合并时**让"矮树挂到高树"** 上 — 保持总树高低. Yang 用 `rank_` 数组追踪高度:

        ```cpp
        if (rank_[rx] < rank_[ry]) parent[rx] = ry;
        else if (rank_[rx] > rank_[ry]) parent[ry] = rx;
        else { parent[ry] = rx; rank_[rx]++; }              // 平手时高度 +1
        ```

    > **两个一起用** 给摊销复杂度 O(α(n)) ≈ O(1), α 是阿克曼函数反函数, 实际数据几乎是常数.

4. **`rank_` 不是真高度, 是"近似高度上界" / Rank is approximate**

    由于路径压缩会动态减小树高, `rank_` 不是当前真实高度. 但作为"高度上界" 来决定合并方向就足够 — 保证总树高不会比 `log n` 差.

    > 严格证明用 amortized 分析, 这里只要知道**`rank_` 是合并方向的参考量** 就行.

5. **初始化: `parent[i] = i` (每个节点自成一集) / Each node is its own root initially**

    Yang 的构造函数 `for (int i = 0; i < n; i++) parent[i] = i;`. `rank_` 默认 0. **基础不变量: 一开始每个节点独立**.

6. **`rank` 是 STL 关键字, 加 `_` 避坑 / Avoid name clash with std::rank**

    C++ `<type_traits>` 里有 `std::rank`. 直接用变量名 `rank` 在某些编译器会有 ambiguous lookup. Yang 加下划线 `rank_` 是好习惯.

7. **复杂度 O(α(n) × (E + Q)) ≈ O(E + Q) / Near-linear**

    - 建 UF: 遍历 E 条边, 每次 `unite` 近 O(1) → O(E).
    - 查询 Q 次, 每次 `find` 近 O(1) → O(Q).

8. **替代解法: BFS / DFS — 单次查询也行 / Alternatives**

    本题只查询一次, BFS / DFS 也是 O(V + E), 不一定非 UF. 但 UF 模板**重复查询有奇效** (如 0547 / 0305 多查询版本).

## Solution

=== "C++"
    ```cpp
    class UnionFind {
        vector<int> parent, rank_;
    public:
        UnionFind(int n) : parent(n), rank_(n, 0) {
            for (int i = 0; i < n; i++) parent[i] = i;             // 自成一集
        }
        int find(int x) {
            // 路径压缩: 递归找根, 顺手把 x 挂到根
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }
        void unite(int x, int y) {
            int rx = find(x), ry = find(y);
            if (rx == ry) return;
            // 按秩合并: 矮树挂到高树
            if (rank_[rx] < rank_[ry]) parent[rx] = ry;
            else if (rank_[rx] > rank_[ry]) parent[ry] = rx;
            else { parent[ry] = rx; rank_[rx]++; }
        }
        bool connected(int x, int y) { return find(x) == find(y); }
    };

    class Solution {
    public:
        bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
            UnionFind uf(n);
            for (auto& e : edges) uf.unite(e[0], e[1]);
            return uf.connected(source, destination);
        }
    };
    ```

=== "Python"
    ```python
    class UnionFind:
        def __init__(self, n: int):
            self.parent = list(range(n))                           # 等价 [0, 1, ..., n-1]
            self.rank = [0] * n

        def find(self, x: int) -> int:
            # 路径压缩 — 迭代版避免深递归栈溢出
            root = x
            while self.parent[root] != root:
                root = self.parent[root]
            # 二次扫描把路径上每点挂到根
            while self.parent[x] != root:
                self.parent[x], x = root, self.parent[x]
            return root

        def unite(self, x: int, y: int) -> None:
            rx, ry = self.find(x), self.find(y)
            if rx == ry:
                return
            # 按秩合并
            if self.rank[rx] < self.rank[ry]:
                self.parent[rx] = ry
            elif self.rank[rx] > self.rank[ry]:
                self.parent[ry] = rx
            else:
                self.parent[ry] = rx
                self.rank[rx] += 1

        def connected(self, x: int, y: int) -> bool:
            return self.find(x) == self.find(y)


    class Solution:
        def validPath(self, n: int, edges: list[list[int]], source: int, destination: int) -> bool:
            uf = UnionFind(n)
            for u, v in edges:
                uf.unite(u, v)
            return uf.connected(source, destination)
    ```

=== "JavaScript"
    ```javascript
    class UnionFind {
        constructor(n) {
            this.parent = Array.from({length: n}, (_, i) => i);    // [0, 1, ..., n-1]
            this.rank = new Array(n).fill(0);
        }
        find(x) {
            if (this.parent[x] !== x) this.parent[x] = this.find(this.parent[x]);
            return this.parent[x];
        }
        unite(x, y) {
            const rx = this.find(x), ry = this.find(y);
            if (rx === ry) return;
            if (this.rank[rx] < this.rank[ry]) this.parent[rx] = ry;
            else if (this.rank[rx] > this.rank[ry]) this.parent[ry] = rx;
            else { this.parent[ry] = rx; this.rank[rx]++; }
        }
        connected(x, y) { return this.find(x) === this.find(y); }
    }

    /**
     * @param {number} n
     * @param {number[][]} edges
     * @param {number} source
     * @param {number} destination
     * @return {boolean}
     */
    var validPath = function(n, edges, source, destination) {
        const uf = new UnionFind(n);
        for (const [u, v] of edges) uf.unite(u, v);
        return uf.connected(source, destination);
    };
    ```

## Complexity

- **Time**: O((V + E) × α(n)) ≈ O(V + E) — 近线性.
- **Space**: O(V) — parent + rank.

## 相关题目

- [§12 Graph 主页](../index.md) — 图论所有题
- 0547\. Number of Provinces (待补) — UF 数连通块个数
- 0684\. Redundant Connection (待补) — UF 找成环边
- 0721\. Accounts Merge (待补) — UF + 哈希做账户合并
- 0305\. Number of Islands II (待补) — 动态加岛, UF 经典
- 1319\. Number of Operations to Make Network Connected (待补) — UF + 计数
- 0990\. Satisfiability of Equality Equations (待补) — UF + 字符方程组
