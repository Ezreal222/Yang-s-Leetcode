# 0455. Assign Cookies / 分发饼干

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Greedy, Two Pointers, Sort · 贪心, 双指针, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/assign-cookies/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given children's greed factors `g[]` (each child wants ≥ `g[i]` cookie size) and cookie sizes `s[]` (each cookie used at most once), return the **maximum** number of children that can be satisfied.

**中文**: 给孩子的胃口 `g[]` (孩子 i 要 ≥ `g[i]` 大小的饼干才满足) 和饼干尺寸 `s[]` (每块只能用一次), 求最多能满足的孩子数.

## 思路

### Core idea

**两数组都升序排序 + 两指针扫**:

- `i` 走 cookies, `j` 走 children.
- `s[i] >= g[j]` (这块饼干够喂当前最小胃口的孩子) → 配对成功, 两指针都进, `res++`.
- `s[i] < g[j]` → 这块太小, 跳到下一块 (只进 i).

最贪的策略: **用现有最小够用的饼干喂现有最小胃口的孩子**.

### Key Insights

1. **贪心正确性: 交换论证 / Why this greedy works**

    假设有最优解里"较小的饼干 X 没给胃口最小的孩子 j, 反而给了胃口更大的孩子 k". 我们可以**交换**: 让 X 喂 j (因为 X 够大喂 k 自然够大喂 j), 把原来喂 j 的那块 Y 留给 k (Y ≥ g[j] 不一定 ≥ g[k], 但 g[k] ≥ g[j], 如果 Y ≥ g[k] 还能配对就不损失; 不行也最多损失 0 个孩子 — 总数不会变小).

    所以"小喂小" 永远不劣于其它任何配对, 局部最优 ⇒ 全局最优.

2. **同款两指针 + 双 sort 的"贪心配对"模板 / Greedy pairing template**

    | 题 | 双数组 | 配对条件 | 贪心方向 |
    |---|---|---|---|
    | **0455 (本题)** | g[], s[] | s[i] >= g[j] | 小喂小 |
    | 0392 Is Subsequence (待补) | s, t (子串/母串) | 等于 | 顺序匹配 |
    | 0925 Long Pressed Name (待补) | name, typed | 等于 + 长按容忍 | 类似 |

    都是"双有序数组 + 单指针线性扫" 套路, 不同的只是配对条件.

3. **反向贪心 (大饼干喂大胃口) 也对 / Symmetric greedy works too**

    倒过来扫: 用最大饼干喂最大胃口, 配不上换次大孩子. 对称的贪心, 同样能证明最优. 两种写法二选一, Yang 的"小喂小" 写法读起来更顺.

4. **指针推进的不对称 / Asymmetric advance**

    配对成功: 两指针都进. 配对失败 (饼干太小): **只进 i**, j 不动 (这块饼干不行, 下一块继续试当前孩子). 写成"两指针都进" 会**跳过本该试的孩子**, 一类小错.

    Yang 的代码用 `s[i++] >= g[j]` 把 i 的推进合并到判定里, 然后 `if true { j++; res++ }`. 等价但稍隐式, 读者要看清.

### 一句话总结

**两数组升序 sort, 两指针扫, 现有最小够用的饼干配现有最小胃口的孩子. 配上 res++ 两指针都进, 配不上只进饼干指针.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findContentChildren(vector<int>& g, vector<int>& s) {
            sort(g.begin(), g.end());                              // 胃口升序
            sort(s.begin(), s.end());                              // 饼干升序
            int res = 0, i = 0, j = 0;
            while (i < (int)s.size() && j < (int)g.size()) {
                if (s[i++] >= g[j]) {                              // i++ 在判定里; 不管成功与否 i 都进
                    j++;                                           // 仅成功时 j 进
                    res++;
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findContentChildren(self, g: list[int], s: list[int]) -> int:
            g.sort()                                               # in-place 升序
            s.sort()
            i = j = res = 0
            while i < len(s) and j < len(g):
                if s[i] >= g[j]:
                    j += 1
                    res += 1
                i += 1                                             # 显式分离 i 推进, 比 i++ 在判定里更清晰
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} g
     * @param {number[]} s
     * @return {number}
     */
    var findContentChildren = function(g, s) {
        g.sort((a, b) => a - b);                                   // 数字排序必须给 compareFn (字典序会出错)
        s.sort((a, b) => a - b);
        let i = 0, j = 0, res = 0;
        while (i < s.length && j < g.length) {
            if (s[i] >= g[j]) {
                j++;
                res++;
            }
            i++;
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(m log m + n log n) — sort 主导, 之后线性扫.
- **Space**: O(1) 额外 (sort in-place).

## 易错点

- **必须先 sort 两个数组**: 没排序贪心策略直接失效, 配对乱来. 这是这题的入场券.
- **配对失败时只进 i, 不进 j**: 当前饼干不行, 同一个孩子还能等下一块. 两指针都进会跳过本该试的孩子 → 答案少.

## 相关题目

- [0040. Combination Sum II](../../08-backtracking/0040-combination-sum-ii/README.md) — 同款"sort + 相邻判定" 思路 (那里是回溯里去重)
- 0392\. Is Subsequence (待补) — 同款两指针扫两数组
- 0860\. Lemonade Change (待补) — 贪心模板入门
- 0135\. Candy (待补) — 双向贪心扫两遍, 经典进阶
- 1029\. Two City Scheduling (待补) — 按 (costA - costB) 排序的贪心
