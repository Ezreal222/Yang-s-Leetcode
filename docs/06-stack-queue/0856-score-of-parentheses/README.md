# 0856. Score of Parentheses / 括号的分数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Stack, String · 栈, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/score-of-parentheses/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: A balanced parentheses string has a score under these rules: `"()"` = 1, `AB` = `A + B`, `(A)` = `2 * score(A)`. Return the score of `s`.

**中文**: 一个合法括号串的分数定义: `"()"` 得 1 分, 拼接 `AB` 得分相加, 嵌套 `(A)` 得分乘 2. 求 `s` 的分数.

## 思路

### Core idea

**栈每一层维护"当前嵌套深度的累加分数"**:

- 初始 push 0 — 虚拟"外层" sentinel.
- 遇 `(`: push 0, 开启一层新累加器.
- 遇 `)`: 弹出内层分数 `v`, 弹出外层累加 `w`, 把 `w + max(2*v, 1)` push 回去.
  - `v == 0` (空括号 `()`): 内层无嵌套, 这对 `()` 贡献 1 分.
  - `v > 0`: 内层有分数, 包一层就乘 2.
- 扫完后, 栈顶就是总分.

### Key Insights

1. **`max(2*v, 1)` 一行处理两种规则 / Branchless score-combine**

    题目的规则是 `()` = 1 和 `(A)` = 2 * score(A). 它们其实是同一条公式 `max(2*v, 1)`:

    - `v = 0` (空括号): `max(0, 1) = 1`. ✅
    - `v > 0` (含内容): `max(2v, 1) = 2v`. ✅

    不需要 `if (v == 0) push 1 else push 2*v;` 分支. 这种"边界 case 跟主流公式合并" 的小优化是面试加分项.

2. **栈每层 = 当前嵌套深度的累加器 / Per-level accumulator**

    栈顶永远是"当前正在累加的最内层"的分数. 看到 `)` 就是这一层关闭, 把它的总分通过 `max(2v, 1)` 折叠回上一层. 这是处理任何**嵌套结构需要分层聚合**的通用模板.

    > 同款思路: HTML/XML 解析的标签栈、JSON 反序列化的对象栈、表达式求值的运算符栈.

3. **初始 sentinel push 0 必备 / Pre-push avoids special case**

    没有这个 sentinel, 第一个 `)` 弹两次会崩 (栈里只有一个值). Push 0 后, 即使整串只有一对 `()`, 最后栈里也是 `[0 + 1] = [1]`, 顺利返回 1.

    同款 sentinel 思想: [0150 Reverse Polish](../0150-evaluate-reverse-polish-notation/README.md) 不用 (直接两两 pop), 但 [0071 Simplify Path](../0071-simplify-path/README.md) 用 `/` 当 sentinel 类似.

4. **跟其它括号 stack 题对照 / Stack-of-X family**

    | 题 | stack 存什么 | 操作 |
    |---|---|---|
    | [0020](../0020-valid-parentheses/README.md) Valid Parens | char (`(` `[` `{`) | 配对判 |
    | [1249](../1249-minimum-remove-to-make-valid-parentheses/README.md) Min Remove | index | 后处理删 |
    | **0856 (本题)** | **int (每层累加分)** | **fold** |
    | 0032 Longest Valid Parens (待补) | index | 跨多 frame 算长度 |

    同一根 stack 模板, 存的元素和折叠规则换换, 解决一大类括号题.

5. **进阶: depth + 2^depth 一遍扫 / O(1) space alternative**

    维护 `depth` 计数器, 每见 `()` (即 `(` 紧跟 `)`) 就累加 `1 << (depth - 1)`. 不需要 stack, O(1) 额外空间. 但栈版的代码意图更直接, 推荐.

### 一句话总结

**栈每层一个累加器, `(` push 0, `)` 弹两个 fold 回 `w + max(2v, 1)`. 初始 push 0 sentinel.**

### 图解

`s = "(()(()))"` 的栈演变:

```
初始:      [0]
'(':       [0, 0]
'(':       [0, 0, 0]
')' v=0:   [0, 0 + max(0,1)] = [0, 1]
'(':       [0, 1, 0]
'(':       [0, 1, 0, 0]
')' v=0:   [0, 1, 0, 1]
')' v=1:   [0, 1, 0 + 2*1] = [0, 1, 2]
')' v=2:   [0, 1 + 2*2] = [0, 5]
')' v=5:   [0 + 2*5] = [10]
返回 10.
```

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int scoreOfParentheses(string s) {
            stack<int> stk;
            stk.push(0);                                 // sentinel: 虚拟外层
            for (char c : s) {
                if (c == '(') {
                    stk.push(0);                         // 开启新层累加器
                } else {
                    int v = stk.top(); stk.pop();        // 内层分
                    int w = stk.top(); stk.pop();        // 外层已累加
                    stk.push(w + max(2 * v, 1));         // 边界跟主公式合并
                }
            }
            return stk.top();
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def scoreOfParentheses(self, s: str) -> int:
            stk = [0]                                    # sentinel: 虚拟外层
            for c in s:
                if c == '(':
                    stk.append(0)
                else:
                    v = stk.pop()
                    w = stk.pop()
                    stk.append(w + max(2 * v, 1))        # max(2v, 1) 合并两种规则
            return stk[0]                                # 等价 stk.top(); 此时 stk 长度为 1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number}
     */
    var scoreOfParentheses = function(s) {
        const stk = [0];                                 // sentinel
        for (const c of s) {
            if (c === '(') {
                stk.push(0);
            } else {
                const v = stk.pop();
                const w = stk.pop();
                stk.push(w + Math.max(2 * v, 1));        // JS Math.max, 同 C++ std::max
            }
        }
        return stk[0];
    };
    ```

## Complexity

- **Time**: O(n) — 每字符一次 push/pop.
- **Space**: O(depth) 栈峰值 = 最大嵌套深度.

## 易错点

- **`max(2*v, 1)` 不是 `2*v + 1`**: 空括号 = 1 跟非空 = 2v 是**互斥的**, 不叠加. 容易把"边界 + 主公式" 写成加法.
- **必须初始 push 0**: 否则第一个 `)` 时栈只有 1 个元素, 弹两次崩. 这个 sentinel 比写 `if (stk.size() == 1)` 特判优雅得多.

## 相关题目

- [0020. Valid Parentheses](../0020-valid-parentheses/README.md) — 同款括号栈基础, 存字符判合法
- [1249. Minimum Remove to Make Valid Parentheses](../1249-minimum-remove-to-make-valid-parentheses/README.md) — 同款栈, 存 index 后处理
- [0150. Evaluate Reverse Polish Notation](../0150-evaluate-reverse-polish-notation/README.md) — 同款 fold pattern, 表达式求值
- [0155. Min Stack](../0155-min-stack/README.md) — 栈每个 frame 携带额外状态 (min)
- 0032\. Longest Valid Parentheses (待补) — 栈存 index, 求最长合法子串
- 1614\. Maximum Nesting Depth of the Parentheses (待补) — 只算深度, 不算分数; 计数器版
