# 0488. Zuma Game / 祖玛游戏

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Backtracking, Memoization, String Simulation, DFS · 回溯, 记忆化, 字符串模拟, DFS
    - **Link**: [LeetCode](https://leetcode.com/problems/zuma-game/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min balls to clear the board (chain-pop on 3+ in a row)** → **DFS + memo on `(board, hand)`**; try each hand ball at each valid position; `shrink(s)` recursively pops runs ≥ 3; prune with **sort(hand) + skip dup letters** and **skip positions that duplicate previous attempts**.
>
> **中文**: **最少投几个球清空 board (3+ 同色连消)** → **DFS + `(board, hand)` 记忆化**; 枚举每个手球每个插入位; `shrink` 递归消除 ≥3 连段; 剪枝: **sort 手 + 同字符跳** + **重复位置跳**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 祖玛桌上有一串彩球 `board`. 你手上有 `hand` 个彩球. 每步选**手上一个球** 插入 board 任意位置; 若插完出现 **≥ 3 个同色相邻** 就消除 (链式反应). 求**清空 board 最少投几次**, 不能清返 `-1`.

**中文**: 祖玛消消乐, 求清空最少步数.

## Key Insights

1. **🔑 结构: 搜索状态 = `(board, hand)` → DFS + 记忆化 / State: (board, hand)**

    每步动作: **哪个手球 × 插入哪个位置** → 得到新 `(board', hand')`. **总步数** 就是 DFS 深度. **相同状态可能被不同路径达到** → 记忆化.

    - **状态 key**: `board + "#" + hand` (字符串拼接, `#` 是分隔避免 "AB" + "CD" vs "A" + "BCD" 撞).
    - **值**: 最少还需几步清空 (-1 = 不可能).

    > **看到"最少步数 + 可搜状态数有限"** → 反射 DFS + memo. 是"游戏类" 题的通用套路.

2. **🔑 `shrink(s)` — 链式消除模拟 / Chain-pop simulation**

    Yang 的写法**递归 shrink**:

    ```cpp
    for (int i = 0, j = 0; i < s.size(); i = j) {
        while (j < s.size() && s[j] == s[i]) j++;   // 找同色段 [i, j)
        if (j - i >= 3) return shrink(s.substr(0, i) + s.substr(j));  // 消 + 递归
    }
    return s;                                        // 无 3+ 段可消
    ```

    - 扫全串找**第一个 ≥ 3 连的段** → 消除 → **递归**继续 (可能消除后前后合并出新的 3+).
    - **递归结束**: 一遍扫无 3+ 连段.

    > **"消除后可能触发新消除"** 是 Zuma 的经典链式反应. **递归**是最干净的表达.

3. **🔑 剪枝 1: `sort(hand)` + 跳同字符 / Prune 1: sorted hand + skip dup**

    ```cpp
    sort(hand.begin(), hand.end());
    ...
    if (i > 0 && hand[i] == hand[i - 1]) continue;
    ```

    - **排序**让同色手球相邻.
    - **跳重复**: 同一步用两个相同颜色的球**结果一样**, 只试第一个.

    > 跟 [0047 Permutations II](../0047-permutations-ii/README.md) 的 sort + `!used[i-1]` 剪枝**同思想** — 排列去重.

4. **🔑 剪枝 2: 插入位置去重 / Prune 2: skip redundant positions**

    ```cpp
    if (j > 0 && board[j - 1] == hand[i]) continue;
    ```

    若**插入位置的前一格**已经是相同颜色, **等价于插在前一格** — 只需要试**该颜色段的最前面一次**.

    ```
    board = "RRB", 手 R
    j = 1: 插入 "RR B" → "RRRB" (可行)
    j = 2: 插入 "RR R B" → "RRRB" (同结果!) 跳过
    ```

    > **同色连段只试插入到该段最左**. 少一半分支.

5. **🔑 剪枝 3: 只在**有可能触发消除**的位置插 / Prune 3: only insert where useful**

    ```cpp
    bool same = j < board.size() && board[j] == hand[i];    // 插在同色球左边 (延长段)
    bool split = j > 0 && j < board.size()
              && board[j - 1] == board[j] && board[j] != hand[i];  // 分裂同色对
    if (!same && !split) continue;
    ```

    - **`same`**: 插入位置右边是同色 → 延长同色段, 可能凑够 3.
    - **`split`**: 前后两个同色 (中间要插), 插入不同色 → 分裂 "XX" 成 "XCX". 这种情况**在特殊 Zuma 变体**里可能有用 (延迟同色触发).
    - 都不满足 → 插入是**纯无效动作**, 剪掉.

    > **"若这一手插入什么变化都不会引起消除, 就是浪费一手"** — 剪枝的直觉理由.

6. **🔑 base cases 顺序 / Base case order matters**

    ```cpp
    if (board.empty()) return 0;      // 清空了 — 需要 0 步
    if (hand.empty())  return -1;     // 手空 board 非空 — 失败
    ```

    **必须先判 board 空**! 若反了, 若同时手空 board 空, 会错判为失败 (-1). Zuma 规则里**手空 board 空** 就是 0 步成功.

7. **🔑 复杂度分析 (松) / Loose complexity bound**

    - 状态数: `(board, hand)` 的可能组合. 手上球数少 (≤ 5), board 长度 ≤ 16 → 有限但难精确.
    - 每次 DFS 里枚举 hand × board_length ≤ 5 × 16 = 80 分支.
    - **记忆化** 保证每状态只被算一次.

    > **Hard 题的复杂度**多数**不精确**, 关键是**剪枝 + 记忆化让实际远小于最坏**.

## Solution

=== "C++"
    ```cpp
    class Solution {
        unordered_map<string, int> memo;

        // 链式消除: 找第一个 ≥ 3 段消除, 递归直到无.
        string shrink(string s) {
            for (int i = 0, j = 0; i < (int)s.size(); i = j) {
                while (j < (int)s.size() && s[j] == s[i]) j++;
                if (j - i >= 3) return shrink(s.substr(0, i) + s.substr(j));
            }
            return s;
        }

        int dfs(const string& board, const string& hand) {
            if (board.empty()) return 0;                             // 清空成功
            if (hand.empty())  return -1;                            // 手空 board 未清 → 失败
            string key = board + "#" + hand;
            if (memo.count(key)) return memo[key];

            int res = INT_MAX;
            for (int i = 0; i < (int)hand.size(); i++) {
                if (i > 0 && hand[i] == hand[i - 1]) continue;       // 剪 1: 同色手球跳

                string nh = hand.substr(0, i) + hand.substr(i + 1);
                for (int j = 0; j <= (int)board.size(); j++) {
                    if (j > 0 && board[j - 1] == hand[i]) continue;  // 剪 2: 位置去重

                    bool same  = j < (int)board.size() && board[j] == hand[i];
                    bool split = j > 0 && j < (int)board.size()
                              && board[j - 1] == board[j] && board[j] != hand[i];
                    if (!same && !split) continue;                    // 剪 3: 无效插入

                    string nb = shrink(board.substr(0, j) + hand[i] + board.substr(j));
                    int t = dfs(nb, nh);
                    if (t != -1) res = min(res, t + 1);
                }
            }
            return memo[key] = (res == INT_MAX ? -1 : res);
        }

    public:
        int findMinStep(string board, string hand) {
            sort(hand.begin(), hand.end());                          // 剪 1 前提
            return dfs(board, hand);
        }
    };
    ```

=== "Python"
    ```python
    from functools import lru_cache

    class Solution:
        def findMinStep(self, board: str, hand: str) -> int:
            hand = ''.join(sorted(hand))

            def shrink(s: str) -> str:
                i = j = 0
                while i < len(s):
                    while j < len(s) and s[j] == s[i]: j += 1
                    if j - i >= 3:
                        return shrink(s[:i] + s[j:])
                    i = j
                return s

            @lru_cache(maxsize=None)
            def dfs(board: str, hand: str) -> int:
                if not board: return 0
                if not hand: return -1
                best = float('inf')
                for i in range(len(hand)):
                    if i > 0 and hand[i] == hand[i - 1]: continue
                    nh = hand[:i] + hand[i + 1:]
                    for j in range(len(board) + 1):
                        if j > 0 and board[j - 1] == hand[i]: continue
                        same = j < len(board) and board[j] == hand[i]
                        split = (j > 0 and j < len(board)
                                 and board[j - 1] == board[j] and board[j] != hand[i])
                        if not same and not split: continue
                        nb = shrink(board[:j] + hand[i] + board[j:])
                        t = dfs(nb, nh)
                        if t != -1:
                            best = min(best, t + 1)
                return -1 if best == float('inf') else best

            return dfs(board, hand)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} board
     * @param {string} hand
     * @return {number}
     */
    var findMinStep = function(board, hand) {
        hand = [...hand].sort().join('');
        const memo = new Map();

        const shrink = (s) => {
            let i = 0, j = 0;
            while (i < s.length) {
                while (j < s.length && s[j] === s[i]) j++;
                if (j - i >= 3) return shrink(s.slice(0, i) + s.slice(j));
                i = j;
            }
            return s;
        };

        const dfs = (board, hand) => {
            if (!board) return 0;
            if (!hand) return -1;
            const key = board + '#' + hand;
            if (memo.has(key)) return memo.get(key);

            let best = Infinity;
            for (let i = 0; i < hand.length; i++) {
                if (i > 0 && hand[i] === hand[i - 1]) continue;
                const nh = hand.slice(0, i) + hand.slice(i + 1);
                for (let j = 0; j <= board.length; j++) {
                    if (j > 0 && board[j - 1] === hand[i]) continue;
                    const same = j < board.length && board[j] === hand[i];
                    const split = j > 0 && j < board.length
                        && board[j - 1] === board[j] && board[j] !== hand[i];
                    if (!same && !split) continue;
                    const nb = shrink(board.slice(0, j) + hand[i] + board.slice(j));
                    const t = dfs(nb, nh);
                    if (t !== -1) best = Math.min(best, t + 1);
                }
            }
            const val = best === Infinity ? -1 : best;
            memo.set(key, val);
            return val;
        };

        return dfs(board, hand);
    };
    ```

## Complexity

- **Time**: 松界, 记忆化让实际远小于最坏. 状态数 O((|board|+1)! × 2^|hand|) 松估.
- **Space**: O(状态数) memo.

## 相关题目

- [0267. Palindrome Permutation II](../0267-palindrome-permutation-ii/README.md) — 回溯生成排列 + 去重
- [0047. Permutations II](../0047-permutations-ii/README.md) — 排列去重母题
- [0037. Sudoku Solver](../0037-sudoku-solver/README.md) — 回溯 + 剪枝
- [0051. N-Queens](../0051-n-queens/README.md) — 经典回溯
- [0079. Word Search](../0079-word-search/README.md) — 网格回溯
- [0301. Remove Invalid Parentheses](../0301-remove-invalid-parentheses/README.md) — 回溯 + 剪枝
- 0679\. 24 Game (待补) — 数字游戏, 回溯 + 浮点
- 0464\. Can I Win (待补) — 博弈 + memo bitmask
- 0691\. Stickers to Spell Word (待补) — Hard, DFS + memo
