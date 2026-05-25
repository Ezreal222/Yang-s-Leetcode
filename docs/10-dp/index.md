# 10 · DP / 动态规划

**EN**: State design, transitions, rolling-array space optimization.

**中文**: 状态设计、转移方程、滚动数组优化。

## 技巧 / Topics

- [DP 通用思维流程 (Thinking Process)](./topic-dp-thinking-process.md) — 看到 DP 题该怎么想: 识别 → 三要素 (状态/转移/初始) → 遍历 → 优化, 核心是"最后一步" 思维
- [折半搜索 (Meet in the Middle)](./topic-meet-in-the-middle.md) — 把 `2^n` 暴力枚举开方成 `2^(n/2)`, 背包不适用时的备选

## 题目 / Problems

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0509 | [Fibonacci Number / 斐波那契数](./0509-fibonacci-number/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0070 | [Climbing Stairs / 爬楼梯](./0070-climbing-stairs/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0746 | [Min Cost Climbing Stairs / 使用最小花费爬楼梯](./0746-min-cost-climbing-stairs/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0062 | [Unique Paths / 不同路径](./0062-unique-paths/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0063 | [Unique Paths II / 不同路径 II](./0063-unique-paths-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0343 | [Integer Break / 整数拆分](./0343-integer-break/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0096 | [Unique Binary Search Trees / 不同的二叉搜索树](./0096-unique-binary-search-trees/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0416 | [Partition Equal Subset Sum / 分割等和子集](./0416-partition-equal-subset-sum/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 1049 | [Last Stone Weight II / 最后一块石头的重量 II](./1049-last-stone-weight-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 2035 | [Partition Array Into Two Arrays to Minimize Sum Difference / 将数组分成两个数组并最小化和的差](./2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) | Hard | ✅ | ☐ ☐ ☐ |
| 1755 | [Closest Subsequence Sum / 最接近目标值的子序列和](./1755-closest-subsequence-sum/README.md) | Hard | ✅ | ☐ ☐ ☐ |
| 0494 | [Target Sum / 目标和](./0494-target-sum/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0474 | [Ones and Zeroes / 一和零](./0474-ones-and-zeroes/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0518 | [Coin Change II / 零钱兑换 II](./0518-coin-change-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0377 | [Combination Sum IV / 组合总和 Ⅳ](./0377-combination-sum-iv/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0322 | [Coin Change / 零钱兑换](./0322-coin-change/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0279 | [Perfect Squares / 完全平方数](./0279-perfect-squares/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0139 | [Word Break / 单词拆分](./0139-word-break/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0198 | [House Robber / 打家劫舍](./0198-house-robber/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0213 | [House Robber II / 打家劫舍 II](./0213-house-robber-ii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0337 | [House Robber III / 打家劫舍 III](./0337-house-robber-iii/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 3840 | [House Robber V / 打家劫舍 V](./3840-house-robber-v/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0121 | [Best Time to Buy and Sell Stock / 买卖股票的最佳时机](./0121-best-time-to-buy-and-sell-stock/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0123 | [Best Time to Buy and Sell Stock III / 买卖股票的最佳时机 III](./0123-best-time-to-buy-and-sell-stock-iii/README.md) | Hard | ✅ | ☐ ☐ ☐ |
| 0188 | [Best Time to Buy and Sell Stock IV / 买卖股票的最佳时机 IV](./0188-best-time-to-buy-and-sell-stock-iv/README.md) | Hard | ✅ | ☐ ☐ ☐ |
| 0309 | [Best Time to Buy and Sell Stock with Cooldown / 最佳买卖股票时机含冷冻期](./0309-best-time-to-buy-and-sell-stock-with-cooldown/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0714 | [Best Time to Buy and Sell Stock with Transaction Fee / 买卖股票的最佳时机含手续费](./0714-best-time-to-buy-and-sell-stock-with-transaction-fee/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0300 | [Longest Increasing Subsequence / 最长递增子序列](./0300-longest-increasing-subsequence/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0674 | [Longest Continuous Increasing Subsequence / 最长连续递增序列](./0674-longest-continuous-increasing-subsequence/README.md) | Easy | ✅ | ☐ ☐ ☐ |
| 0718 | [Maximum Length of Repeated Subarray / 最长重复子数组](./0718-maximum-length-of-repeated-subarray/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 1143 | [Longest Common Subsequence / 最长公共子序列](./1143-longest-common-subsequence/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 1035 | [Uncrossed Lines / 不相交的线](./1035-uncrossed-lines/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0152 | [Maximum Product Subarray / 乘积最大子数组](./0152-maximum-product-subarray/README.md) | Medium | ✅ | ☐ ☐ ☐ |
| 0115 | [Distinct Subsequences / 不同的子序列](./0115-distinct-subsequences/README.md) | Hard | ✅ | ☐ ☐ ☐ |
| 0583 | [Delete Operation for Two Strings / 两个字符串的删除操作](./0583-delete-operation-for-two-strings/README.md) | Medium | ✅ | ☐ ☐ ☐ |
