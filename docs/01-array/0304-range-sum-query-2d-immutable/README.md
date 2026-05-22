# 0304. Range Sum Query 2D - Immutable / 二维区域和检索 - 矩阵不可变

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Matrix, Prefix Sum, Design · 数组, 矩阵, 前缀和, 设计
    - **Link**: [LeetCode](https://leetcode.com/problems/range-sum-query-2d-immutable/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given an `m × n` matrix, support `sumRegion(r1, c1, r2, c2)` returning the sum of the sub-matrix from `(r1, c1)` to `(r2, c2)` (inclusive). Multiple queries; matrix is fixed.

**中文**: `m × n` 矩阵, 支持 `sumRegion(r1, c1, r2, c2)` — 求左上 `(r1, c1)` 到右下 `(r2, c2)` 的子矩阵和 (闭区间). 多次查询, 矩阵不变.

## Key Insights

1. **二维前缀和定义 (带哨兵) / 2D prefix sum with sentinel**

    `prefix[i][j] = 矩阵 (0,0) 到 (i-1, j-1) 的矩形和` (前 i 行前 j 列, 不含第 i 行 / 第 j 列).

    数组尺寸 `(m+1) × (n+1)`, **第 0 行和第 0 列全 0** (哨兵). 让所有边界 case 都自动处理.

2. **递推公式: 容斥原理 / Recurrence via inclusion-exclusion**

    ```
    prefix[i+1][j+1] = prefix[i][j+1]      // 上面那块
                    + prefix[i+1][j]      // 左边那块
                    - prefix[i][j]        // 重叠的 (左上角) 多算一次, 减回
                    + matrix[i][j];       // 加上当前格子
    ```

    画图就明白: "上块 + 左块" 把"左上角小方块" 算了**两次**, 减一次回平.

3. **查询公式: 容斥原理 (反向) / Query via inclusion-exclusion**

    求子矩阵 `(r1, c1)` 到 `(r2, c2)`:

    ```
    sum = prefix[r2+1][c2+1]   // 包含 (r2, c2) 的左上整块
        - prefix[r1][c2+1]     // 减掉 r1 之上的部分
        - prefix[r2+1][c1]     // 减掉 c1 之左的部分
        + prefix[r1][c1];      // 左上角小块被减了两次, 加回
    ```

    画图: 想要的矩形 = 大矩形 - 上条 - 左条 + 左上小块.

4. **哨兵让所有 +1 偏移统一 / Sentinel removes corner cases**

    没哨兵时, `r1 == 0` 或 `c1 == 0` 要特判 (减不到那一行/列). 加 1 行 1 列 0 后, **任何查询都用同一公式**, 不用 if.

    > 二维前缀和的"+1 偏移" 是新手最常错的地方. 记住: **prefix 比 matrix 大一圈, prefix[i+1][j+1] 对应 matrix[i][j]**.

5. **复杂度: O(mn) 预处理, O(1) 查询 / Build O(mn), query O(1)**

    朴素查询 O((r2-r1) × (c2-c1)). 前缀和把"重复算" 提前 → 任何子矩阵和都是**4 次查表 + 3 次加减**, 与子矩阵大小**无关**.

## Solution

=== "C++"
    ```cpp
    class NumMatrix {
        vector<vector<int>> prefix;                                // (m+1) × (n+1) 带哨兵
    public:
        NumMatrix(vector<vector<int>>& matrix) {
            int m = matrix.size(), n = matrix[0].size();
            prefix.assign(m + 1, vector<int>(n + 1, 0));
            for (int i = 0; i < m; i++) {
                for (int j = 0; j < n; j++) {
                    prefix[i + 1][j + 1] = prefix[i][j + 1]        // 上块
                                         + prefix[i + 1][j]        // 左块
                                         - prefix[i][j]            // 重叠减回
                                         + matrix[i][j];           // 当前格
                }
            }
        }
        int sumRegion(int r1, int c1, int r2, int c2) {
            return prefix[r2 + 1][c2 + 1]
                 - prefix[r1][c2 + 1]                              // 上条
                 - prefix[r2 + 1][c1]                              // 左条
                 + prefix[r1][c1];                                 // 左上角加回
        }
    };
    ```

=== "Python"
    ```python
    class NumMatrix:
        def __init__(self, matrix: list[list[int]]):
            m, n = len(matrix), len(matrix[0])
            # 列表推导建 (m+1) × (n+1) 零矩阵: 注意不能用 [[0]*(n+1)]*(m+1), 那是浅拷贝, 所有行指向同一个 list
            self.prefix = [[0] * (n + 1) for _ in range(m + 1)]
            for i in range(m):
                for j in range(n):
                    self.prefix[i + 1][j + 1] = (
                        self.prefix[i][j + 1]
                        + self.prefix[i + 1][j]
                        - self.prefix[i][j]
                        + matrix[i][j]
                    )

        def sumRegion(self, r1: int, c1: int, r2: int, c2: int) -> int:
            p = self.prefix
            return p[r2 + 1][c2 + 1] - p[r1][c2 + 1] - p[r2 + 1][c1] + p[r1][c1]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} matrix
     */
    var NumMatrix = function(matrix) {
        const m = matrix.length, n = matrix[0].length;
        // Array.from + 工厂函数: 每行独立 new Array, 避免 fill(arr) 的浅引用陷阱 (类似 Python 的 [...]*n 坑)
        this.prefix = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                this.prefix[i + 1][j + 1] = this.prefix[i][j + 1]
                                          + this.prefix[i + 1][j]
                                          - this.prefix[i][j]
                                          + matrix[i][j];
            }
        }
    };

    /**
     * @param {number} row1
     * @param {number} col1
     * @param {number} row2
     * @param {number} col2
     * @return {number}
     */
    NumMatrix.prototype.sumRegion = function(r1, c1, r2, c2) {
        const p = this.prefix;
        return p[r2 + 1][c2 + 1] - p[r1][c2 + 1] - p[r2 + 1][c1] + p[r1][c1];
    };
    ```

## Complexity

- **Time**: 构造 O(m·n); 每次 `sumRegion` O(1).
- **Space**: O(m·n) — prefix 矩阵.

## 相关题目

- [0303. Range Sum Query - Immutable](../0303-range-sum-query-immutable/README.md) — 一维前缀和, 本题基础
- [1094. Car Pooling](../1094-car-pooling/README.md) — 一维差分 (前缀和的对偶)
- 0308\. Range Sum Query 2D - Mutable (待补) — 二维可变 → 二维树状数组
- 0363\. Max Sum of Rectangle No Larger Than K (待补) — 二维前缀和 + 子矩阵和
- 1314\. Matrix Block Sum (待补) — 二维前缀和应用
- 1314\. Sum of Submatrices Smaller Than K (待补) — 二维前缀和 + 计数
- 0221\. Maximal Square (待补) — 二维 DP 相关, 同样有"左/上/左上" 三方关系
