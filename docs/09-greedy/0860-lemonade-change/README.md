# 0860. Lemonade Change / 柠檬水找零

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Greedy, Array · 贪心, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/lemonade-change/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Lemonade costs $5. Customers pay in $5, $10, or $20 bills (in `bills[]` order). Starting with no money, can you give correct change to every customer in order?

**中文**: 柠檬水 5 元. 顾客按 `bills[]` 顺序付钱, 每个用 5 / 10 / 20. 起始无任何零钱, 判断能否给每个顾客找对零.

## Key Insights

1. **核心贪心: 20 找零优先 10+5, 不是 5+5+5 / Prefer big bill first**

    给 20 找零 (需要 15) 有两种组合: `10 + 5` 或 `5 + 5 + 5`. **必须优先 10+5**.

    原因: 5 用途更广 — 既能给 10 找零, 也能给 20 找零. 10 只能给 20 找零. 多保留 5 才能应付未来. 用 `5+5+5` 找 20 会把 5 用光, 下次遇到 10 就死.

    一句话: **小面额留着, 大面额先花**.

2. **只跟踪 5 和 10 的数量, 不需要 20 / 20 is useless for change**

    收到 20 不能用作任何后续找零 (没有 25 / 35 / 50 这种 bill). 所以连数都不用数, 直接进账抽屉.

3. **贪心正确性 / Exchange argument**

    任何"用 5+5+5 找 20" 的解, 都能改写成"用 10+5 找 20" — 只要当时有 10 可用. 这个改写不会变差: 它保留了 +2 个 5 (省下来), 多消耗一个 10 (反正 10 只能给 20 找). 局部最优 ⇒ 全局最优.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool lemonadeChange(vector<int>& bills) {
            int count5 = 0, count10 = 0;
            for (int b : bills) {
                if (b == 5) {
                    count5++;
                } else if (b == 10) {
                    if (count5 == 0) return false;
                    count5--;
                    count10++;
                } else {                                          // b == 20
                    // 优先用 10 + 5 (保留更多 5)
                    if (count10 > 0 && count5 > 0) {
                        count10--;
                        count5--;
                    } else if (count5 >= 3) {                     // 退而求其次: 三个 5
                        count5 -= 3;
                    } else {
                        return false;
                    }
                }
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lemonadeChange(self, bills: list[int]) -> bool:
            count5 = count10 = 0
            for b in bills:
                if b == 5:
                    count5 += 1
                elif b == 10:
                    if count5 == 0:
                        return False
                    count5 -= 1
                    count10 += 1
                else:                                             # b == 20
                    if count10 > 0 and count5 > 0:                # 优先 10+5
                        count10 -= 1
                        count5 -= 1
                    elif count5 >= 3:                             # 退而求其次 5+5+5
                        count5 -= 3
                    else:
                        return False
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} bills
     * @return {boolean}
     */
    var lemonadeChange = function(bills) {
        let count5 = 0, count10 = 0;
        for (const b of bills) {
            if (b === 5) {
                count5++;
            } else if (b === 10) {
                if (count5 === 0) return false;
                count5--;
                count10++;
            } else {                                              // b === 20
                if (count10 > 0 && count5 > 0) {                  // 优先 10+5
                    count10--;
                    count5--;
                } else if (count5 >= 3) {
                    count5 -= 3;
                } else {
                    return false;
                }
            }
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n).
- **Space**: O(1).

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 贪心入门, 双 sort + 两指针
- [1005. Maximize Sum Of Array After K Negations](../1005-maximize-sum-of-array-after-k-negations/README.md) — 同款"优先用大的" 贪心
- [0134. Gas Station](../0134-gas-station/README.md) — 一遍扫贪心
- 0322\. Coin Change (待补) — 找零进阶: 任意面额求最少硬币, DP 不再是贪心
- 0518\. Coin Change II (待补) — 找零方案数, DP
- 0763\. Partition Labels (待补) — 同款一遍扫贪心 + 区间合并
