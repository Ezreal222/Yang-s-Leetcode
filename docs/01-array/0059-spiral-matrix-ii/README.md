# 0059. Spiral Matrix II / 螺旋矩阵 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Matrix, Simulation · 数组, 矩阵, 模拟
    - **Link**: [LeetCode](https://leetcode.com/problems/spiral-matrix-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **n×n matrix filled 1..n² in spiral** → **simulation**: either loop-by-layer with a strict **half-open `[start, n - offset)`** invariant, or single pass with a **direction array** that rotates on wall/visited.
>
> **中文**: **n×n 螺旋填 1..n²** → **模拟**: 按圈走, **左闭右开** 统一边界; 或方向数组单次扫, 遇墙/已访问就转 90°.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给 `n`, 生成 n×n 矩阵, 按**顺时针螺旋** 填 `1, 2, ..., n²`.

**中文**: 给 n, 螺旋填数字.

## Key Insights

1. **🔑 模拟题的核心 = 边界一致性 / Simulation = boundary consistency**

    螺旋题不难 — 但**最容易翻车的是边界**: 四条边谁包谁不包? 角点是否重复填? 关键是**全程用同一种区间约定**.

    Yang v1 的约定: **每条边都是 `[start, n - offset)` 左闭右开** — 角落留给下一条边. 4 个 for 循环结构对称, 看起来像一个咒语:

    ```
    j: [starty, n - offset)        // 上边: 不填右上角
    i: [startx, n - offset)        // 右边: 不填右下角
    j: [n - offset, starty)        // 下边: 不填左下角
    i: [n - offset, startx)        // 左边: 不填左上角
    ```

    > **任选一种 (左闭右开 / 都闭) 但全程贯彻**. 混用 = bug 工厂.

2. **🔑 两种风格 / Two flavors**

    | | **v1 按圈模拟** | **v2 方向数组** |
    |---|---|---|
    | 思路 | 一圈四边, 外圈到内圈 | 一格一格走, 遇墙转向 |
    | 循环数 | 4 (每边一个) | 1 (n² 次填数) |
    | 奇 n 中心 | **要特判** `mat[mid][mid]` | **不需要** |
    | 可推广性 | 仅螺旋类 | **任何"按规则走"** (DFS / BFS / 机器人路径都借这套) |
    | 边界依赖 | 区间约定要小心 | 越界 + 已填判定 |

    > v1 是**代码随想录**的经典写法 — 训练**控制循环不变量**. v2 更通用, 网格遍历的标配.

3. **🔑 v1: `offset` 每圈 + 1 — 控制"内退一格" / Shrink by one each loop**

    `offset` 初始 1, 每圈结束 +1. 第 k 圈范围 `[k - 1, n - k)` (左闭右开).

    - `loop = n / 2`: 几圈 — n=3 走 1 圈, n=4 走 2 圈, n=5 走 2 圈 + 中心一格.
    - `mid = n / 2`: 若 n 奇, 中心格坐标 (mid, mid).
    - `count` 累加 1..n², 最后 `if (n % 2) mat[mid][mid] = count;` 兜底.

    > Yang 这版代码**几乎是教科书** — 把 `for (j; ...; j++)` 这种"复用外部 j" 的写法用上, 在每条边结束时 j (或 i) 自然落到下条边的起点.

4. **🔑 v2: 方向数组 + "遇墙转" 模板 / Direction array + wall pivot**

    四方向顺时针: `{(0,1), (1,0), (0,-1), (-1,0)}`. 当前方向 `dir`. 每步:

    ```
    试探下一格 (nr, nc)
    若越界 or mat[nr][nc] != 0:
        dir = (dir + 1) % 4         // 顺时针转 90°
        重算 nr, nc
    走过去, 填数
    ```

    > **`(dir + 1) % 4`** 是网格题里**最通用的转向公式** — 四方向上下左右循环. 记下它.

    `mat[nr][nc] != 0` 等价"已访问" — **省了一个 visited 数组** 因为我们用"0 表示空" 当哨兵.

5. **🔑 为什么 v2 不用特判奇 n? / Why v2 needs no center handling**

    v2 一格一格填, **填到第 n² 个就停**. 中心格自然落在某一步, 不需要单独处理. v1 因为是"按层走" 的, 若 n 奇, 中心是"零层" — 没有"边" 可走, 只能手动补.

6. **复杂度 O(n²) 时间, O(n²) 空间 / Quadratic**

    两版相同 — 都填 n² 个格子, 结果矩阵占 n² 空间. 没法更优.

## Solution

=== "C++"

    **v1: 按圈模拟 + 左闭右开 (Yang 风格)**

    ```cpp
    class Solution {
    public:
        vector<vector<int>> generateMatrix(int n) {
            vector<vector<int>> res(n, vector<int>(n));
            int i, j;
            int startx = 0, starty = 0;
            int offset = 1, count = 1;
            int loop = n / 2, mid = n / 2;
            while (loop--) {
                i = startx;
                j = starty;
                for (j; j < n - offset; j++) res[i][j] = count++;           // 上: [starty, n-offset)
                for (i; i < n - offset; i++) res[i][j] = count++;           // 右
                for (j; j > starty;    j--) res[i][j] = count++;            // 下
                for (i; i > startx;    i--) res[i][j] = count++;            // 左
                startx++; starty++; offset++;                                // 收缩一圈
            }
            if (n % 2 == 1) res[mid][mid] = count;                          // n 奇: 补中心
            return res;
        }
    };
    ```

    **v2: 方向数组 + 遇墙转 (更通用)**

    ```cpp
    class Solution {
    public:
        vector<vector<int>> generateMatrix(int n) {
            vector<vector<int>> mat(n, vector<int>(n, 0));
            vector<pair<int,int>> dirs = {{0,1}, {1,0}, {0,-1}, {-1,0}};    // 右下左上
            int dir = 0, r = 0, c = 0;
            for (int num = 1; num <= n * n; num++) {
                mat[r][c] = num;
                int nr = r + dirs[dir].first;
                int nc = c + dirs[dir].second;
                if (nr < 0 || nr >= n || nc < 0 || nc >= n || mat[nr][nc] != 0) {
                    dir = (dir + 1) % 4;                                    // 遇墙/已填 → 转 90°
                    nr = r + dirs[dir].first;
                    nc = c + dirs[dir].second;
                }
                r = nr; c = nc;
            }
            return mat;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def generateMatrix(self, n: int) -> list[list[int]]:
            # v2 风格 — Python 写起来比 v1 干净
            mat = [[0] * n for _ in range(n)]
            # 方向数组: 顺时针 右↓左↑. 对应 C++ pair<int,int>
            dirs = [(0, 1), (1, 0), (0, -1), (-1, 0)]
            d = 0
            r = c = 0
            for num in range(1, n * n + 1):
                mat[r][c] = num
                nr, nc = r + dirs[d][0], c + dirs[d][1]
                # 越界或已填 → 转向. `not 0 <= nr < n` 比 `nr < 0 or nr >= n` 更 Pythonic
                if not (0 <= nr < n and 0 <= nc < n) or mat[nr][nc] != 0:
                    d = (d + 1) % 4
                    nr, nc = r + dirs[d][0], c + dirs[d][1]
                r, c = nr, nc
            return mat
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number[][]}
     */
    var generateMatrix = function(n) {
        // v2 风格. JS 用 Array.from 二维初始化 — 必须用工厂函数, 不能 fill([0]) (会共享同一引用)
        const mat = Array.from({length: n}, () => new Array(n).fill(0));
        const dirs = [[0, 1], [1, 0], [0, -1], [-1, 0]];
        let d = 0, r = 0, c = 0;
        for (let num = 1; num <= n * n; num++) {
            mat[r][c] = num;
            let nr = r + dirs[d][0];
            let nc = c + dirs[d][1];
            if (nr < 0 || nr >= n || nc < 0 || nc >= n || mat[nr][nc] !== 0) {
                d = (d + 1) % 4;        // 转向公式: (d + 1) % 4
                nr = r + dirs[d][0];
                nc = c + dirs[d][1];
            }
            r = nr;
            c = nc;
        }
        return mat;
    };
    ```

## Complexity

- **Time**: O(n²) — 填 n² 个格子.
- **Space**: O(n²) — 结果矩阵.

## 相关题目

- 0054\. Spiral Matrix (待补) — 给矩阵**读出** 螺旋顺序 (本题逆问题)
- 0885\. Spiral Matrix III (待补) — 任意起点 + 走出边界要继续, 方向数组思路
- 2326\. Spiral Matrix IV (待补) — 链表灌入矩阵 + 螺旋
- 0498\. Diagonal Traverse (待补) — 对角螺旋, 类似方向反转
- 0048\. Rotate Image (待补) — 同样是 n×n 矩阵几何变换
