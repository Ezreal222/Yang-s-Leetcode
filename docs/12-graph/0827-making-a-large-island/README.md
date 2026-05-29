# 0827. Making A Large Island / 最大人工岛

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Graph, DFS, Grid, Flood Fill · 图, 深度优先, 网格, 洪水填充
    - **Link**: [LeetCode](https://leetcode.com/problems/making-a-large-island/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: `0/1` 网格. **最多翻一个 `0` 为 `1`**, 求翻完后最大岛屿的面积. 也可以不翻.

**中文**: 至多翻一个 0 为 1, 求最大岛屿格子数 (允许不翻).

## Key Insights

1. **🔑 两遍扫: 先标 ID 算面积, 再枚举 0 看能连多少 / Two passes: label + lookup**

    朴素做法: 对每个 0 假设翻它, 然后 DFS 算新岛屿面积 — O(m²n²), TLE.

    Yang 的两遍解法:

    1. **Pass 1**: 对每个未处理岛屿 DFS, 给所有格子标一个**唯一 ID** (≥ 2), 并记录 `areaOf[id] = area`.
    2. **Pass 2**: 对每个 `0` 格子, 看它**四邻居** 各属哪个 ID (用 set 去重), 答案 = `1 + Σ areaOf[id]`.

    复杂度 O(m × n) — 每格 Pass 1 + Pass 2 各 O(1).

    > **"标 ID + 查表" 是网格 + 多次查询的标准套路**. 比"每次重算" 快一个量级.

2. **🔑 ⚠ 必须 `unordered_set<int>` 去重邻居 ID / Dedup neighbor IDs**

    若一个 `0` 的四个邻居里有两个属于**同一个 U 形岛**, 不去重就会把那个岛的面积**加两次**, 答案偏大.

    Yang 用 `unordered_set<int> neighborIds` 收集, 自动去重. 然后遍历 set 求和.

    > 这是这题最容易翻车的点. 写朴素 sum 不去重直接 WA.

3. **🔑 复用 grid 当 ID 存储 / Reuse grid as ID map**

    不用单开 `id[i][j]` 数组 — DFS 时直接 `g[i][j] = id`. 之后第二遍读 `g[ni][nj] >= 2` 就是邻居的 ID. 省 O(m × n) 空间.

    > `0` (水), `1` (未访问陆地), `≥2` (已标 ID 陆地) — **三种语义复用一个矩阵**, 跟 [0130 三符号](../0130-surrounded-regions/README.md) 同精神.

4. **ID 从 2 起步 / IDs start at 2**

    `0` 和 `1` 已经被原始数据用了 → ID 必须 `≥ 2` 避免冲突. Yang `int id = 2;` 从 2 开始递增.

5. **`maxArea` 双重比较 / Track max across both passes**

    - Pass 1 算出每个原岛的面积, 直接更新 `maxArea` — 处理"不翻" 的情况 (最大原岛).
    - Pass 2 算出"翻 0 能合成" 的面积, 比较更新 — 处理"翻一个" 的情况.

    最终 `maxArea` 是两种最优的最大值. 边界自洽:

    - 全 1 网格: Pass 2 不触发, 答案 = 整张图大小.
    - 全 0 网格: Pass 1 不触发, Pass 2 算出 `sum = 1`, 答案 = 1.

6. **DFS 返回面积 (同 [0695](../0695-max-area-of-island/README.md)) / DFS returns area**

    `dfs(i, j, id)` 染色 + 累加面积返回. 跟 0695 同套路, 只是染色用 `id` 而非 0.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int m, n;
        vector<vector<int>> g;
        vector<pair<int, int>> dirs = {{0,1},{1,0},{-1,0},{0,-1}};
        unordered_map<int, int> areaOf;                            // id → 面积

        int largestIsland(vector<vector<int>>& grid) {
            g = grid;
            m = g.size(); n = g[0].size();
            int maxArea = 0;
            int id = 2;                                            // ≥ 2 防跟 0/1 冲突

            // Pass 1: 给每个岛打 ID, 记录面积
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (g[i][j] == 1) {
                        int area = dfs(i, j, id);
                        areaOf[id] = area;
                        maxArea = max(maxArea, area);              // "不翻" 情况
                        id++;
                    }
                }
            }
            // Pass 2: 枚举每个 0, 看翻它能连多少
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    if (g[i][j] != 0) continue;
                    unordered_set<int> neighborIds;                // ⚠ 去重邻居 ID
                    for (auto& [dx, dy] : dirs) {
                        int ni = i + dx, nj = j + dy;
                        if (ni < 0 || ni >= m || nj < 0 || nj >= n) continue;
                        if (g[ni][nj] >= 2) neighborIds.insert(g[ni][nj]);
                    }
                    int sum = 1;                                   // 翻的这格自己 +1
                    for (int nid : neighborIds) sum += areaOf[nid];
                    maxArea = max(maxArea, sum);
                }
            }
            return maxArea;
        }

    private:
        int dfs(int i, int j, int id) {
            if (i < 0 || i >= m || j < 0 || j >= n || g[i][j] != 1) return 0;
            g[i][j] = id;
            int area = 1;
            for (auto& [dx, dy] : dirs) area += dfs(i + dx, j + dy, id);
            return area;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def largestIsland(self, grid: list[list[int]]) -> int:
            m, n = len(grid), len(grid[0])
            dirs = [(0, 1), (1, 0), (-1, 0), (0, -1)]
            area_of: dict[int, int] = {}

            def dfs(i: int, j: int, idx: int) -> int:
                if i < 0 or i >= m or j < 0 or j >= n or grid[i][j] != 1:
                    return 0
                grid[i][j] = idx
                a = 1
                for dx, dy in dirs:
                    a += dfs(i + dx, j + dy, idx)
                return a

            max_area = 0
            idx = 2                                                # 从 2 开始
            for i in range(m):
                for j in range(n):
                    if grid[i][j] == 1:
                        a = dfs(i, j, idx)
                        area_of[idx] = a
                        max_area = max(max_area, a)
                        idx += 1

            for i in range(m):
                for j in range(n):
                    if grid[i][j] != 0:
                        continue
                    # set 去重邻居 ID — Pythonic 一行
                    ids = {grid[i + dx][j + dy]
                           for dx, dy in dirs
                           if 0 <= i + dx < m and 0 <= j + dy < n and grid[i + dx][j + dy] >= 2}
                    total = 1 + sum(area_of[k] for k in ids)
                    max_area = max(max_area, total)
            return max_area
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} grid
     * @return {number}
     */
    var largestIsland = function(grid) {
        const m = grid.length, n = grid[0].length;
        const dirs = [[0,1],[1,0],[-1,0],[0,-1]];
        const areaOf = new Map();

        const dfs = (i, j, id) => {
            if (i < 0 || i >= m || j < 0 || j >= n || grid[i][j] !== 1) return 0;
            grid[i][j] = id;
            let area = 1;
            for (const [dx, dy] of dirs) area += dfs(i + dx, j + dy, id);
            return area;
        };

        let maxArea = 0, id = 2;
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] === 1) {
                    const a = dfs(i, j, id);
                    areaOf.set(id, a);
                    if (a > maxArea) maxArea = a;
                    id++;
                }
            }
        }
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (grid[i][j] !== 0) continue;
                const ids = new Set();                             // 去重
                for (const [dx, dy] of dirs) {
                    const ni = i + dx, nj = j + dy;
                    if (ni < 0 || ni >= m || nj < 0 || nj >= n) continue;
                    if (grid[ni][nj] >= 2) ids.add(grid[ni][nj]);
                }
                let sum = 1;
                for (const k of ids) sum += areaOf.get(k);
                if (sum > maxArea) maxArea = sum;
            }
        }
        return maxArea;
    };
    ```

## Complexity

- **Time**: O(m × n) — 两遍扫.
- **Space**: O(m × n) — `areaOf` map + 递归栈.

## 相关题目

- [0200. Number of Islands](../0200-number-of-islands/README.md) — flood fill 母题
- [0695. Max Area of Island](../0695-max-area-of-island/README.md) — 单纯求最大岛 (本题"不翻" 子问题)
- [1020. Number of Enclaves](../1020-number-of-enclaves/README.md) — 反向 flood fill
- [0130. Surrounded Regions](../0130-surrounded-regions/README.md) — 反向 flood fill 翻转
- [0417. Pacific Atlantic Water Flow](../0417-pacific-atlantic-water-flow/README.md) — 双源 flood fill + 取交集
- 0305\. Number of Islands II (待补) — 动态加岛, Union-Find
- 1905\. Count Sub Islands (待补) — 子岛屿计数
