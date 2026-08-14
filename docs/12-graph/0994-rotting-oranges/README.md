# 0994. Rotting Oranges / 腐烂的橘子

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: BFS, Multi-source BFS, Grid · 广度优先, 多源 BFS, 网格
    - **Link**: [LeetCode](https://leetcode.com/problems/rotting-oranges/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min minutes till all fresh oranges rot (spreads to 4-neighbors per minute)** → **multi-source BFS**: enqueue **all initial rotten oranges together**, layer-by-layer expand (each layer = 1 minute), track fresh count for early-return + final answer.
>
> **中文**: **所有新鲜橘子烂完需最少分钟数 (每分钟 4 邻居传染)** → **多源 BFS**: 所有初始烂橘子**同时**入队, 层序扩展 (每层 = 1 分钟), fresh 计数用于早退 + 判无解.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 网格 `grid`: 0 = 空, 1 = 新鲜橘子, 2 = 烂橘子. **每分钟**每个烂橘子的**4 邻居** 新鲜橘子也变烂. 求**所有新鲜烂完** 的最少分钟数; 无法全烂返 -1.

**中文**: 网格里的烂橘子每分钟传染 4 邻居, 求全烂最少分钟数.

## Key Insights

1. **🔑 灵魂: 多源 BFS — 所有起点同时入队 / Multi-source BFS: enqueue all sources together**

    单源 BFS 是从**一个起点** 扩. 本题**多个起点** (所有初始烂橘子), 但它们**同一时刻** 开始传染.

    → **一次性把所有初始烂橘子入队**, 然后**层序 BFS** — 每层就是"这一分钟新烂的" — 时间自然从 0 开始每层 +1.

    ```
    Init: 所有烂橘子入队 (时刻 0)
    Layer 1: 传染其 4 邻居 (时刻 1)
    Layer 2: 再传染 (时刻 2)
    ...
    ```

    > **看到"多个起点 + 同时扩散"** → 反应多源 BFS. 跟单源 BFS 只差**"入队全部起点"** 这一步.

2. **🔑 层序遍历: `sz = q.size()` + minute++ / Layer BFS: snapshot size + increment minute**

    Yang 的关键结构:

    ```cpp
    while (!q.empty() && fresh > 0) {
        int sz = q.size();                       // 快照本层大小
        for (int i = 0; i < sz; i++) {
            auto [r, c] = q.front(); q.pop();
            for (auto& d : dirs) {
                // 传染 4 邻居, 新烂的入队
            }
        }
        minutes++;                                // 本层做完 = 1 分钟
    }
    ```

    - **`sz = q.size()`** 记本层节点数 (避免边循环边加入影响).
    - **for sz 次**处理这一层.
    - **`minutes++`** 层完成 = 时间前进 1.

    > **BFS 层序 = 每层一个"时间步 / 距离步"**. 跟 [0127 Word Ladder](../0127-word-ladder/README.md) 的层序求最短同款.

3. **🔑 早退优化: `fresh > 0` / Early exit when no fresh left**

    Yang 的**双条件循环**: `!q.empty() && fresh > 0`. 若 fresh 归 0 → 提前退 (即使 q 还有节点也不必再传染, 因为没鲜橘可染).

    - **`if (fresh == 0) return 0;`** 一开始就判 — 无新鲜直接返 0.

    > 早退让 BFS 不多算几层无效. 常数优化, 但**思维严谨** 的体现.

4. **🔑 `fresh` 计数一箭三雕 / `fresh` counter: three uses**

    | 用途 | 语义 |
    |---|---|
    | **BFS 早退** | fresh > 0 时才继续 |
    | **最终判定** | 循环退出后 fresh > 0 → 有烂不到的孤立橘 → 返 -1 |
    | **性能** | 免最后再扫一遍网格 |

    > 单一变量承载多种意图, **信息复用**.

5. **🔑 原地标记 `grid[nr][nc] = 2` / In-place mark, no visited array**

    传染时直接把新鲜橘 (1) 改成烂橘 (2), **免建 visited 数组**. 因为**烂了不会再变** — 幂等安全.

    > **"用原网格值当 flag"** 是网格 BFS/DFS 的常见空间优化 (跟 [0130 Surrounded Regions](../0130-surrounded-regions/README.md) 的 `'#'` 标记同源).

6. **🔑 方向数组 `dirs[4][2]` / Direction array**

    ```cpp
    int dirs[4][2] = {{0,1},{1,0},{0,-1},{-1,0}};
    for (auto& d : dirs) {
        int nr = r + d[0], nc = c + d[1];
        ...
    }
    ```

    避免写 4 次 `nr = r-1, r+1, c-1, c+1`. **网格题必备**.

7. **🔑 无法全烂的情形 / Unreachable oranges → -1**

    循环退出后 `fresh > 0` — 说明**存在孤立的新鲜橘子** (被 0 隔开无法传染). 返 -1.

    ```cpp
    return fresh == 0 ? minutes : -1;
    ```

8. **🔑 跟其他 BFS 题的关系 / vs other BFS**

    | 题 | 类型 | 特点 |
    |---|---|---|
    | [0127 Word Ladder](../0127-word-ladder/README.md) | 单源 BFS | 找最短路径长度 |
    | **0994 (本题)** | **多源 BFS** | **时间层 = 距离** |
    | 0286 Walls and Gates (待补) | 多源 BFS | 求每格到最近门 |
    | 0542 01 Matrix (待补) | 多源 BFS | 求每格到最近 0 |

    > **多源 BFS 一族**都是"平行扩散 + 层数 = 距离/时间". 学一得四.

9. **🔑 复杂度 O(m × n) 时间/空间 / Linear in grid size**

    - Time: 每格入队/出队各 1 次.
    - Space: queue 最坏装满整个 grid.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int orangesRotting(vector<vector<int>>& grid) {
            int m = grid.size(), n = grid[0].size();
            queue<pair<int, int>> q;
            int fresh = 0;

            // 初始化: 所有烂橘子入队 (多源), 数新鲜
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (grid[i][j] == 2) q.push({i, j});
                    else if (grid[i][j] == 1) ++fresh;
                }
            }
            if (fresh == 0) return 0;                              // 无新鲜直接返 0

            int minutes = 0;
            int dirs[4][2] = {{0,1},{1,0},{0,-1},{-1,0}};

            while (!q.empty() && fresh > 0) {
                int sz = q.size();                                  // 层序快照
                for (int i = 0; i < sz; i++) {
                    auto [r, c] = q.front(); q.pop();
                    for (auto& d : dirs) {
                        int nr = r + d[0], nc = c + d[1];
                        if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                        if (grid[nr][nc] != 1) continue;             // 非新鲜, 跳
                        grid[nr][nc] = 2;                            // 原地标记
                        fresh--;
                        q.push({nr, nc});
                    }
                }
                minutes++;                                          // 本层完 = 1 分钟
            }

            return fresh == 0 ? minutes : -1;                       // 有剩余新鲜 → -1
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def orangesRotting(self, grid: list[list[int]]) -> int:
            m, n = len(grid), len(grid[0])
            q = deque()
            fresh = 0
            for i in range(m):
                for j in range(n):
                    if grid[i][j] == 2: q.append((i, j))
                    elif grid[i][j] == 1: fresh += 1
            if fresh == 0: return 0

            minutes = 0
            dirs = [(0, 1), (1, 0), (0, -1), (-1, 0)]
            while q and fresh > 0:
                # 层序: 记本层大小, 处理 sz 次
                for _ in range(len(q)):
                    r, c = q.popleft()
                    for dr, dc in dirs:
                        nr, nc = r + dr, c + dc
                        if 0 <= nr < m and 0 <= nc < n and grid[nr][nc] == 1:
                            grid[nr][nc] = 2
                            fresh -= 1
                            q.append((nr, nc))
                minutes += 1
            return minutes if fresh == 0 else -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} grid
     * @return {number}
     */
    var orangesRotting = function(grid) {
        const m = grid.length, n = grid[0].length;
        const q = [];
        let fresh = 0;
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] === 2) q.push([i, j]);
                else if (grid[i][j] === 1) fresh++;
            }
        }
        if (fresh === 0) return 0;
        let minutes = 0;
        const dirs = [[0, 1], [1, 0], [0, -1], [-1, 0]];
        // JS 数组当 queue, shift 是 O(N). 大数据用真 deque; 本题 grid ≤ 100 无所谓
        while (q.length && fresh > 0) {
            const sz = q.length;
            for (let i = 0; i < sz; i++) {
                const [r, c] = q.shift();
                for (const [dr, dc] of dirs) {
                    const nr = r + dr, nc = c + dc;
                    if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                    if (grid[nr][nc] !== 1) continue;
                    grid[nr][nc] = 2;
                    fresh--;
                    q.push([nr, nc]);
                }
            }
            minutes++;
        }
        return fresh === 0 ? minutes : -1;
    };
    ```

## Complexity

- **Time**: O(m × n) — 每格入队/出队各 1 次.
- **Space**: O(m × n) — queue 最坏.

## 相关题目

- [0127. Word Ladder](../0127-word-ladder/README.md) — 单源 BFS 层序求最短
- [0200. Number of Islands](../0200-number-of-islands/README.md) — 网格 DFS/BFS
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — 网格 DFS
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 网格 DFS + 原地标记
- [0417. Pacific Atlantic Water Flow](../0417-pacific-atlantic-water-flow/README.md) — 双源 DFS
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — 网格 DFS
- 0286\. Walls and Gates (待补) — 多源 BFS 同款
- 0542\. 01 Matrix (待补) — 多源 BFS 求最近 0
- 1091\. Shortest Path in Binary Matrix (待补) — 8-邻 BFS
- 0815\. Bus Routes (待补) — BFS 变体, 站 + 线路
