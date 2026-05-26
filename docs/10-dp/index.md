# 10 · DP / 动态规划

**EN**: State design, transitions, rolling-array space optimization.

**中文**: 状态设计、转移方程、滚动数组优化。

## 技巧 / Topics

- [DP 通用思维流程 (Thinking Process)](./topic-dp-thinking-process.md) — 看到 DP 题该怎么想: 识别 → 三要素 (状态/转移/初始) → 遍历 → 优化, 核心是"最后一步" 思维
- [折半搜索 (Meet in the Middle)](./topic-meet-in-the-middle.md) — 把 `2^n` 暴力枚举开方成 `2^(n/2)`, 背包不适用时的备选

## 题目分类 / Problems by Category

> 38 题分 9 类. 同类题共享状态模式 + 转移结构, 排序大体按学习顺序.

### 线性 DP / Linear

**特征**: 单序列 `dp[i]` 只依赖前几项, 多带"相邻约束".

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0509 | [Fibonacci Number / 斐波那契数](./0509-fibonacci-number/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0070 | [Climbing Stairs / 爬楼梯](./0070-climbing-stairs/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0746 | [Min Cost Climbing Stairs / 使用最小花费爬楼梯](./0746-min-cost-climbing-stairs/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0198 | [House Robber / 打家劫舍](./0198-house-robber/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0213 | [House Robber II / 打家劫舍 II](./0213-house-robber-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0337 | [House Robber III / 打家劫舍 III](./0337-house-robber-iii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 3840 | [House Robber V / 打家劫舍 V](./3840-house-robber-v/README.md) | Medium | ✅ | ☐ ☐ ☐ |

### 网格 DP / Grid

**特征**: 二维 `dp[i][j]`, 网格上路径/方案. 转移从上/左来.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0062 | [Unique Paths / 不同路径](./0062-unique-paths/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0063 | [Unique Paths II / 不同路径 II](./0063-unique-paths-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |

### 数学 / 经典 / Classic & Math

**特征**: "枚举切点 + 子问题相乘/求和", 卡特兰数家族.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0343 | [Integer Break / 整数拆分](./0343-integer-break/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0096 | [Unique Binary Search Trees / 不同的二叉搜索树](./0096-unique-binary-search-trees/README.md) | Medium | ✅ | ☐ ☐ ☐ |

### 背包 DP / Knapsack

**特征**: 物品 + 容量, "选/不选" 二元决策. 0/1 倒序 j, 完全正序 j.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0416 | [Partition Equal Subset Sum / 分割等和子集](./0416-partition-equal-subset-sum/README.md) (0/1 母题) | Medium | ✅ | ☐ ☐ ☐ |
| 1049 | [Last Stone Weight II / 最后一块石头的重量 II](./1049-last-stone-weight-ii/README.md) (0/1 最小差) | Medium | ✅ | ☐ ☐ ☐ |
| 0494 | [Target Sum / 目标和](./0494-target-sum/README.md) (0/1 计数) | Medium | ✅ | ☐ ☐ ☐ |
| 0474 | [Ones and Zeroes / 一和零](./0474-ones-and-zeroes/README.md) (二维 0/1) | Medium | ✅ | ☐ ☐ ☐ |
| 0518 | [Coin Change II / 零钱兑换 II](./0518-coin-change-ii/README.md) (完全, 组合数) | Medium | ✅ | ☐ ☐ ☐ |
| 0377 | [Combination Sum IV / 组合总和 Ⅳ](./0377-combination-sum-iv/README.md) (完全, 排列数) | Medium | ✅ | ☐ ☐ ☐ |
| 0322 | [Coin Change / 零钱兑换](./0322-coin-change/README.md) (完全, 最值) | Medium | ✅ | ☐ ☐ ☐ |
| 0279 | [Perfect Squares / 完全平方数](./0279-perfect-squares/README.md) (完全, 最值) | Medium | ✅ | ☐ ☐ ☐ |
| 0139 | [Word Break / 单词拆分](./0139-word-break/README.md) (完全, 判定) | Medium | ✅ | ☐ ☐ ☐ |

### 状态机 DP / State Machine (股票系列)

**特征**: 每天/每步多个"身份" 状态间转移. 持有/不持有/冷冻等.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0121 | [Best Time to Buy and Sell Stock / 买卖股票的最佳时机](./0121-best-time-to-buy-and-sell-stock/README.md) (1 次, 母题) | Easy | ✅ | ☐ ☐ ☐ |
| 0123 | [Best Time to Buy and Sell Stock III / 买卖股票的最佳时机 III](./0123-best-time-to-buy-and-sell-stock-iii/README.md) (最多 2 次) | Hard | ✅ | ☐ ☐ ☐ |
| 0188 | [Best Time to Buy and Sell Stock IV / 买卖股票的最佳时机 IV](./0188-best-time-to-buy-and-sell-stock-iv/README.md) (最多 k 次) | Hard | ✅ | ☐ ☐ ☐ |
| 0309 | [Best Time to Buy and Sell Stock with Cooldown / 最佳买卖股票时机含冷冻期](./0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) (∞ + 冷冻) | Medium | ✅ | ☐ ☐ ☐ |
| 0714 | [Best Time to Buy and Sell Stock with Transaction Fee / 买卖股票的最佳时机含手续费](./0714-best-time-to-buy-and-sell-stock-with-transaction-fee/README.md) (∞ + 手续费) | Medium | ✅ | ☐ ☐ ☐ |

> 0122 (∞ 次, 贪心) 在 [§09 0122](../09-greedy/0122-best-time-to-buy-and-sell-stock-ii/README.md), 也带 DP 版本对照.

### 序列 DP / Single Sequence

**特征**: 单数组/字符串, `dp[i]` = "以 i 结尾" 的最优. 答案常需 `max(dp[*])`.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0300 | [Longest Increasing Subsequence / 最长递增子序列](./0300-longest-increasing-subsequence/README.md) (LIS, 含 O(n log n) 二分) | Medium | ✅ | ☐ ☐ ☐ |
| 0674 | [Longest Continuous Increasing Subsequence / 最长连续递增序列](./0674-longest-continuous-increasing-subsequence/README.md) (LIS 连续版) | Easy | ✅ | ☐ ☐ ☐ |
| 0152 | [Maximum Product Subarray / 乘积最大子数组](./0152-maximum-product-subarray/README.md) (Kadane 乘积版) | Medium | ✅ | ☐ ☐ ☐ |

> Kadane 母题 0053 在 [§09 贪心](../09-greedy/0053-maximum-subarray/README.md).

### 双序列 DP / Double Sequence

**特征**: 二维 `dp[i][j]`, 两个数组/字符串配对. 子数组强制双端结尾, 子序列允许跳.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0718 | [Maximum Length of Repeated Subarray / 最长重复子数组](./0718-maximum-length-of-repeated-subarray/README.md) (子数组, 强制结尾) | Medium | ✅ | ☐ ☐ ☐ |
| 1143 | [Longest Common Subsequence / 最长公共子序列](./1143-longest-common-subsequence/README.md) (LCS 母题, 子序列) | Medium | ✅ | ☐ ☐ ☐ |
| 1035 | [Uncrossed Lines / 不相交的线](./1035-uncrossed-lines/README.md) (= LCS 变形) | Medium | ✅ | ☐ ☐ ☐ |
| 0115 | [Distinct Subsequences / 不同的子序列](./0115-distinct-subsequences/README.md) (LCS 计数版) | Hard | ✅ | ☐ ☐ ☐ |
| 0583 | [Delete Operation for Two Strings / 两个字符串的删除操作](./0583-delete-operation-for-two-strings/README.md) (LCS + 公式) | Medium | ✅ | ☐ ☐ ☐ |
| 0072 | [Edit Distance / 编辑距离](./0072-edit-distance/README.md) (三操作终极版) | Medium | ✅ | ☐ ☐ ☐ |

### 区间 DP / Interval

**特征**: `dp[i][j]` 表示区间 `[i, j]` 的最优. **i 倒序, j 顺序** 是这类的标志.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0647 | [Palindromic Substrings / 回文子串](./0647-palindromic-substrings/README.md) (子串判定) | Medium | ✅ | ☐ ☐ ☐ |
| 0516 | [Longest Palindromic Subsequence / 最长回文子序列](./0516-longest-palindromic-subsequence/README.md) (子序列) | Medium | ✅ | ☐ ☐ ☐ |
| 0312 | [Burst Balloons / 戳气球](./0312-burst-balloons/README.md) ("枚举最后破" 经典) | Hard | ✅ | ☐ ☐ ☐ |
| 1000 | [Minimum Cost to Merge Stones / 合并石头的最低成本](./1000-minimum-cost-to-merge-stones/README.md) (3D 区间 DP) | Hard | ✅ | ☐ ☐ ☐ |
| 1312 | [Minimum Insertion Steps to Make a String Palindrome / 让字符串成为回文串的最少插入次数](./1312-minimum-insertion-steps-to-make-a-string-palindrome/README.md) (LPS 镜像) | Hard | ✅ | ☐ ☐ ☐ |
| 1039 | [Minimum Score Triangulation of Polygon / 多边形三角剖分的最低得分](./1039-minimum-score-triangulation-of-polygon/README.md) (0312 结构孪生) | Medium | ✅ | ☐ ☐ ☐ |
| 1547 | [Minimum Cost to Cut a Stick / 切棍子的最小成本](./1547-minimum-cost-to-cut-a-stick/README.md) (哨兵 + 排序 + 切点) | Hard | ✅ | ☐ ☐ ☐ |
| 0730 | [Count Different Palindromic Subsequences / 统计不同回文子序列](./0730-count-different-palindromic-subsequences/README.md) (容斥去重) | Hard | ✅ | ☐ ☐ ☐ |
| 1771 | [Maximize Palindrome Length From Subsequences / 由子序列构造的最长回文串的长度](./1771-maximize-palindrome-length-from-subsequences/README.md) (LPS + 端点跨界约束) | Hard | ✅ | ☐ ☐ ☐ |

### 折半搜索 / Meet in the Middle

**特征**: DP 失效 (值域太大) + n 中等 (≤ 40), 用 `2^(n/2)` 暴力枚举两半再合并. 见 [topic 页](./topic-meet-in-the-middle.md).

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 2035 | [Partition Array Into Two Arrays to Minimize Sum Difference / 将数组分成两个数组并最小化和的差](./2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) (折半 + popcount 分桶) | Hard | ✅ | ☐ ☐ ☐ |
| 1755 | [Closest Subsequence Sum / 最接近目标值的子序列和](./1755-closest-subsequence-sum/README.md) (折半纯模板) | Hard | ✅ | ☐ ☐ ☐ |
