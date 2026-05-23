# 0316. Remove Duplicate Letters / 去除重复字母

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Monotonic Stack, Greedy, String, Hash Table · 单调栈, 贪心, 字符串, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/remove-duplicate-letters/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given string `s`, return the smallest **lexicographic** string that contains each letter of `s` **exactly once** (so it's a deduplicated subsequence of `s`).

**中文**: 给字符串 `s`, 返回**字典序最小**的字符串, 包含 `s` 里每种字母**恰好一次** (相当于去重后的最小字典序子序列).

## Key Insights

1. **核心: 单调栈 + 贪心 + lastPos lookahead / Monotonic stack + greedy + future check**

    维护一个 (大致) 递增的栈作为答案. 扫到每个字符 `c`:

    - **`c` 已在栈中** → 跳过 (每种字母只要一次).
    - 否则尝试把栈顶比 `c` 大的字符 **弹掉**, 让 `c` 往前坐 (字典序变小). 但只有当**栈顶字符在后面还会出现** (`lastPos[top] > i`), 才敢弹 — 不然弹了就永远没了.
    - 弹完后 push `c`, 标记 inStack.

    > 一句话: **大的让小的, 前提是大的"还回得来"**.

2. **lastPos 数组: 唯一的 lookahead 信息 / The one piece of future info we need**

    预处理 `lastPos[c] = c 最后出现的 index`. 一次 O(n) 扫.

    没这个数组就没法判断"能不能弹". 这是本题**唯一的预处理**, 也是把"O(n²) 暴力" 降到"O(n) 单调栈" 的关键.

3. **inStack 数组: O(1) 判 "已在栈中" / Membership check in O(1)**

    用 `vector<bool>(26)` 记录每个字母是否当前在栈里. 不用每次扫栈 → O(1) 判.

    push 时设 `true`, pop 时设 `false`. **配对的状态维护**, 千万别只 push 不维护 inStack — 会重复入栈.

4. **"单调"是大致的, 不严格 / "Monotonic" is loose, not strict**

    经典单调栈 (e.g. [0739 每日温度](待补)) 严格递增/递减. 本题的栈**允许"大字母留在前面"** — 因为如果后面没该字母了, 必须留下, 即使破坏递增.

    例: `s = "bcabc"` → 答案 `"abc"`. 扫到 b: stk=`b`. 扫 c: stk=`bc`. 扫 a: b/c 在后面都还有 → 全弹 → stk=`a`. 扫 b: 加 → stk=`ab`. 扫 c: 加 → stk=`abc`. **比 0739 多了"后面还有吗?" 这个条件**.

5. **贪心证明 (sketch) / Why greedy works**

    每个时刻栈代表"目前为止能做到的字典序最小前缀". 当扫到 `c` 时:

    - 若栈顶 `t > c` 且 `t` 后面还会出现 → 把 `t` 推迟 (从栈弹, 后面再加回来) 严格让前缀更小. 必做.
    - 若栈顶 `t > c` 但 `t` 后面没了 → 不能弹, 否则失去 `t` (题目要求每种字母都要有).
    - 若栈顶 `t < c` → 不动.

    每次决策都是"局部字典序最优", 加上"未来不损失任何字母", 累积起来就是全局字典序最小. **局部最优 ⇒ 全局最优**.

6. **跟其它单调栈题的家族关系 / Mono-stack family**

    | 题 | 栈维护什么 | 弹出条件 |
    |---|---|---|
    | **0316 (本题)** | 答案前缀 (类递增字母) | 栈顶 > 新字符 **且** 后面还有 |
    | 0739 每日温度 (待补) | 待解决的下标 | 栈顶温度 < 新温度 |
    | 0084 柱状图 (待补) | 递增高度 | 栈顶高度 > 新高度 |
    | 0042 接雨水 (待补) | 递减高度 | 栈顶 < 新高度 → 算积水 |
    | 0402 移掉 K 位数字 (待补) | 答案前缀 | 栈顶 > 新数字 + 还有删配额 |

    > **0316 vs 0402** 是孪生兄弟: 都是"栈维护答案 + 弹掉栈顶让结果变小". 差别只在弹出条件 (本题是 lastPos, 0402 是删除配额 k).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string removeDuplicateLetters(string s) {
            vector<int>  lastPos(26, 0);
            vector<bool> inStack(26, false);
            for (int i = 0; i < (int)s.size(); i++) {
                lastPos[s[i] - 'a'] = i;                           // 每个字母最后出现位置
            }
            string stk;                                            // 直接用 string 当栈, back()/pop_back() 即可
            for (int i = 0; i < (int)s.size(); i++) {
                char c = s[i];
                if (inStack[c - 'a']) continue;                    // 已在答案里, 跳过
                while (!stk.empty() && stk.back() > c
                       && lastPos[stk.back() - 'a'] > i) {         // 栈顶后面还会再来 → 安全弹
                    inStack[stk.back() - 'a'] = false;
                    stk.pop_back();
                }
                stk.push_back(c);
                inStack[c - 'a'] = true;
            }
            return stk;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def removeDuplicateLetters(self, s: str) -> str:
            # 一行字典推导: 同 key 重复写入, 留下最后位置 (跟 0763 同款 trick)
            last_pos = {c: i for i, c in enumerate(s)}
            stk = []
            in_stack = set()                                       # set 替代 bool 数组, O(1) 判 in
            for i, c in enumerate(s):
                if c in in_stack:
                    continue
                # while 弹栈: 栈顶比 c 大, 且栈顶后面还会出现
                while stk and stk[-1] > c and last_pos[stk[-1]] > i:
                    in_stack.remove(stk.pop())                     # pop 同时维护 in_stack
                stk.append(c)
                in_stack.add(c)
            return ''.join(stk)                                    # list of char → string
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string}
     */
    var removeDuplicateLetters = function(s) {
        // plain object 当 dict; 同 key 重复写入, 留最后位置
        const lastPos = {};
        for (let i = 0; i < s.length; i++) lastPos[s[i]] = i;
        const stk = [];
        // Set 替代 bool 数组; .has() / .add() / .delete() 都是 O(1)
        const inStack = new Set();
        for (let i = 0; i < s.length; i++) {
            const c = s[i];
            if (inStack.has(c)) continue;
            while (stk.length && stk[stk.length - 1] > c
                   && lastPos[stk[stk.length - 1]] > i) {
                inStack.delete(stk.pop());
            }
            stk.push(c);
            inStack.add(c);
        }
        return stk.join('');                                       // array → string
    };
    ```

## Complexity

- **Time**: O(n) — 每个字符最多入栈出栈一次.
- **Space**: O(σ), σ = 字符集大小 (26).

## 相关题目

- 0402\. Remove K Digits (待补) — 孪生题, 删 k 位让数字最小; 弹出条件是"还有删配额"
- 0739\. Daily Temperatures (待补) — 单调栈入门, 求"下一个更大"
- 0496\. Next Greater Element I (待补) — 单调栈基础
- 0503\. Next Greater Element II (待补) — 循环数组版
- 0084\. Largest Rectangle in Histogram (待补) — 单调栈进阶
- 0042\. Trapping Rain Water (待补) — 单调栈经典
- 1081\. Smallest Subsequence of Distinct Characters (待补) — 本题原题 (LC 重复)
- [0763. Partition Labels](../../09-greedy/0763-partition-labels/README.md) — 同款 lastPos 预处理 trick
