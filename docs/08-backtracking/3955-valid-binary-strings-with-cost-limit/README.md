# 3955. Valid Binary Strings With Cost Limit / 满足代价限制的有效二进制字符串

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, String · 回溯, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/valid-binary-strings-with-cost-limit/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给整数 `n` 和 `k`. 二进制串 `s` 的**代价** = 所有 `s[i] == '1'` 的下标之和 (0-indexed). 一个串**有效** 当且仅当:

1. 不含两个**连续** 的 `'1'`
2. 代价 `≤ k`

返回所有长度为 `n` 的有效串.

**中文**: 长度 n 的二进制串, 1 不能相邻 + 1 的下标和 ≤ k, 返回所有合法串.

## Key Insights

1. **🔑 回溯枚举"在哪些下标放 1" / Backtrack over positions of 1s**

    全 0 串自动合法 (cost = 0, 无相邻). 在此基础上"添加一些 1" → 枚举 1 的位置即可. 跟 [0078 Subsets](../0078-subsets/README.md) 的"选哪些下标" 同思路.

2. **🔑 三个剪枝同时生效 / Three constraints in one loop**

    Yang 的核心 backtrack:

    ```cpp
    for (int i = index; i < n; i++) {
        if (cost + i > k) break;                                            // 剪枝 1: cost 超限
        cur[i] = '1';
        backtrack(n, k, cost + i, cur, i + 2);                              // 剪枝 2: 跳到 i + 2 防相邻
        cur[i] = '0';
    }
    ```

    - **剪枝 1** (`cost + i > k` 直接 break): 内层 `i` 升序, 若当前 i 已经超, 后续 i 必更大也超 → break 而非 continue, 省一堆无效尝试.
    - **剪枝 2** (`i + 2` 跳一格): 在 i 放了 1, **下一个 1 至少放在 i + 2** (i + 1 会相邻). 直接 `index = i + 2` 约束子问题, 不用单独维护"上一个 1 的位置".
    - **剪枝 3** (`index` 参数): 防重复枚举同一组合 — 后续只在 i 之后选, 不回头.

    > **三剪枝同时生效**: cost / 相邻 / 顺序. 写在一个 for 里, 函数干净.

3. **🔑 在 backtrack 入口就 `res.push_back(cur)` / Record at every node**

    每一层递归一进来就把当前 `cur` 加入答案. 这意味着**每个"加完一些 1 之后的状态"** 都是一个合法串:

    - 进入根层 → 加全 0 串
    - 加一个 1 后递归 → 加只有这一个 1 的串
    - 再加一个 → 加两个 1 的串
    - ...

    > **每节点 push** 是"求所有子集" 套路 (跟 0078 / 0090 Subsets 一样). 跟"只在叶子 push" 的"求所有排列" 不同, 别混.

4. **`i + 2` 同时避免相邻 + 顺序 / `i + 2` solves two problems**

    若写 `i + 1`, 子问题里能选 i + 1 → 跟 i 相邻. 写 `i + 2` 直接禁止. **小常数差别但语义关键**.

5. **复杂度 O(2^n × n) 上界, 实际剪枝下小得多 / Worst-case exponential**

    长度 n 的无相邻 1 串个数是 Fibonacci F(n+2), 数量级 φ^n ≈ 1.618^n. 加 cost 限制后更小. LC 数据 `n ≤ 12` → 至多几百串, 完全可以.

6. **代码顺手做 fast prefix 截断 / Push-then-extend pattern**

    Yang 的设计 — **先 push, 再尝试扩展** — 不需要"完工后判合法" 的最后一步. 每个状态本身就是合法的串 (cost 已剪枝, 相邻已剪枝), 直接收.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> res;

        vector<string> generateValidStrings(int n, int k) {
            string s(n, '0');                                               // 起手全 0
            backtrack(n, k, 0, s, 0);
            return res;
        }

    private:
        void backtrack(int n, int k, int cost, string& cur, int index) {
            res.push_back(cur);                                             // 当前状态即合法串
            for (int i = index; i < n; i++) {
                if (cost + i > k) break;                                    // 剪枝 1: cost 升序超限
                cur[i] = '1';
                backtrack(n, k, cost + i, cur, i + 2);                      // 跳 i + 2 防相邻
                cur[i] = '0';                                               // 回溯
            }
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def generateValidStrings(self, n: int, k: int) -> list[str]:
            res = []
            cur = list('0' * n)                                             # 用 list 方便改 (str 不可变)

            def backtrack(index: int, cost: int) -> None:
                res.append(''.join(cur))                                    # 每个状态即合法串
                for i in range(index, n):
                    if cost + i > k:                                        # 升序超限直接 break
                        break
                    cur[i] = '1'
                    backtrack(i + 2, cost + i)                              # 跳 i + 2 防相邻
                    cur[i] = '0'

            backtrack(0, 0)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @param {number} k
     * @return {string[]}
     */
    var generateValidStrings = function(n, k) {
        const res = [];
        const cur = new Array(n).fill('0');

        const backtrack = (index, cost) => {
            res.push(cur.join(''));
            for (let i = index; i < n; i++) {
                if (cost + i > k) break;                                    // 剪枝
                cur[i] = '1';
                backtrack(i + 2, cost + i);                                 // 跳 i + 2
                cur[i] = '0';
            }
        };

        backtrack(0, 0);
        return res;
    };
    ```

## Complexity

- **Time**: O(F(n+2) × n) — Fibonacci 数量级的合法串数 × 每串 O(n) 拷贝.
- **Space**: O(n) 递归栈 + O(总输出) 答案存储.

## 相关题目

- [0078. Subsets](../0078-subsets/README.md) — 每节点 push 的模板, 跟本题"枚举哪些下标选" 同思想
- [0090. Subsets II](../0090-subsets-ii/README.md) — 含重复元素的子集
- [0131. Palindrome Partitioning](../0131-palindrome-partitioning/README.md) — 同款"在 index 起步遍历" 回溯
- [0093. Restore IP Addresses](../0093-restore-ip-addresses/README.md) — 字符串切分回溯
- [0306. Additive Number](../0306-additive-number/README.md) — 字符串数字分割回溯
- [0473. Matchsticks to Square](../0473-matchsticks-to-square/README.md) — 装载 + 剪枝回溯
- 0140\. Word Break II (待补) — 字符串切分回溯
