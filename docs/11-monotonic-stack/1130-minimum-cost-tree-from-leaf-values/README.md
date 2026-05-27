# 1130. Minimum Cost Tree From Leaf Values / 叶值的最小代价生成树 (单调栈解法)

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Greedy · 单调栈, 贪心
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-cost-tree-from-leaf-values/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

> 同一道题, 区间 DP 解法见 [§10 DP · 1130](../../10-dp/1130-minimum-cost-tree-from-leaf-values/README.md). 本页只讲 **O(n) 单调栈** 解法.

## Problem

**EN**: 给数组 `arr`, 按顺序作为某二叉树的叶子. 每个内部节点 = 左子树 max × 右子树 max, 树的总代价 = 所有内部节点之和. 求**最小代价**.

**中文**: `arr` 按序做叶子, 内部节点 = 左右子树 max 相乘, 求总和最小.

## Key Insights

1. **🔑 贪心观察: 小叶子应该尽早被"吃掉" / Small leaves should be merged out first**

    每个叶子在自己被合并 (变成某父节点的子) 时, 贡献一次乘积. **小叶子早点被吃掉, 跟它最小的大邻居配对**, 总代价最小:

    - 若小叶子留到最后, 它会跟更大的 max 相乘, 代价大.
    - 跟最小可能的对方相乘, 单次代价最小; 而它消失后, 周围的 max 关系跟没它一样.

    > 这是**"贪心一次性吃掉小元素"** 模式. 每个非最大叶子最终都跟它的"最小较大邻居"配对一次贡献.

2. **🔑 单调递减栈维护 / Decreasing stack**

    扫数组, 维护**单调递减栈** (栈顶是当前未被处理的最小值). 来一个新元素 `a`:

    - **`a < stack.back()`**: 直接入栈 (没有任何元素需要被它"吃").
    - **`a >= stack.back()`**: 栈顶 `mid` 找到了它**右边更大的邻居 a** — 现在该把 `mid` 吃掉了! `mid` 跟左 (`new top`) 和右 (`a`) 中**小的那个** 相乘, 加入答案. pop `mid`, 继续比较新栈顶跟 `a`.

    重复至栈顶 > a, 入栈 a.

3. **🔑 INT_MAX 哨兵防栈空 / Sentinel to avoid underflow**

    Yang `vector<int> st = {INT_MAX}` — 在栈底放 `INT_MAX`. 这样 `st.back()` 永远存在 (`min(a, st.back())` 在 mid 没左邻居时退化成 a), 不用判空.

    > 哨兵技巧再次出现 — 跟 [1547 cuts 补 0/n](../../10-dp/1547-minimum-cost-to-cut-a-stick/README.md) / [0375 dp(n+2)](../../10-dp/0375-guess-number-higher-or-lower-ii/README.md) 同精神.

4. **扫完后还有残留: 单调递减栈合并 / Drain remaining decreasing chain**

    扫描结束栈里可能剩单调递减序列 (除哨兵). 此时每个 mid 都没有更大的右邻居, 只能跟左邻居 (栈中下一个, 更大) 相乘. **从顶往下逐个吃**, 直到只剩哨兵 + 最大元素.

    ```cpp
    while (st.size() > 2) {
        int mid = st.back(); st.pop_back();
        res += mid * st.back();             // 只剩左邻居, 直接相乘
    }
    ```

5. **复杂度 O(n) 的来源 / Why O(n)**

    每个元素入栈一次, 出栈一次. 内层 while 总迭代数线性. **O(n) 时间, O(n) 空间** — 比 [DP O(n³)](../../10-dp/1130-minimum-cost-tree-from-leaf-values/README.md) 高效得多.

6. **跟 DP 解法的关系 / Relation to DP solution**

    DP 是通解, 单调栈是该特定结构下的"贪心 O(n) hack". 学过 [区间 DP 全套](../../10-dp/index.md) 之后再看单调栈解, 会发现它本质是"DP 转移结构 + 贪心切点选择" 的极致简化.

    > **DP → 找规律 → 贪心** 是经典优化路径. 同款例子: 0322 Coin Change DP vs 某些面额下的贪心.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int mctFromLeafValues(vector<int>& arr) {
            long res = 0;
            vector<int> st = {INT_MAX};                                         // 哨兵防栈空
            for (int a : arr) {
                while (st.back() <= a) {                                        // 栈顶可被 a 吃掉
                    int mid = st.back(); st.pop_back();
                    // mid 跟左 (new top) 和右 (a) 中更小的那个相乘
                    res += (long)mid * min(a, st.back());
                }
                st.push_back(a);
            }
            // 扫完仍有残留: 单调递减栈合并
            while (st.size() > 2) {
                int mid = st.back(); st.pop_back();
                res += (long)mid * st.back();
            }
            return (int)res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def mctFromLeafValues(self, arr: list[int]) -> int:
            # 单调递减栈, 栈底放 INF 哨兵防空
            st = [float('inf')]
            res = 0
            for a in arr:
                # 栈顶 ≤ a → 栈顶 mid 找到右大邻居 a, 吃掉
                while st[-1] <= a:
                    mid = st.pop()
                    res += mid * min(a, st[-1])                                 # 跟左右更小那个配对
                st.append(a)
            # 残留: 单调递减栈, 从顶往下逐个吃
            while len(st) > 2:
                mid = st.pop()
                res += mid * st[-1]
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} arr
     * @return {number}
     */
    var mctFromLeafValues = function(arr) {
        const st = [Infinity];                                                  // 哨兵
        let res = 0;
        for (const a of arr) {
            while (st[st.length - 1] <= a) {
                const mid = st.pop();
                res += mid * Math.min(a, st[st.length - 1]);
            }
            st.push(a);
        }
        while (st.length > 2) {
            const mid = st.pop();
            res += mid * st[st.length - 1];
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 每元素入栈出栈各一次.
- **Space**: O(n) — 栈最坏存全部元素.

## 相关题目

- [§10 DP · 1130 区间 DP 解法](../../10-dp/1130-minimum-cost-tree-from-leaf-values/README.md) — 同题 O(n³) DP 解, 通解但慢
- [0316. Remove Duplicate Letters](../0316-remove-duplicate-letters/README.md) — 单调栈 + 贪心 + 字典序
- 0496\. Next Greater Element I (待补) — 单调栈最基础
- 0739\. Daily Temperatures (待补) — 单调栈找下一个更大
- 0084\. Largest Rectangle in Histogram (待补) — 单调栈经典进阶
- 0962\. Maximum Width Ramp (待补) — 单调栈 + 双指针
