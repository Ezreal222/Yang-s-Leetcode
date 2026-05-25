# 0072. Edit Distance / 编辑距离

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: DP, Double Sequence, Levenshtein · 动态规划, 双序列, 编辑距离
    - **Link**: [LeetCode](https://leetcode.com/problems/edit-distance/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给两个字符串 `word1`, `word2`. 每次操作可以**插入 / 删除 / 替换** 一个字符 (任一字符串). 求把 `word1` 变成 `word2` 的**最少操作数** (Levenshtein 距离).

**中文**: 把 `word1` 变成 `word2`, 每次插/删/替一个字符, 求最少操作数.

## Key Insights

1. **🔑 双序列 DP 终极版: [0583 (只删)](../0583-delete-operation-for-two-strings/README.md) + 替换 + 插入 / Adds replace + insert to 0583**

    跟 [0583 删除](../0583-delete-operation-for-two-strings/README.md) 同模板, 但**允许三种操作**: 删 / 插 / 替. 转移多一个候选 (替换), 公式从 2 选 min 变成 3 选 min.

    | 题 | 允许操作 | 不匹配时转移 |
    |---|---|---|
    | [0583](../0583-delete-operation-for-two-strings/README.md) | 删 (任一边) | `min(dp[i-1][j], dp[i][j-1]) + 1` |
    | **0072 (本题)** | **删 + 插 + 替** | **`min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1`** |

    多出的 `dp[i-1][j-1] + 1` 就是**替换**.

2. **状态: `dp[i][j] = 把 word1[0..i) 变成 word2[0..j) 的最少操作数` / Convert prefix to prefix**

    "**前 i 个变前 j 个**". 答案是 `dp[n][m]` (全变成全).

3. **🔑 转移: 三种操作各自的 DP 来源 / Three operations, three sources**

    匹配时 (`word1[i-1] == word2[j-1]`): 末尾字符已经对上, **不用动**:

    $$dp[i][j] = dp[i-1][j-1]$$

    不匹配时, 三种操作的最少代价取 min, 再 +1 (本次操作):

    | 操作 | 含义 | 来源 |
    |---|---|---|
    | **删** word1[i-1] | 把 word1 末位干掉, 用 word1[0..i-1) 去变 word2[0..j) | `dp[i-1][j] + 1` |
    | **插** word2[j-1] 到 word1 | 把 word2 末位"凑出来", 用 word1[0..i) 去变 word2[0..j-1) | `dp[i][j-1] + 1` |
    | **替** word1[i-1] → word2[j-1] | 末位强行改成 word2 的, 用 word1[0..i-1) 去变 word2[0..j-1) | `dp[i-1][j-1] + 1` |

    > **"删 word1" vs "插到 word1"** 是镜像操作 — "删 word1[i-1]" 跟"在 word2 那侧插一个一样的" 等价. 但在转移表达上对应不同坐标方向: 删 → 减 i, 插 → 减 j. **不要混**.

4. **`dp[i-1][j-1] + 1` 既是"替换" 也是"删 word2 一个 + 在 word1 插一个" / Replace = delete + insert (1 op vs 2)**

    替换是"一步等价两步" 的捷径 — 同时让两边的末位前进, 只花 1 操作. 没有替换的话, 同样的效果要 2 步 (删一个 + 插一个).

5. **初始化 `dp[i][0] = i`, `dp[0][j] = j` / Base case: empty → all deletes/inserts**

    - `dp[i][0] = i`: word1 前 i 个变空串 = 全删, i 次操作.
    - `dp[0][j] = j`: 空串变 word2 前 j 个 = 全插, j 次操作.

    跟 [0583 v2](../0583-delete-operation-for-two-strings/README.md) 一样的边界. **漏写一行 = WA**.

6. **`min({a, b, c})` initializer_list 写法 / Multi-arg min**

    跟 [0152](../0152-maximum-product-subarray/README.md) 一样用花括号 `min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]})` 比嵌套 `min(min(a, b), c)` 干净. C++11+.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int minDistance(string word1, string word2) {
            int n = word1.size(), m = word2.size();
            vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
            for (int i = 0; i <= n; i++) dp[i][0] = i;             // 全删
            for (int j = 0; j <= m; j++) dp[0][j] = j;             // 全插
            for (int i = 1; i <= n; i++) {
                for (int j = 1; j <= m; j++) {
                    if (word1[i - 1] == word2[j - 1]) {
                        dp[i][j] = dp[i - 1][j - 1];               // 末尾已对上, 不动
                    } else {
                        // 三种操作取 min: 替 / 插 / 删
                        dp[i][j] = min({dp[i - 1][j - 1],          // 替换
                                        dp[i][j - 1],              // 插入
                                        dp[i - 1][j]}) + 1;        // 删除
                    }
                }
            }
            return dp[n][m];
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minDistance(self, word1: str, word2: str) -> int:
            n, m = len(word1), len(word2)
            dp = [[0] * (m + 1) for _ in range(n + 1)]
            for i in range(n + 1):
                dp[i][0] = i                                       # 全删
            for j in range(m + 1):
                dp[0][j] = j                                       # 全插
            for i in range(1, n + 1):
                for j in range(1, m + 1):
                    if word1[i - 1] == word2[j - 1]:
                        dp[i][j] = dp[i - 1][j - 1]
                    else:
                        # min(...) 接散参: 替 / 插 / 删
                        dp[i][j] = min(dp[i - 1][j - 1], dp[i][j - 1], dp[i - 1][j]) + 1
            return dp[n][m]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} word1
     * @param {string} word2
     * @return {number}
     */
    var minDistance = function(word1, word2) {
        const n = word1.length, m = word2.length;
        const dp = Array.from({length: n + 1}, () => new Array(m + 1).fill(0));
        for (let i = 0; i <= n; i++) dp[i][0] = i;
        for (let j = 0; j <= m; j++) dp[0][j] = j;
        for (let i = 1; i <= n; i++) {
            for (let j = 1; j <= m; j++) {
                if (word1[i - 1] === word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.min(dp[i - 1][j - 1], dp[i][j - 1], dp[i - 1][j]) + 1;
                }
            }
        }
        return dp[n][m];
    };
    ```

## Complexity

- **Time**: O(n × m).
- **Space**: O(n × m). (可压一维, 同 [1143](../1143-longest-common-subsequence/README.md) 用 `prevDiag` 暂存)

## 相关题目

- [1143. Longest Common Subsequence](../1143-longest-common-subsequence/README.md) — 双序列 DP 母题, 求长度
- [0583. Delete Operation for Two Strings](../0583-delete-operation-for-two-strings/README.md) — **只允许删**, 本题的简化版
- [0115. Distinct Subsequences](../0115-distinct-subsequences/README.md) — 双序列 DP 计数版
- 0392\. Is Subsequence (待补) — 判定 t 是不是 s 的子序列
- 0712\. Minimum ASCII Delete Sum for Two Strings (待补) — 0583 加权版
- 0044\. Wildcard Matching (待补) — 双序列 DP 终极, 带 `?` / `*` 通配符
- 0010\. Regular Expression Matching (待补) — 双序列 DP 加正则, 进阶
