# 0739. Daily Temperatures / 每日温度

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Array · 单调栈, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/daily-temperatures/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: 给每日温度 `temperatures`. 对每天 `i`, 找出**几天后** 才会出现比 `temperatures[i]` 更高的温度. 之后无更高则返 `0`.

**中文**: 对每个位置, 找下一个比自己温度高的位置到自己的距离, 没有就 0.

## Key Insights

1. **🔑 "下一个更大元素" 模板 — 单调栈最基础题 / Next Greater Element template**

    单调栈最纯粹的应用. 模板:

    - **维护单调递减栈**, 栈底大栈顶小.
    - 来一个 `temperatures[i]`, 把所有"小于 `temperatures[i]`" 的栈顶弹出 — 它们的"下一个更大" 就是 `i`.
    - 弹完把 `i` 压栈.

    > 看到"对每个元素找右边/左边第一个更大/更小" → 立刻反应单调栈, O(n).

2. **🔑 栈里存"索引" 不是"值" / Stack holds indices, not values**

    你笔记里强调的点. 存索引的两个好处:

    - **方便算距离**: `res[stk.top()] = i - stk.top()` 直接是天数差.
    - **方便取值**: 比较要 `temperatures[i] > temperatures[stk.top()]` — **两端都解引用**.

    > 新手最容易把"值" 跟"索引" 混了. **栈里存什么, 比较和使用时都要用对方式**.

3. **🔑 ⚠ 循环必须从 `i = 0` 开始 (或先 push 0) / Include index 0**

    必须让索引 0 进入栈, 否则它的答案永远是 0 (即使后面有更高温度). 两种正确写法:

    ```cpp
    // 写法 A (推荐): 从 i = 0 开始
    for (int i = 0; i < n; i++) { ...; stk.push(i); }

    // 写法 B: 先 push 0, 再从 i = 1 开始
    stk.push(0);
    for (int i = 1; i < n; i++) { ... }
    ```

    > **经典 bug**: 写"`for i = 1; ...; i++`" 但没 push 0 — `res[0]` 永远是 0. 例: `[73, 74]` 会得 `[0, 0]` 而非 `[1, 0]`. 写法 A 更稳, 不容易漏 — **推荐**.

4. **递减栈的不变量 / Decreasing-stack invariant**

    栈中索引对应的温度**严格递减**. 来一个 `temperatures[i]`:

    - 若大于栈顶温度 → 栈顶找到答案, 弹出, 继续比较.
    - 若小于等于栈顶温度 → 自己进栈 (维持递减).

    最终栈里剩下的索引都"没有更大的后续" → `res = 0` (默认值, 不用动).

5. **复杂度 O(n) 摊销 / Amortized O(n)**

    每个索引最多入栈一次, 出栈一次. 总操作 O(n). **单调栈所有题的复杂度论证都是这个套路**.

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I'm given a list of daily temperatures, and for each day I need to return **how many days I have to wait until a warmer temperature**. If no future day is warmer, return 0 for that day. Let me confirm a few things:"

- "The output is the **same length** as the input?" *(yes.)*
- "**Strictly greater**, not `>=`?" *(yes — strict.)*
- "**Temperature range** — could there be ties matter for overflow? just checking." *(usually 30–100, no overflow concern.)*
- "Any constraint on `n`? Just so I can pick the right complexity." *(usually up to 10⁵ — rules out O(n²).)*

### 2. Brainstorm approaches (1 min)

> "Two ways I'd think about this.
>
> **Approach 1 — brute force**: for each day, scan forward until I find a warmer day. That's O(n²). At n = 10⁵ that's 10¹⁰ operations — TLE.
>
> **Approach 2 — monotonic stack**. The key observation: when I see a temperature `t[i]`, I can look **backward** at all pending days that were **cooler** and instantly tell them 'your answer is today'. Once a day gets its answer, I never need it again. So I keep a **stack of days waiting for a warmer future** — the stack stays **monotonically decreasing** in temperature.
>
> Approach 2 is O(n) amortized — each index is pushed once and popped once. That's my pick."

### 3. Sketch the algorithm before coding (1 min)

> "The core loop:
>
> - Stack holds **indices** (not values) of days still waiting for an answer, in decreasing-temperature order.
> - For each new day `i`:
>   - While the stack isn't empty **and** `t[i] > t[stack.top()]`: pop the top — that day's answer is `i - top`.
>   - Push `i` onto the stack.
> - Any indices left in the stack at the end have no warmer future — their answer stays 0 (init default).
>
> Two design choices worth flagging out loud:
>
> 1. **Store indices, not temperatures.** Because I need to compute the *distance* `i - j` when a day gets resolved, I have to know its original index. Comparing values is easy: just `t[i]` vs `t[stack.top()]`.
> 2. **Start the loop at `i = 0`**, not `i = 1`. Common bug: if you start at 1 assuming 'nothing to compare yet', index 0 never enters the stack and its answer is stuck at 0 even if day 1 is warmer. Starting at 0 lets the invariant handle everything uniformly."

### 4. Code while narrating (2 min)

```cpp
vector<int> dailyTemperatures(vector<int>& t) {
    int n = t.size();
    vector<int> res(n, 0);         // default 0 means "no warmer future"
    stack<int> stk;                 // stack of indices, temperatures decreasing top→bottom
    for (int i = 0; i < n; i++) {
        while (!stk.empty() && t[i] > t[stk.top()]) {
            int j = stk.top(); stk.pop();
            res[j] = i - j;         // answer for day j is today
        }
        stk.push(i);
    }
    return res;
}
```

> "About 10 lines. The `while` inside `for` looks like it could be O(n²), but each index is popped at most once across the entire run — that's the amortization argument."

### 5. Trace an example (1 min)

> "Let me trace `t = [73, 74, 75, 71, 69, 72, 76, 73]`:
>
> | i | t[i] | Stack before | Pops (with answers) | Stack after |
> |---|---|---|---|---|
> | 0 | 73 | [] | — | [0] |
> | 1 | 74 | [0] | pop 0 → res[0]=1 | [1] |
> | 2 | 75 | [1] | pop 1 → res[1]=1 | [2] |
> | 3 | 71 | [2] | — | [2,3] |
> | 4 | 69 | [2,3] | — | [2,3,4] |
> | 5 | 72 | [2,3,4] | pop 4 → res[4]=1; pop 3 → res[3]=2 | [2,5] |
> | 6 | 76 | [2,5] | pop 5 → res[5]=1; pop 2 → res[2]=4 | [6] |
> | 7 | 73 | [6] | — | [6,7] |
>
> End of loop: stack [6, 7] — days 6 and 7 have no warmer future, so res[6] = res[7] = 0 by default.
>
> Result: `[1, 1, 4, 2, 1, 1, 0, 0]`. Matches the expected output."

### 6. Complexity (20s)

> "**Time O(n)** amortized — every index is pushed once and popped at most once, so the total work inside the `while` across the whole loop is bounded by n. **Space O(n)** worst case — if the input is strictly decreasing, everything stays on the stack."

### 7. Follow-ups + related (1 min)

> "A few natural extensions:
>
> 1. **Next Greater Element I & II** (0496, 0503) — same monotonic-stack template, just with a mapping table or a circular array (traverse `2n` for the circular case).
> 2. **Online Stock Span** (0901) — streaming version, stack holds `(price, span)` pairs so we can batch-collapse consecutive smaller prices.
> 3. **Largest Rectangle in Histogram** (0084) — same idea taken further: monotonic stack tells you, for each bar, how far you can extend left/right, then compute area.
>
> All of these share the same amortization argument: 'each element enters and leaves the stack once.' Once you internalize the 'next greater' template, the family opens up."

> "Any follow-ups you'd like me to code?"

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> dailyTemperatures(vector<int>& temperatures) {
            int n = temperatures.size();
            vector<int> res(n, 0);
            stack<int> stk;                                        // 栈存索引
            for (int i = 0; i < n; i++) {                          // ⚠ 从 0 开始, 别漏
                // 注意两端都解引用比较
                while (!stk.empty() && temperatures[i] > temperatures[stk.top()]) {
                    res[stk.top()] = i - stk.top();                // 距离 = 索引差
                    stk.pop();
                }
                stk.push(i);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def dailyTemperatures(self, temperatures: list[int]) -> list[int]:
            n = len(temperatures)
            res = [0] * n
            stk = []                                               # list 当栈, append/pop O(1)
            # enumerate 一次拿 (索引, 值), Pythonic
            for i, t in enumerate(temperatures):
                while stk and t > temperatures[stk[-1]]:
                    j = stk.pop()
                    res[j] = i - j
                stk.append(i)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} temperatures
     * @return {number[]}
     */
    var dailyTemperatures = function(temperatures) {
        const n = temperatures.length;
        const res = new Array(n).fill(0);
        const stk = [];
        for (let i = 0; i < n; i++) {
            while (stk.length && temperatures[i] > temperatures[stk[stk.length - 1]]) {
                const j = stk.pop();
                res[j] = i - j;
            }
            stk.push(i);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 每索引入栈出栈各一次.
- **Space**: O(n) — 栈最坏存全部.

## 相关题目

- [0496. Next Greater Element I](../0496-next-greater-element-i/README.md) — 同模板, 栈存值 + map 映射
- [0503. Next Greater Element II](../0503-next-greater-element-ii/README.md) — 循环数组, 扫 `2n` 遍处理环
- [0901. Online Stock Span](../0901-online-stock-span/README.md) — 流式数据 + 单调栈 (栈存 `(price, span)` 块)
- [0316. Remove Duplicate Letters](../0316-remove-duplicate-letters/README.md) — 单调栈 + 贪心字典序
- [1130. Minimum Cost Tree From Leaf Values (单调栈解法)](../1130-minimum-cost-tree-from-leaf-values/README.md) — 单调栈进阶应用
- [0084. Largest Rectangle in Histogram](../0084-largest-rectangle-in-histogram/README.md) — 单调栈经典进阶
- [0042. Trapping Rain Water](../0042-trapping-rain-water/README.md) — 单调栈或双指针 (经典 Hard)
