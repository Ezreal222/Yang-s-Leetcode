# 0376. Wiggle Subsequence / 摆动序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, DP · 贪心, 动态规划
    - **Link**: [LeetCode](https://leetcode.com/problems/wiggle-subsequence/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: A wiggle sequence has strictly alternating differences (positive, negative, positive, …). Given `nums`, return the length of the **longest wiggle subsequence** (delete some elements, no re-order).

**中文**: 摆动序列 = 相邻差值正负交替 (严格). 给 `nums`, 求最长摆动子序列长度 (可以删元素, 不重排).

## 思路

### Core idea

**贪心数峰谷**. 不需要真的构造子序列, 只数转折点:

- 初始 `res = 1` (序列至少包含 nums[0]).
- 扫每对相邻差 `curDiff = nums[i+1] - nums[i]`.
- 若跟 `preDiff` 异号 (一升一降) → 找到一个转折, `res++`, **更新 preDiff = curDiff**.
- 平的 (diff == 0) 不算转折, **preDiff 保留之前方向**, 等下一个真转折.

### Key Insights

1. **`preDiff <= 0` 的 `<=` 处理"开局平台" / The `<=` handles initial plateaus**

    判定条件: `(preDiff >= 0 && curDiff < 0) || (preDiff <= 0 && curDiff > 0)`.

    用 `<=` / `>=` (含等号) 是关键: `preDiff` 初值 = 0, 第一对差出来必须能匹配 ("第一次有了真正方向, 等同于一次转折"). 写 `< / >` 会丢掉序列的第一个有效转折.

    例: `[1, 2, 2, 1]` 答案 2. 第一对 `2-1=1 > 0`, `preDiff = 0`. 用 `<=` 才能命中 `(0 <= 0 && 1 > 0)` → `res = 2`.

2. **`preDiff = curDiff` 只在 `res++` 时更新 / Asymmetric update is THE platform trick**

    每步都更新就完蛋: 遇平台 (curDiff == 0), preDiff 被覆盖成 0, 之后判异号会失常.

    Yang 的写法**只在确认转折时更新** — 平台期间 preDiff 保留旧方向, 直到下一个真转折出现, 才能正确识别"方向变了".

    例: `[1, 2, 2, 2, 1]` 答案 2. 走完平台 (preDiff 还是 1), 遇到 1 - 2 = -1 → `(1 >= 0 && -1 < 0)` ✅ res = 2. 如果中途把 preDiff 设成 0, 这个转折会被误判.

3. **替代解: DP / Alternative O(n) DP**

    ```cpp
    int up = 1, down = 1;
    for (int i = 1; i < n; i++) {
        if (nums[i] > nums[i-1])      up = down + 1;
        else if (nums[i] < nums[i-1]) down = up + 1;
        // ==: 都不动
    }
    return max(up, down);
    ```
    `up` = "最后一对差为正" 的最长波动长度, `down` = "最后一对差为负". 状态转移很直观. **贪心版代码更短, 但 DP 版更容易解释正确性**.

4. **为什么贪心正确 / Why counting peaks/valleys is enough**

    波动子序列的"转折点" 就是原序列里的"局部峰" 和"局部谷". 任何最长波动子序列必然取一个开头 + 所有峰谷 (中间平地或单调段都不用). 数峰谷 + 1 = 长度.

5. **edge: n == 1 直接返 1**

    单元素序列就是长度 1 的波动. Yang 显式判了, 也可以让 for 不进入 (`nums.size() - 1 == 0`) 自然返 1, 等价.

### 一句话总结

**贪心数峰谷: preDiff vs curDiff 异号就 res++ 并更新 preDiff. 关键: 用 `<=`/`>=` 处理开局, 只在转折时更新 preDiff 处理平台.**

### 图解

`nums = [1, 7, 4, 9, 2, 5]`:

```
差: [6, -3, 5, -7, 3]
符: [+, -,  +, -,  +]      每个相邻差都翻号 → 全是转折
res: 1 + 5 = 6
```

`nums = [1, 17, 5, 10, 13, 15, 10, 5, 16, 8]`:
```
差:  [16, -12, 5, 3, 2, -5, -5, 11, -8]
符:  [+,  -,   +, +, +, -,  -,  +,  -]
转折: ✓  ✓    ✓  -  -  ✓   -   ✓   ✓
res = 1 + 6 = 7
```

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int wiggleMaxLength(vector<int>& nums) {
            if (nums.size() == 1) return 1;
            int res = 1, preDiff = 0, curDiff = 0;
            for (int i = 0; i < (int)nums.size() - 1; i++) {
                curDiff = nums[i + 1] - nums[i];
                // <= / >= 处理开局 (preDiff 初值 0); 异号判定
                if ((preDiff >= 0 && curDiff < 0) || (preDiff <= 0 && curDiff > 0)) {
                    res++;
                    preDiff = curDiff;                       // 关键: 只在转折时更新, 处理平台
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def wiggleMaxLength(self, nums: list[int]) -> int:
            if len(nums) == 1:
                return 1
            res, pre_diff = 1, 0
            for i in range(len(nums) - 1):
                cur_diff = nums[i + 1] - nums[i]
                # 异号判定: <= / >= 含等号; 只在转折时更新 pre_diff (平台 trick)
                if (pre_diff >= 0 and cur_diff < 0) or (pre_diff <= 0 and cur_diff > 0):
                    res += 1
                    pre_diff = cur_diff
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var wiggleMaxLength = function(nums) {
        if (nums.length === 1) return 1;
        let res = 1, preDiff = 0;
        for (let i = 0; i < nums.length - 1; i++) {
            const curDiff = nums[i + 1] - nums[i];
            if ((preDiff >= 0 && curDiff < 0) || (preDiff <= 0 && curDiff > 0)) {
                res++;
                preDiff = curDiff;                           // 只在转折时更新
            }
        }
        return res;
    };
    ```

### 替代: DP 版

```cpp
int wiggleMaxLength(vector<int>& nums) {
    int up = 1, down = 1;
    for (int i = 1; i < (int)nums.size(); i++) {
        if (nums[i] > nums[i-1])      up = down + 1;
        else if (nums[i] < nums[i-1]) down = up + 1;
    }
    return max(up, down);
}
```
代码更长一点但语义直白: `up[i]` / `down[i]` 是"最后一段是升 / 降" 的最长波动. 不需要 preDiff 平台 trick.

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 易错点

- **`<=` 不是 `<`**: 含等号才能处理 preDiff 初值 0 (开局) 和单元素平台. 写 `<` 会漏第一个转折.
- **`preDiff = curDiff` 只在转折时更新**: 每步都更新 → 平台 (diff=0) 会污染 preDiff, 之后判异号失常. 这是这题最容易写错的一行.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门, 双 sort + 双指针
- 0300\. Longest Increasing Subsequence (待补) — 同款"子序列长度", 但单调递增, 贪心 + 二分 / DP
- 0053\. Maximum Subarray (待补) — 同款一遍贪心, 累加子段和
- 0738\. Monotone Increasing Digits (待补) — 贪心遇平台需要回退处理, 跟本题"平台" 思想呼应
- 0978\. Longest Turbulent Subarray (待补) — 摆动的"子数组" 版 (连续), 同 DP 思路
