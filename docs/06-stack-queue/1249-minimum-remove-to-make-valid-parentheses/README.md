# 1249. Minimum Remove to Make Valid Parentheses / 移除无效的括号

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Stack, String · 栈, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a string `s` with `'('`, `')'`, and letters, remove the **minimum** number of parentheses so the result is valid. Return **any** valid result.

**中文**: 给一个含 `'('`, `')'` 和字母的字符串 `s`, 删最少括号让结果合法. 返回**任意一个**合法结果.

## 思路

### Core idea

**两遍扫 + stack 存未匹配 `(` 的 index**:

1. **Pass 1**: 扫一遍, `(` 入栈 (存 index), 遇 `)` 时若栈非空就 pop (匹配上一个 `(`), 否则标记当前 `)` 为"要删".
2. **Pass 1 收尾**: 栈里剩的 index 就是"未匹配的 `("`, 全标记为"要删".
3. **Pass 2**: 线性扫, 跳过标记位输出.

跟 [0301](../../08-backtracking/0301-remove-invalid-parentheses/README.md) 区别: 那题要"所有最短方案" → 回溯; 这题只要"任意一个" → 贪心 + stack 一次过.

### Key Insights

1. **V1 → V2 的复杂度跳变: O(n²) → O(n) / The big lesson**

    Yang 的 V1 用 `vector<int> removeIndex` 存要删的下标. 第二遍输出时:
    ```cpp
    if (find(removeIndex.begin(), removeIndex.end(), i) == removeIndex.end()) res += s[i];
    ```
    **`find()` 在 vector 里是 O(n)**, 套在 for 循环里就是 **O(n²)**. 数据稍大就慢.

    V2 用 `vector<bool> remove(n, false)`. 查询 `remove[i]` 是 **O(1)**, 第二遍总 O(n).

    **要记的红线**: 任何"循环里反复查某下标/某值是否在集合里" 的场景, **绝不能用 `find` 在 vector / list 里搜**. 改 `unordered_set` / `unordered_map` / `vector<bool>` 一律 O(1).

2. **Stack 存 index, 不是字符 / Index in stack, not char**

    [0020 Valid Parentheses](../0020-valid-parentheses/README.md) 用 stack 存字符判合法. 这里升级一档: 存**位置**, 因为我们最后需要"删哪些位置", 不只是"是否合法". 同样的 stack 模板, 拓宽了用途.

3. **Mark and sweep 模式 / Two-pass: mark then filter**

    Pass 1 用 bool[] / set 决定每个位置去留, Pass 2 线性扫输出. 这是任何"按条件过滤位置" 题的首选骨架. 同款应用: 字符串清洗、数组某条件删除、扫雷标记等.

4. **V1 的 preemptive push + pop 思路也对, 但绕 / The reverse-marking variant**

    Yang 的 V1 反过来: **每个 `(` 都先标记为"要删", 匹配上再撤销 (pop_back)**. 等价于"默认删, 配对后赦免". 算法是对的 (我 traced 几个 case 都通过), 但:

    - pop_back 依赖"removeIndex 末尾就是最近未匹配 `(`" 这个隐式 invariant, 不容易一眼读懂.
    - 跟 V2 的"默认留, 不匹配才删" 比反着想, 调试时更费脑.

    **结论**: 两种思路都对, 但**正向标记 (V2) 是默认写法**. 反向思路属于"知道可以但日常不用".

5. **跟 [0301 Remove Invalid Parentheses](../../08-backtracking/0301-remove-invalid-parentheses/README.md) 的对照 / Different problem despite name similarity**

    | 题 | 输出 | 算法 |
    |---|---|---|
    | 0301 | **所有**最短方案 | 回溯 + 同层去重 |
    | **1249 (本题)** | **任意一个**方案 | 贪心 + stack 一次过 |

    需求换了 ("一个" vs "全部"), 算法范式就完全变. 名字相似但是两类题.

### 一句话总结

**Stack 存未匹配 `(` 的 index, 遇 `)` 配则 pop 否则标删. 最后剩在栈里的 `(` 也标删. bool[] 标记, 第二遍过滤输出. O(n).**

### 图解

`s = "lee(t(c)o)de)"`:

```
位置: 0 1 2 3 4 5 6 7 8 9 10 11 12
字符: l e e ( t ( c ) o )  d  e  )

Pass 1:
  i=3 '(': stack=[3]
  i=5 '(': stack=[3, 5]
  i=7 ')': stack 非空, pop → stack=[3]
  i=9 ')': stack 非空, pop → stack=[]
  i=12 ')': stack 空 → remove[12] = true

Pass 1 收尾: stack 空, 无需再标记.
Pass 2: 跳过 remove[12], 输出 "lee(t(c)o)de"
```

## Solution

=== "C++"
    === "推荐: V2 stack + bool[]"
        ```cpp
        class Solution {
        public:
            string minRemoveToMakeValid(string s) {
                stack<int> stk;                              // 存未匹配 '(' 的 index
                vector<bool> remove(s.size(), false);
                for (int i = 0; i < (int)s.size(); i++) {
                    if (s[i] == '(') {
                        stk.push(i);
                    } else if (s[i] == ')') {
                        if (!stk.empty()) stk.pop();         // 匹配掉一个 '('
                        else remove[i] = true;               // 多余 ')' 标删
                    }
                }
                while (!stk.empty()) {                       // 剩下的 '(' 都没匹配, 标删
                    remove[stk.top()] = true;
                    stk.pop();
                }
                string res;
                for (int i = 0; i < (int)s.size(); i++)
                    if (!remove[i]) res += s[i];             // O(1) 查 → 总 O(n)
                return res;
            }
        };
        ```

    === "V1 (Yang 第一版, O(n²) 慢)"
        ```cpp
        // 思路: 每个 '(' 先标"要删", 匹配上再 pop_back 撤销.
        // bug 没有, 但 find() 在 vector 里是 O(n) → 总 O(n²).
        class Solution {
        public:
            string minRemoveToMakeValid(string s) {
                string res;
                vector<int> removeIndex;
                int count = 0;
                for (int i = 0; i < (int)s.size(); i++) {
                    if (s[i] == '(') {
                        count++;
                        removeIndex.push_back(i);
                    } else if (s[i] == ')') {
                        if (--count < 0) {
                            removeIndex.push_back(i);
                            count = 0;
                        } else {
                            removeIndex.pop_back();          // 撤销最近 '(' 的删标记
                        }
                    }
                }
                for (int i = 0; i < (int)s.size(); i++)
                    if (find(removeIndex.begin(), removeIndex.end(), i) == removeIndex.end())
                        res += s[i];                          // ← O(n) per call, 总 O(n²)
                return res;
            }
        };
        ```

=== "Python"
    ```python
    class Solution:
        def minRemoveToMakeValid(self, s: str) -> str:
            stk = []                                          # list 当 stack 用
            remove = [False] * len(s)
            for i, c in enumerate(s):                         # enumerate 同时拿 (index, char)
                if c == '(':
                    stk.append(i)
                elif c == ')':
                    if stk:
                        stk.pop()
                    else:
                        remove[i] = True
            for i in stk:                                     # 剩下未匹配的 '('
                remove[i] = True
            # 一行过滤: 等价 C++ 第二遍 for, ''.join 拼接 O(n)
            return ''.join(c for i, c in enumerate(s) if not remove[i])
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string}
     */
    var minRemoveToMakeValid = function(s) {
        const stk = [];
        const remove = new Array(s.length).fill(false);
        for (let i = 0; i < s.length; i++) {
            if (s[i] === '(') {
                stk.push(i);
            } else if (s[i] === ')') {
                if (stk.length) stk.pop();
                else remove[i] = true;
            }
        }
        // 一次性把剩下的 '(' index 都标删. for...of 直接迭代数组
        for (const i of stk) remove[i] = true;
        // Array.from + filter + join: 把字符串拆 char 数组, 跳过标删位, 再拼回
        return Array.from(s).filter((_, i) => !remove[i]).join('');
    };
    ```

## Complexity

| 版本 | Pass 1 | Pass 2 | 总 |
|---|---|---|---|
| V1 (find in vector) | O(n) | **O(n²)** | **O(n²)** |
| V2 (stack + bool[]) | O(n) | O(n) | **O(n)** |

Space O(n) — bool[] + stack 峰值.

## 易错点

- **不要在循环里 `find()` vector / list**: 这是 O(n²) 的 # 1 来源. 改 `unordered_set` / `vector<bool>` 让查询变 O(1).
- **stack 存 index 不是字符**: 同 [0020](../0020-valid-parentheses/README.md) 模板的升级版. 需要定位才能"删那些位置", 所以记 index.

## 相关题目

- [0020. Valid Parentheses](../0020-valid-parentheses/README.md) — 同款 stack 判括号, 但只问合法 (本题升级到"修哪些")
- [0301. Remove Invalid Parentheses](../../08-backtracking/0301-remove-invalid-parentheses/README.md) — 返回**所有**最短方案, 回溯版的同题
- [0150. Evaluate Reverse Polish Notation](../0150-evaluate-reverse-polish-notation/README.md) — 同款 stack 处理表达式
- [0071. Simplify Path](../0071-simplify-path/README.md) — 同款 stack + 后处理字符串
- 0022\. Generate Parentheses (待补) — 反向: 生成所有 n 对合法括号, 回溯 + (left, right) 计数
- 0032\. Longest Valid Parentheses (待补) — 求最长合法子串, stack 或 DP 两种解
