# 0684. Redundant Connection / 冗余连接

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, Union-Find, Tree · 图, 并查集, 树
    - **Link**: [LeetCode](https://leetcode.com/problems/redundant-connection/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给一张 n 节点的图, 它本来是棵树, **多加了一条边** 形成一个环. 边按输入顺序排列, 找出可以**删除以恢复树** 的边. 若有多解, 返回**输入中最后出现** 的那条.

**中文**: 树 + 1 条多余边, 找能删的那条 (使图重新成树). 多解返回输入最后的.

## Key Insights

1. **🔑 "树 + 多一条边 = 一个环" → UF 找成环边 / Tree + 1 extra = cycle, UF detects**

    遍历 edges, 用 UF 边连边. **第一条让"已经连通的两点又被连接" 的边**就是成环的那条 — 就是答案.

    > 这是 UF 在"检测成环" 类题的经典应用. 跟"连通块计数" / "动态加边" 同属 UF 三大典型场景.

2. **🔑 "第一条成环边" = "输入中最后出现的环上边" / First cycle-edge = last in input among cycle edges**

    乍看矛盾, 其实自洽: 遍历到某条边 (u, v) 时 `find(u) == find(v)`, 说明 u, v 之前已经被**别的环上边** 连通过 — 那些边都在它**之前**. 所以这条"成环边" 自然是环里**输入最晚** 的那条. 完美匹配题目"多解返回最后" 的要求.

    > 一个看似巧合的良好行为, 实际是算法结构的必然.

3. **🔑 `unite` 返回 bool — 信号化"成环" / Return bool from unite**

    Yang 给 `unite` 加返回值: `false` 表示"未合并 (因为已连通)" = 检测到环. 这样主函数一行就能拿到答案:

    ```cpp
    if (!uf.unite(e[0], e[1])) return e;
    ```

    > 跟 [1971](../1971-find-if-path-exists-in-graph/README.md) 的 `void unite` 版本对比, 这是 UF 模板的**小升级 — 让 API 自带信号位**.

4. **`iota(parent.begin(), parent.end(), 0)` 初始化 / Idiomatic init**

    Yang 用 `<numeric>` 的 `iota` 一行填 `[0, 1, 2, ..., n-1]`, 比 `for (i) parent[i] = i;` 干净.

    > 同款 idiomatic 写法: Python `list(range(n))`, JS `Array.from({length: n}, (_, i) => i)`.

5. **大小 `n + 1`: 节点编号从 1 起步 / Size n+1 for 1-indexed nodes**

    LC 节点编号 1..n. 索引 0 留空但要预留, 否则 `parent[n]` 越界. `UnionFind uf(n + 1)`.

    > **看清节点编号是从 0 还是 1 起**, 大小相应 `+1`. 漏掉直接 RE.

6. **跟 [1971](../1971-find-if-path-exists-in-graph/README.md) 的对比 / vs 1971**

    | | [1971](../1971-find-if-path-exists-in-graph/README.md) | **0684 (本题)** |
    |---|---|---|
    | 问 | 两点连通? | 找成环的边 |
    | `unite` 返回 | void | **bool (成环信号)** |
    | 输出 | 单 bool | 一条边 |
    | UF 用法 | 全建完再 `connected` | 边建边 check |

7. **复杂度 O(N × α(N)) ≈ O(N) / Near-linear**

    N 条边, 每次 UF 操作摊销近 O(1).

## Solution

=== "C++"
    ```cpp
    class UnionFind {
        vector<int> parent, rank_;
    public:
        UnionFind(int n) : parent(n), rank_(n) {
            iota(parent.begin(), parent.end(), 0);                  // 一行填 0..n-1
        }
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);        // 路径压缩
            return parent[x];
        }
        // 返回 true 表示成功合并, false 表示已连通 (成环)
        bool unite(int x, int y) {
            int rx = find(x), ry = find(y);
            if (rx == ry) return false;                              // ⚠ 成环信号
            if (rank_[rx] < rank_[ry]) parent[rx] = ry;
            else if (rank_[rx] > rank_[ry]) parent[ry] = rx;
            else { parent[ry] = rx; rank_[rx]++; }
            return true;
        }
    };

    class Solution {
    public:
        vector<int> findRedundantConnection(vector<vector<int>>& edges) {
            int n = edges.size();
            UnionFind uf(n + 1);                                     // 节点 1..n, 预留 0
            for (auto& e : edges) {
                if (!uf.unite(e[0], e[1])) return e;                 // 第一条成环 = 答案
            }
            return {};                                                // 不可能到这里
        }
    };
    ```

=== "Python"
    ```python
    class UnionFind:
        def __init__(self, n: int):
            self.parent = list(range(n))                            # 等价 iota
            self.rank = [0] * n

        def find(self, x: int) -> int:
            if self.parent[x] != x:
                self.parent[x] = self.find(self.parent[x])
            return self.parent[x]

        def unite(self, x: int, y: int) -> bool:
            rx, ry = self.find(x), self.find(y)
            if rx == ry:
                return False                                        # 成环
            if self.rank[rx] < self.rank[ry]:
                self.parent[rx] = ry
            elif self.rank[rx] > self.rank[ry]:
                self.parent[ry] = rx
            else:
                self.parent[ry] = rx
                self.rank[rx] += 1
            return True


    class Solution:
        def findRedundantConnection(self, edges: list[list[int]]) -> list[int]:
            n = len(edges)
            uf = UnionFind(n + 1)                                   # 1-indexed
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
            const rx = this.find(x), ry = this.find(y);
            if (rx === ry) return false;                            // 成环
            if (this.rank[rx] < this.rank[ry]) this.parent[rx] = ry;
            else if (this.rank[rx] > this.rank[ry]) this.parent[ry] = rx;
            else { this.parent[ry] = rx; this.rank[rx]++; }
            return true;
        }
    }

    /**
     * @param {number[][]} edges
     * @return {number[]}
     */
    var findRedundantConnection = function(edges) {
        const uf = new UnionFind(edges.length + 1);
        for (const [u, v] of edges) {
            if (!uf.unite(u, v)) return [u, v];
        }
        return [];
    };
    ```

## Complexity

- **Time**: O(N × α(N)) ≈ O(N).
- **Space**: O(N) — UF parent + rank.

## 相关题目

- [1971. Find if Path Exists in Graph](../1971-find-if-path-exists-in-graph/README.md) — UF 判连通母题
- 0547\. Number of Provinces (待补) — UF 数连通块个数
- 0721\. Accounts Merge (待补) — UF + 哈希做账户合并
- [0685. Redundant Connection II](../0685-redundant-connection-ii/README.md) — 有向图版 (Hard), 三种情况, 入度 + UF
- 1319\. Number of Operations to Make Network Connected (待补) — UF + 数额外边/缺失连接
- 1631\. Path With Minimum Effort (待补) — UF 按边权排序 (Kruskal 思想)
