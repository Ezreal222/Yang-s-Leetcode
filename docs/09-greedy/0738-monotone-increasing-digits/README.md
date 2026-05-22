# 0738. Monotone Increasing Digits / 单调递增的数字

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Math, String · 贪心, 数学, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/monotone-increasing-digits/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given integer `n`, return the largest integer `≤ n` whose digits are **monotonically non-decreasing** (each digit ≤ next).

**中文**: 给整数 `n`, 返回 `≤ n` 中"每位数字单调非递减" 的最大整数.

## Key Insights

1. **核心贪心: 找到第一个违规位, 前一位 -1, 后面全填 9 / Decrement & fill-9**

    扫到 `s[i-1] > s[i]` (违规) — 想让数字尽量大又满足非递减:

    - **`s[i-1]` 减 1** (因为不能让前一位 > 后一位, 减 1 是最小让步).
    - **`s[i]..end` 全填 9** (后缀填到最大, 数字才尽量大).

    例: `n = 332` → 扫到 `3 > 2` → `3 → 2`, `2 → 9` → `329`. 但 `329` 还违规 (`3 > 2`)! 所以必须**从右往左**扫.

2. **必须从右往左扫 / Must scan right-to-left**

    从左往右扫的反例: `n = 100` → 看到 `1 > 0` → `0, 9, 0` → 还有 `9 > 0` 违规, 错. 

    从右往左: `100` → 末尾 `0 < 0` 不违规, 倒数第二 `0 < 1` 违规 → `1 → 0`, `00 → 99` → `099` → `99`. 对.

    > **为什么从右往左对**: 减 1 操作只**影响左边**(把前一位变小), 这个新变化我们下一轮 (`i--`) 还会处理. 从左往右减 1 反而会留下右边未处理的旧违规, 出错.

3. **v1 vs v2 (flag 优化) / Flag = single fill-9 pass**

    - **v1 (Yang 的初版)**: 扫到违规就**当场**把 `i..end` 全填 9. 一个数字可能被反复填 (`332` → `329` → `299`, 中间 `2 → 9` 然后又被覆盖? 实际不会, 因为下一轮 i 更小, 后缀不会扩展, 但**多次 9 填充**仍然冗余).
    - **v2 (推荐)**: 维护 `flag = len` (默认无违规). 每次违规只更新 `flag = i`. **扫完后** 再一次性 `s[flag..end] = '9'`. 一遍扫 + 一遍填 → O(n) 严格.

    > 一句话: **v1 边扫边填, v2 记位置统一填**. 后者代码更干净, 性能更稳.

4. **为什么是 `flag = i` 不是 `flag = i - 1` / Where to start filling 9s**

    `s[i-1]` 被减 1, 它本身参与"前面位 + 1 的非递减序列", 不能填 9. **从 `i` 起填** 才对.

    特别注意: 减 1 后 `s[i-1]` 可能变成 `0` (如 `100` 的中间 `0 → ... → 0`). 这没问题 — 整数 `099` stoi 后是 `99`, **前导 0 自动消失**.

5. **不要用 long / 整数转字符串再转回 / Stay in string land**

    `n ≤ 1e9`, `int` 够用. 但**直接在 int 上做位运算太绕** (要 `%10` / `/10` 拆位还要重组). 转 string 后用下标随机访问 + 直接修改, 代码短 5 倍.

## Solution

=== "C++"
    === "v2 推荐: flag 一遍填"
        ```cpp
        class Solution {
        public:
            int monotoneIncreasingDigits(int n) {
                string s = to_string(n);
                int len = s.size();
                int flag = len;                                    // 从 flag 起的后缀都填 9; len 表示无违规
                for (int i = len - 1; i >= 1; i--) {
                    if (s[i] < s[i - 1]) {
                        s[i - 1]--;
                        flag = i;                                  // 只记最左的违规位置, 不当场填
                    }
                }
                for (int i = flag; i < len; i++) s[i] = '9';       // 一次填到底
                return stoi(s);
            }
        };
        ```

    === "v1 (边扫边填)"
        ```cpp
        // 思路也对, 但每次违规都重新填一遍 9, 冗余
        class Solution {
        public:
            int monotoneIncreasingDigits(int n) {
                string s = to_string(n);
                for (int i = s.size() - 1; i >= 1; i--) {
                    if (s[i] < s[i - 1]) {
                        s[i - 1] -= 1;
                        for (int j = i; j < (int)s.size(); j++) s[j] = '9';
                    }
                }
                return stoi(s);
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def monotoneIncreasingDigits(self, n: int) -> int:
            # list(str(n)): str 在 Python 不可变, 转成 list 才能就地改; 等价 C++ 的 string
            s = list(str(n))
            flag = len(s)                                          # 默认无违规
            for i in range(len(s) - 1, 0, -1):                     # range(start, stop, step): 包含 len-1, 到 1 止 (不含 0)
                if s[i] < s[i - 1]:
                    s[i - 1] = str(int(s[i - 1]) - 1)              # 数字字符 -1, 要先转 int 再转回 str
                    flag = i
            for i in range(flag, len(s)):
                s[i] = '9'
            return int(''.join(s))                                 # ''.join: list 拼回字符串, 等价 C++ 的 string 隐式拼
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {number}
     */
    var monotoneIncreasingDigits = function(n) {
        // JS string 也不可变, 转 array 才能就地改
        const s = String(n).split('');
        let flag = s.length;
        for (let i = s.length - 1; i >= 1; i--) {
            if (s[i] < s[i - 1]) {                                 // string 比较按字符 ASCII, '0'<'9' 顺序刚好等价数字比
                s[i - 1] = String(Number(s[i - 1]) - 1);           // 数字字符 -1: 先 Number 再 String
                flag = i;
            }
        }
        for (let i = flag; i < s.length; i++) s[i] = '9';
        return Number(s.join(''));                                 // join('') 拼回, Number 解析 (前导 0 自动去掉)
    };
    ```

## Complexity

- **Time**: O(d) where d = digits of n (≈ log₁₀ n). 两次扫.
- **Space**: O(d) — string buffer.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同款"扫一遍维护状态" 贪心入门
- [0860. Lemonade Change](../0860-lemonade-change/README.md) — 同款"边扫边做局部最优决策"
- 0670\. Maximum Swap (待补) — 类似"数位贪心找交换点" 的姊妹题
- 0402\. Remove K Digits (待补) — 数位贪心 + 单调栈, 进阶
- 0321\. Create Maximum Number (待补) — 数位贪心 + 多源合并, 更进阶
