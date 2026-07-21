# 0424. Longest Repeating Character Replacement / 替换后的最长重复字符

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Sliding Window, Two Pointers, Counting, String · 滑动窗口, 双指针, 计数, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/longest-repeating-character-replacement/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Longest substring turnable into all-same via ≤ k replacements** → **sliding window + 26-count**; window valid ⇔ `windowLen - maxCount ≤ k`; **use `if` not `while` to shrink** (window only grows/holds), and **`maxCount` is a *high-water mark* — never shrinks**.
>
> **中文**: **最多 k 次替换后, 最长同字符子串** → **滑窗 + 26 计数**; 窗口合法 ⇔ `窗口长 - maxCount ≤ k`; **缩窗用 `if` 不用 `while`** (窗口只涨不缩), **`maxCount` 只增不减** (只关心历史最高).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s` 和整数 `k`. 可以**替换任意 ≤ k 个字符**. 求替换后**最长的"全同字符" 子串** 的长度.

- 例: `s = "AABABBA"`, `k = 1` → 4 (`"AABA" + 换 B` 或 `"BABB" + 换 A`).

**中文**: 最多改 k 次字符, 求全同字符最长子串.

## Key Insights

1. **🔑 窗口合法性: `windowLen - maxCount ≤ k` / Valid ⇔ len − maxCount ≤ k**

    对窗口 `[left, right]`:

    - **maxCount** = 窗口内**最频繁字符**的次数.
    - 若把其他字符**全换成**这个最频繁的, 需要 `windowLen - maxCount` 次替换.
    - **合法** ⇔ 这个替换次数 ≤ k.

    ```
    "AABAB"  window, maxCount('A') = 3, len = 5, k = 2
    需要换: 5 - 3 = 2 ≤ 2 ✅
    ```

    > **"变成全同 = 补差价 to maxCount"** 是这题的模型.

2. **🔑 灵魂 trick: `maxCount` **永远不减** — 高水位标记 / `maxCount` is a high-water mark**

    正统做法: 每次窗口变了, **重新扫 26 个 count 找 maxCount**. 但这样每步 O(26), 总 O(26n).

    **观察**: 我们**只关心 "曾经的 maxCount 是多少"**! 因为:

    - 若曾经 `maxCount = 5` 让窗口大小 = 5+k, 现在 `maxCount` 减到 3 → 新窗口能做的**最好也不过 3+k < 5+k**. **不会更新答案**.
    - → **让 `maxCount` 只增不减**, 反正**只会低估当前**, 但**结果 max 一样**.

    → **不需要重新扫 count**. 只在 `cnt[s[right]]++` 后 `maxCount = max(maxCount, cnt[s[right]])` 即可.

    > **"只关心最优解 → 状态可以 stale (陈旧)"** 是一个非常聪明的滑窗优化. 头一次见的题被震惊很正常.

3. **🔑 关键: 用 `if` 不用 `while` 缩窗 / `if`, not `while`, to shrink**

    ```cpp
    if (right - left + 1 - maxCount > k) {
        cnt[s[left] - 'A']--;
        left++;
    }
    ```

    - **`if`**: 违反时**只右移 left 一格**. 因为**right 每次涨 1**, 我们**只需要让 window "不再变大"** — 保持之前的最大合法大小.
    - **不用 while**: 若用 while 缩到"再次合法", 窗口可能缩到很小. **但答案不会更好** — 因为记录的 maxLen 是已经**捕获过的历史最大**. 缩掉不会漏解.

    **关键**: `if` 让 window **要么涨 (right 涨 left 不动) 要么滑 (两个都涨)**, **永远不缩**. 最终 maxLen = window 涨到的最大值.

    > **对撞感 vs 保持感**: 大部分滑窗是"该缩就缩", 本题是"**只保持最大合法尺寸**". 是**独特姿势**.

4. **🔑 26 计数数组 (`cnt[26]`) / Counting array**

    只用大写英文 → **`int cnt[26]`** 就够. 每次:

    - `cnt[s[right] - 'A']++` (加进窗).
    - `cnt[s[left] - 'A']--` (若缩窗, 移出).

    > **UB 提醒**: `int cnt[26];` 未初始化跟你之前 [0383](../../03-hash-table/0383-ransom-note/README.md), [0266](../../03-hash-table/0266-palindrome-permutation/README.md) 一样是雷. 你这次写了 `= {0}` ✅ — 修好了!

5. **🔑 每步都记 maxLen / Track maxLen every step**

    ```cpp
    maxLen = max(maxLen, right - left + 1);
    ```

    因为 window "只保持不缩", `right - left + 1` 就是**当前窗口大小**. 每步跟 maxLen 比取大.

6. **🔑 跟其他滑窗题的对比 / vs other sliding window**

    | 题 | 姿势 | 缩窗 |
    |---|---|---|
    | [0209](../0209-minimum-size-subarray-sum/README.md) 最短 | 满足即缩 (贪心缩) | `while` |
    | [0003](../0003-longest-substring-without-repeating-characters/README.md) 最长 | 违反才缩 | `while` |
    | **0424 (本题)** | 违反**滑一格** | **`if`** (window 只涨不缩) |

    > **三种滑窗姿势各有语境**. 本题是"**capture 最大历史**" 的第三种.

7. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear**

    - Time: 一遍扫, `if` 分支一次操作 O(1).
    - Space: 26 int 常量.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int characterReplacement(string s, int k) {
            int cnt[26] = {0};                                       // 显式清零
            int left = 0, maxCount = 0, maxLen = 0;
            for (int right = 0; right < (int)s.size(); right++) {
                cnt[s[right] - 'A']++;
                maxCount = max(maxCount, cnt[s[right] - 'A']);       // 只增不减 (高水位)
                if (right - left + 1 - maxCount > k) {                // 违反 → 滑一格 (if 非 while)
                    cnt[s[left] - 'A']--;
                    left++;
                }
                maxLen = max(maxLen, right - left + 1);
            }
            return maxLen;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def characterReplacement(self, s: str, k: int) -> int:
            # dict 简易版; 也可 list [0]*26. dict 无 char 编码开销
            cnt = {}
            left = max_count = max_len = 0
            for right, c in enumerate(s):
                cnt[c] = cnt.get(c, 0) + 1
                max_count = max(max_count, cnt[c])          # 高水位
                if right - left + 1 - max_count > k:
                    cnt[s[left]] -= 1
                    left += 1
                max_len = max(max_len, right - left + 1)
            return max_len
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {number} k
     * @return {number}
     */
    var characterReplacement = function(s, k) {
        // JS 用 Map 或 Object. Map 遍历有序但这里不需要
        const cnt = {};
        let left = 0, maxCount = 0, maxLen = 0;
        for (let right = 0; right < s.length; right++) {
            const c = s[right];
            cnt[c] = (cnt[c] || 0) + 1;
            maxCount = Math.max(maxCount, cnt[c]);
            if (right - left + 1 - maxCount > k) {
                cnt[s[left]]--;
                left++;
            }
            maxLen = Math.max(maxLen, right - left + 1);
        }
        return maxLen;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍扫, 每步 O(1).
- **Space**: O(1) — 26 int 或 hash map 常数级.

## 相关题目

- [0003. Longest Substring Without Repeating Characters](../0003-longest-substring-without-repeating-characters/README.md) — 最长无重复子串, `while` 缩
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 最短满足和 ≥ target
- [0011. Container With Most Water](../0011-container-with-most-water/README.md) — 对撞 + 贪心
- [0187. Repeated DNA Sequences](../../03-hash-table/0187-repeated-dna-sequences/README.md) — 定长滑窗 + 滚动哈希
- [0076. Minimum Window Substring](../0076-minimum-window-substring/README.md) — 最短窗口子串, Hard
- [0567. Permutation in String](../0567-permutation-in-string/README.md) — 定长滑窗 + 频次签名
- 0438\. Find All Anagrams in a String (待补) — 定长滑窗 + 频次
- 0904\. Fruit Into Baskets (待补) — 至多 2 种字符最长子数组
- 0159\. Longest Substring with At Most Two Distinct Characters (待补) — 广义 k 种字符版
- 0340\. Longest Substring with At Most K Distinct Characters (待补) — 更广义
