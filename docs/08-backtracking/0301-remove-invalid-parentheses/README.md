# 0301. Remove Invalid Parentheses / 删除无效的括号

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Backtracking, String, BFS · 回溯, 字符串, 广度优先搜索
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-invalid-parentheses/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a string `s` with `'('`, `')'`, and letters, remove the **minimum number** of parentheses to make it valid. Return **all unique** valid results.

**中文**: 给一个含 `'('`, `')'` 和字母的字符串 `s`, 删**最少**括号使之合法. 返回**所有不同**的合法结果.

## 思路

### Core idea

两步走:

1. **预扫一次** 算出**最少要删多少个 `(` 和 `)`** (`removeLeft`, `removeRight`).
2. **回溯**: for 循环每个位置, 决定是否"删这一位". 用 `(start, removeLeft, removeRight)` 推进, **同层相邻同字符跳重**. 当 `removeLeft == 0 && removeRight == 0` 时 `isValid` 检查, 通过就收.

### Key Insights

1. **预计算 = 把"最少删多少" 写死, 回溯只生成"恰好删够" 的结果 / Pre-scan locks the target**

    一次线性扫描: 维护 `removeLeft` 计数, 遇 `(` 加 1, 遇 `)` 时若有未匹配的 `(` 就 `removeLeft--`, 否则 `removeRight++`. 扫完后 `removeLeft` 是"剩下匹配不上的 `("`, `removeRight` 同理.

    这两个数字就是**最少必须删的下限**. 回溯只生成"删恰好这么多" 的串, 自动保证"最短". 不预计算就要在每个长度上都试, 复杂度爆炸.

2. **同层去重: 相邻同字符跳 / Adjacent same-char skip**

    `if (i > start && s[i] == s[i-1]) continue;` — 删两个相同字符 (例如 `((` 删第一个 vs 第二个) 产生相同的子串. 同 [0040](../0040-combination-sum-ii/README.md) 的"树枝同值留, 树层同值跳" 思路, 这里"树层" 就是当前 backtrack 调用的 for 循环.

3. **`recurse(i)` 不是 `recurse(i+1)` / Same-position re-entry after deletion**

    删了 s[i] 之后, 新字符串里**当前位置仍然是 i** (后面的字符前移). 所以下层 `start = i`. 同 [0039 Combination Sum 的 `recurse(i)`](../0039-combination-sum/README.md) 思路 — 允许在同一逻辑位置继续操作.

    如果传 `i + 1` 就丢掉了"在 i 位置后续删的可能", 答案不全.

4. **isValid 一次扫 / Single-pass balance check**

    标准计数器: `count` 遇 `(` 加 1 遇 `)` 减 1, 中途 < 0 立即 false, 结尾必须 == 0. 同 [0020 Valid Parentheses](../../06-stack-queue/0020-valid-parentheses/README.md) 但只考虑圆括号.

5. **BFS 是教科书替代 / BFS variant**

    从原串开始, 每轮"删一个字符" 产生新串, 入队. 第一次发现合法串的那一层就是答案层 — 那层所有合法串都是答案. 同时间复杂度, 但状态空间用 `unordered_set` 去重显式得多, 代码更长. 回溯版更紧凑.

6. **跟 [0051 / 0037 的对比 / The "Hard 回溯" family pattern**

    | 题 | 状态 | 输出 |
    |---|---|---|
    | [0051](../0051-n-queens/README.md) | 棋盘 + row | 收所有解 |
    | [0037](../0037-sudoku-solver/README.md) | 棋盘 + 找下一空格 | bool 短路 |
    | [0052](../0052-n-queens-ii/README.md) | 棋盘 + row | int 累加 |
    | **0301 (本题)** | 字符串 + (start, rmL, rmR) | 收所有最短解 |

    都共享"显式回溯 + 局部约束剪枝" 骨架, 状态空间和输出模式各异.

### 一句话总结

**预扫定下"最少删 removeLeft / removeRight"; 回溯 for 每位试删, 相邻同字符跳; 删完恰好够时 isValid 验证 + 收果实.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> res;
        bool isValid(const string& s) {
            int count = 0;
            for (char c : s) {
                if (c == '(') count++;
                else if (c == ')' && --count < 0) return false;
            }
            return count == 0;
        }
        void backtrack(const string& s, int start, int removeLeft, int removeRight) {
            if (removeLeft == 0 && removeRight == 0) {
                if (isValid(s)) res.push_back(s);
                return;
            }
            for (int i = start; i < (int)s.size(); i++) {
                if (i > start && s[i] == s[i - 1]) continue;                          // 同层去重
                if (s[i] == '(' && removeLeft > 0)
                    backtrack(s.substr(0, i) + s.substr(i + 1), i, removeLeft - 1, removeRight);
                if (s[i] == ')' && removeRight > 0)
                    backtrack(s.substr(0, i) + s.substr(i + 1), i, removeLeft, removeRight - 1);
            }
        }
        vector<string> removeInvalidParentheses(string s) {
            int removeLeft = 0, removeRight = 0;
            for (char c : s) {                                                        // 预扫: 算最少删多少
                if (c == '(') removeLeft++;
                else if (c == ')') {
                    if (removeLeft > 0) removeLeft--;
                    else removeRight++;
                }
            }
            backtrack(s, 0, removeLeft, removeRight);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def removeInvalidParentheses(self, s: str) -> list[str]:
            def is_valid(t: str) -> bool:
                count = 0
                for c in t:
                    if c == '(':
                        count += 1
                    elif c == ')':
                        count -= 1
                        if count < 0:
                            return False
                return count == 0
            # 预扫
            remove_left = remove_right = 0
            for c in s:
                if c == '(':
                    remove_left += 1
                elif c == ')':
                    if remove_left > 0:
                        remove_left -= 1
                    else:
                        remove_right += 1
            res = []
            def backtrack(t: str, start: int, rl: int, rr: int):
                if rl == 0 and rr == 0:
                    if is_valid(t):
                        res.append(t)
                    return
                for i in range(start, len(t)):
                    if i > start and t[i] == t[i - 1]:
                        continue                                          # 同层去重
                    # 字符串切片拼接 = C++ substr + concat
                    if t[i] == '(' and rl > 0:
                        backtrack(t[:i] + t[i+1:], i, rl - 1, rr)
                    if t[i] == ')' and rr > 0:
                        backtrack(t[:i] + t[i+1:], i, rl, rr - 1)
            backtrack(s, 0, remove_left, remove_right)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string[]}
     */
    var removeInvalidParentheses = function(s) {
        const isValid = (t) => {
            let count = 0;
            for (const c of t) {
                if (c === '(') count++;
                else if (c === ')' && --count < 0) return false;
            }
            return count === 0;
        };
        let removeLeft = 0, removeRight = 0;
        for (const c of s) {
            if (c === '(') removeLeft++;
            else if (c === ')') {
                if (removeLeft > 0) removeLeft--;
                else removeRight++;
            }
        }
        const res = [];
        const backtrack = (t, start, rl, rr) => {
            if (rl === 0 && rr === 0) {
                if (isValid(t)) res.push(t);
                return;
            }
            for (let i = start; i < t.length; i++) {
                if (i > start && t[i] === t[i - 1]) continue;             // 同层去重
                // t.slice(0, i) + t.slice(i + 1) = C++ substr 拼接, 等价 Python t[:i] + t[i+1:]
                if (t[i] === '(' && rl > 0)
                    backtrack(t.slice(0, i) + t.slice(i + 1), i, rl - 1, rr);
                if (t[i] === ')' && rr > 0)
                    backtrack(t.slice(0, i) + t.slice(i + 1), i, rl, rr - 1);
            }
        };
        backtrack(s, 0, removeLeft, removeRight);
        return res;
    };
    ```

## Complexity

- **Time**: O(C(n, rl+rr) × n) — 选 `rl + rr` 个位置删, 每条 path 拷贝/验证 O(n). 最坏 O(2^n × n).
- **Space**: O(n) recursion + 每层 substring 拷贝 O(n).

## 易错点

- **必须预扫定 removeLeft/removeRight**: 不限定数量, 回溯会生成大量过短/过长非法串, 全 isValid 一遍仍要 O(2^n) — 而且没法保证"最短" 答案.
- **`recurse(i)` 不是 `recurse(i+1)`**: 删了 s[i] 后, 新串当前位置还是 i. 写 i+1 会丢答案. 同 0039 思路.

## 相关题目

- [0040. Combination Sum II](../0040-combination-sum-ii/README.md) — 同款"相邻同元素同层跳"去重
- [0039. Combination Sum](../0039-combination-sum/README.md) — 同款 `recurse(i)` 同位置再进入思路
- [0051. N-Queens](../0051-n-queens/README.md) / [0037. Sudoku Solver](../0037-sudoku-solver/README.md) — 同款 Hard 回溯模板, 不同状态
- [0020. Valid Parentheses](../../06-stack-queue/0020-valid-parentheses/README.md) — isValid 的标准实现 (但完整版含所有括号种类)
- 0022\. Generate Parentheses (待补) — 构造合法括号, 同款 (left, right) 计数回溯
- 0032\. Longest Valid Parentheses (待补) — 求最长合法括号子串, DP 解 (不是回溯)
