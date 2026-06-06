# 0727. Minimum Window Subsequence / 最小窗口子序列

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Two Pointers, String, Sliding Window · 双指针, 字符串, 滑窗
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-window-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给字符串 `s1`, `s2`. 找 `s1` 中**最短的连续子串**, 使 `s2` 是它的**子序列** (即 s2 的字符按序出现在子串里, 可以不连续). 若有多解返**最左** 的, 不存在返 `""`.

**中文**: 找 s1 中最短的连续子串, 让 s2 作为它的子序列存在.

## Key Insights

1. **🔑 两指针 + 回溯 — 不是 DP / Two pointers + backtrack, not DP**

    本题有 DP 解 `dp[i][j] = 以 s1[i] 结尾匹配 s2[0..j] 的最短长度`, O(m × n) 时空. 但**双指针 + 反向缩窗** 更直观, 时空 O(m × n) 最坏但常数小. Yang 用后者.

    > **DP 解 vs 双指针解** 各有所长: DP 通解但费空间, 双指针在"匹配/缩窗" 类题里更紧凑.

2. **🔑 三段式: 正扫匹配 → 反扫缩窗 → 跳到 left+1 重启 / Three-phase template**

    1. **正扫**: i 在 s1, j 在 s2. 匹配则 j++, 否则只 i++. 当 j == n (s2 全匹配) → 记 `right = i`.
    2. **反扫缩窗**: 从当前位置往左走, 反向匹配 s2 (j 从 n-1 倒着退), 找到使 s2 作子序列的**最左 left**. 此时 `[left, right]` 是当前可行的最紧窗.
    3. **更新答案 + 重启**: 比较 `right - left + 1` 跟 bestLen; **跳到 `i = left + 1`, 重置 `j = 0`** 继续找下一个候选.

    > **为什么是 `left + 1` 而不是 `right + 1`?** 因为 `[left+1, ?]` 可能存在另一个**更短** 的窗口 (起点更晚, 但可能用更少字符就匹完 s2). 跳到 right + 1 会漏掉重叠窗口.

3. **🔑 反扫缩窗的核心 / Backward shrink details**

    Yang 的反扫逻辑:

    ```cpp
    j--;                                 // 从 n-1 起步 (上面 j 加到了 n)
    while (j >= 0) {
        if (s1[i] == s2[j]) j--;         // 匹配则 j 退一步
        i--;                             // 不论是否匹配, i 都退
    }
    i++; j++;                            // 退过头了, 修正一格
    int left = i;
    ```

    反扫结束: `j < 0` 表示 s2 的所有字符**逆序** 都在 `[i+1, right]` 里找到 → 最左可行起点是 i+1.

    > **正扫 + 反扫"两边逼近"** 是这题的精髓. 一遍正扫确定右边界, 一遍反扫确定左边界, 配合得到当前最紧窗.

4. **复杂度 O(m × n) 最坏 / Worst-case quadratic**

    每个 i 可能被正/反扫各一次, 加上重启从 left + 1 起步. 退化情况是双 O(m × n). 大多数实际数据下接近线性.

    > LC 数据下 m, n ≤ 几万, 这版能过. 极端 case 需要 DP O(m × n) 版.

5. **`bestStart == -1` 兜底返空 / Default empty if no match**

    若 s1 始终没找到 s2 作子序列 (正扫 j 永远到不了 n), `bestStart` 保持初值 -1, 最终返 "".

6. **跟 0076 Minimum Window Substring (待补) 的对比 / vs 0076**

    | | 0076 (待补) | **0727 (本题)** |
    |---|---|---|
    | 找 | s 中**包含 t 全部字符** (含重复) 的最短子串 | s1 中**含 t 为子序列** 的最短子串 |
    | 解法 | 滑动窗口 + 哈希计数 | 两指针 + 反扫缩窗 / DP |
    | 字符顺序 | t 顺序无关 | **s2 顺序必须保持** |

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string minWindow(string s1, string s2) {
            int m = s1.size(), n = s2.size();
            int i = 0, j = 0;
            int bestStart = -1, bestLen = INT_MAX;

            while (i < m) {
                if (s1[i] == s2[j]) {
                    j++;
                    if (j == n) {                                          // s2 全匹配, 进入反扫
                        int right = i;
                        j--;                                                // 退到 n-1
                        while (j >= 0) {
                            if (s1[i] == s2[j]) j--;                        // 反向匹配则 j--
                            i--;
                        }
                        i++; j++;                                           // 退过头, 修正
                        int left = i;

                        if (right - left + 1 < bestLen) {                   // 更新答案
                            bestLen = right - left + 1;
                            bestStart = left;
                        }
                        // ⚠ 跳到 left + 1 而不是 right + 1, 防漏重叠窗
                        i = left + 1;
                        j = 0;
                        continue;
                    }
                }
                i++;
            }
            return bestStart == -1 ? "" : s1.substr(bestStart, bestLen);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def minWindow(self, s1: str, s2: str) -> str:
            m, n = len(s1), len(s2)
            i = j = 0
            best_start, best_len = -1, float('inf')

            while i < m:
                if s1[i] == s2[j]:
                    j += 1
                    if j == n:                                         # s2 全匹配
                        right = i
                        j -= 1                                          # 退到 n-1
                        while j >= 0:                                   # 反扫缩窗
                            if s1[i] == s2[j]:
                                j -= 1
                            i -= 1
                        i += 1                                          # 修正
                        j += 1
                        left = i

                        if right - left + 1 < best_len:
                            best_len, best_start = right - left + 1, left

                        i = left + 1                                    # 跳到 left+1 重启
                        j = 0
                        continue
                i += 1

            return "" if best_start == -1 else s1[best_start:best_start + best_len]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s1
     * @param {string} s2
     * @return {string}
     */
    var minWindow = function(s1, s2) {
        const m = s1.length, n = s2.length;
        let i = 0, j = 0;
        let bestStart = -1, bestLen = Infinity;

        while (i < m) {
            if (s1[i] === s2[j]) {
                j++;
                if (j === n) {
                    const right = i;
                    j--;
                    while (j >= 0) {
                        if (s1[i] === s2[j]) j--;
                        i--;
                    }
                    i++; j++;
                    const left = i;

                    if (right - left + 1 < bestLen) {
                        bestLen = right - left + 1;
                        bestStart = left;
                    }
                    i = left + 1;
                    j = 0;
                    continue;
                }
            }
            i++;
        }
        return bestStart === -1 ? "" : s1.slice(bestStart, bestStart + bestLen);
    };
    ```

## Complexity

- **Time**: O(m × n) 最坏 — 多数 LC 数据接近 O(m + n).
- **Space**: O(1).

## 相关题目

- [0027. Remove Element](../0027-remove-element/README.md) — 同向双指针基础
- 0076\. Minimum Window Substring (待补) — 同款"最小窗口" 但**字符无序**, 用滑动窗口 + 哈希
- 0392\. Is Subsequence (待补) — 判定 t 是否 s 子序列, 双指针入门
- 0524\. Longest Word in Dictionary through Deleting (待补) — 双指针子序列匹配
- 0792\. Number of Matching Subsequences (待补) — 多 t 子序列匹配, 桶 + 推进指针
- 0115\. Distinct Subsequences — DP 计数版 → [§10 0115](../../10-dp/0115-distinct-subsequences/README.md)
