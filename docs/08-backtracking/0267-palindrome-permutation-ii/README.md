# 0267. Palindrome Permutation II / 回文排列 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, Palindrome, Dedupe, Hash Map · 回溯, 回文, 去重, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/palindrome-permutation-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Generate all palindrome permutations of `s`** → check feasibility (≤ 1 odd char, like [0266](../../03-hash-table/0266-palindrome-permutation/README.md)); build **half** (each char / 2) + **mid** (odd char if any); **backtrack permutations of half** with **sort + `!used[i-1]` dedupe**; assemble `half + mid + reverse(half)`.
>
> **中文**: **列出 s 所有回文排列** → 先判可行 (奇字符 ≤ 1); 建 **half** (每字符 //2 份) + **mid** (奇字符); **回溯 half 的排列**, **排序 + `!used[i-1]` 剪枝** 去重; 拼 `half + mid + 反转 half`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s`. 返回它**所有不同回文排列**. 无解返 `[]`.

**中文**: 生成 s 的所有回文排列.

## Key Insights

1. **🔑 灵魂化简: 只回溯**半个**回文 / Only backtrack half the palindrome**

    若 s 有 n 个字符, 朴素回溯 n! 种排列 → 判每个是回文 → **超浪费**.

    **观察**: 回文完全由**左半 + (中位)** 决定. 右半 = 左半的反转.

    → **只回溯 n/2 长的半串**, 复杂度 **n! → (n/2)!**. 例 n=10 时: 3.6M → 120 (30000× 缩减).

    > **"约束型问题" 通常有维度可减**. 找到能 "生成 → 对称扩展" 的核心, 搜索空间指数级降低.

2. **🔑 三步流程 / Three-step pipeline**

    ```
    Step 1: 频次统计, 判可行 (0266 的判定)
    Step 2: 建 half + mid
        - 每字符出现次数 // 2 次入 half
        - 有 1 个奇字符 → 它是 mid
    Step 3: 回溯生成 half 的所有排列, 每个拼成回文
    ```

    Yang 的代码就是这三步的直接翻译. **每步独立, 边界清晰** = 好设计.

3. **🔑 建 half 前**先判可行**, 早退空返 / Feasibility check first**

    ```cpp
    if (freq % 2 == 1) {
        if (!mid.empty()) return {};       // 已有一个奇字符, 又来一个 → 无解
        mid = string(1, c);
    }
    ```

    - **≤ 1 个奇字符** ⇔ 可回文 (跟 [0266](../../03-hash-table/0266-palindrome-permutation/README.md) 灵魂一致).
    - **发现第 2 个奇字符** 立刻返 `[]`, 别浪费回溯.

4. **🔑 排列去重经典模板: 排序 + `!used[i-1]` / Classic permutation dedupe**

    ```cpp
    sort(half.begin(), half.end());
    ...
    if (i > 0 && half[i] == half[i - 1] && !used[i - 1]) continue;
    ```

    - **先排序**: 让相同字符相邻.
    - **`!used[i-1]` 剪枝**: 上一个相同字符**没被用** → 意味着"我们是从 i-1 那条分支**回退**回来的" → 若再选 i, 结果跟"先 i-1 再 i" 一样 → **重复分支, 剪掉**.

    **反例试想**: 若允许 `used[i-1] = false + 选 i`, 生成两个"aa" 一样的排列 (第一次 "a[0]a[1]", 第二次 "a[1]a[0]"). 相同结果.

    > **回溯生成排列 II** ([0047](../0047-permutations-ii/README.md)) 的通用去重模板. 记住 "**排序 + 上一个同值没被用 → 剪**".

5. **🔑 拼接: `half + mid + reverse(half)` / Assemble**

    ```cpp
    string full = path + mid + string(path.rbegin(), path.rend());
    ```

    - `path` 是回溯到的一个半串排列.
    - `mid` 是中间字符 (奇长时) 或 "" (偶长).
    - `string(rbegin, rend)` 用**反迭代器**构造反转副本. (跟 [2744](../../03-hash-table/2744-find-maximum-number-of-string-pairs/README.md) 同款招式).

    > **一行组装完整回文**. 三段式拼接优雅.

6. **🔑 何时到 base? `path.size() == half.size()` / When to hit base case**

    每次选一个 half 里的字符 push 到 path. 选满 half.size() 个 → 到 base → 组装 + 收结果.

    > 回溯**长度型 base case** 的常见模式. 不用管值, 只管数量.

7. **🔑 复杂度: O((n/2)! × n) / Time complexity**

    - 回溯生成 (n/2)! 排列 (去重后可能少很多).
    - 每次组装回文 O(n).
    - Space: O(n) 递归 + 输出.

    > 相比朴素 n! × n 是**指数级**优化.

8. **🔑 跟 [0266](../../03-hash-table/0266-palindrome-permutation/README.md) 关系: 判 → 生成 / Judge → Generate**

    | | 0266 | **0267 (本题)** |
    |---|---|---|
    | 问题 | 是否存在回文排列 | **列出所有** |
    | 输出 | bool | `vector<string>` |
    | 关键判定 | 奇字符 ≤ 1 | 同 + **回溯生成** |
    | 复杂度 | O(n) | O((n/2)! × n) |

    > **"判 → 构造 → 列出"** 是同一问题的**三级难度**. 判是最基础, 列出是最难.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> generatePalindromes(string s) {
            // Step 1: 统计频次
            unordered_map<char, int> cnt;
            for (char c : s) cnt[c]++;

            // Step 2: 建 half + mid, 顺便判可行
            string mid;
            vector<char> half;
            for (auto& [c, freq] : cnt) {
                if (freq % 2 == 1) {
                    if (!mid.empty()) return {};             // 2 个奇字符 → 无解
                    mid = string(1, c);
                }
                for (int i = 0; i < freq / 2; i++) half.push_back(c);
            }

            // Step 3: 回溯 half 排列
            sort(half.begin(), half.end());                  // 排序 → 让相同字符相邻, 去重前提
            vector<string> res;
            string path;
            vector<bool> used(half.size(), false);
            backtrack(half, used, path, mid, res);
            return res;
        }

    private:
        void backtrack(vector<char>& half, vector<bool>& used, string& path,
                       const string& mid, vector<string>& res) {
            if (path.size() == half.size()) {
                // 拼装: path + mid + 反转 path
                res.push_back(path + mid + string(path.rbegin(), path.rend()));
                return;
            }
            for (int i = 0; i < (int)half.size(); i++) {
                if (used[i]) continue;
                if (i > 0 && half[i] == half[i - 1] && !used[i - 1]) continue;   // 同层去重
                used[i] = true;
                path.push_back(half[i]);
                backtrack(half, used, path, mid, res);
                path.pop_back();
                used[i] = false;
            }
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def generatePalindromes(self, s: str) -> list[str]:
            cnt = Counter(s)
            # 判可行 + 建 half + mid — 一次遍历搞完
            mid = ""
            half = []
            for c, freq in cnt.items():
                if freq & 1:            # freq % 2 的位运算加速版
                    if mid: return []
                    mid = c
                half.extend([c] * (freq // 2))       # 一次加 freq/2 个 c
            half.sort()
            res = []
            path = []
            used = [False] * len(half)

            def backtrack():
                if len(path) == len(half):
                    left = ''.join(path)
                    # left[::-1] 是 Python 切片反转 — 跟 C++ string(rbegin, rend) 一样
                    res.append(left + mid + left[::-1])
                    return
                for i in range(len(half)):
                    if used[i]: continue
                    if i > 0 and half[i] == half[i - 1] and not used[i - 1]: continue
                    used[i] = True
                    path.append(half[i])
                    backtrack()
                    path.pop()
                    used[i] = False

            backtrack()
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string[]}
     */
    var generatePalindromes = function(s) {
        // Step 1: 计数
        const cnt = {};
        for (const c of s) cnt[c] = (cnt[c] || 0) + 1;

        // Step 2: 判可行 + 建 half + mid
        let mid = "";
        const half = [];
        for (const [c, freq] of Object.entries(cnt)) {
            if (freq & 1) {
                if (mid) return [];
                mid = c;
            }
            // Array.from({length: k}, () => c) 生成 k 个 c 的数组; ... spread 展开
            half.push(...Array.from({length: Math.floor(freq / 2)}, () => c));
        }
        half.sort();

        const res = [];
        const path = [];
        const used = new Array(half.length).fill(false);

        const backtrack = () => {
            if (path.length === half.length) {
                const left = path.join('');
                // reverse 会原地改数组, 得先 slice 复制. 或用 .split('').reverse().join('')
                res.push(left + mid + [...left].reverse().join(''));
                return;
            }
            for (let i = 0; i < half.length; i++) {
                if (used[i]) continue;
                if (i > 0 && half[i] === half[i - 1] && !used[i - 1]) continue;
                used[i] = true;
                path.push(half[i]);
                backtrack();
                path.pop();
                used[i] = false;
            }
        };
        backtrack();
        return res;
    };
    ```

## Complexity

- **Time**: O((n/2)! × n) — 回溯 half 排列 + 每次拼装 O(n).
- **Space**: O(n) — 递归栈 + path + used.

## 相关题目

- [0266. Palindrome Permutation](../../03-hash-table/0266-palindrome-permutation/README.md) — 判可行版, 母题
- [0242. Valid Anagram](../../03-hash-table/0242-valid-anagram/README.md) — 计数数组母题
- [0246. Strobogrammatic Number](../../05-two-pointers/0246-strobogrammatic-number/README.md) — 判"旋转对称"
- [2744. Find Maximum Number of String Pairs](../../03-hash-table/2744-find-maximum-number-of-string-pairs/README.md) — 反迭代器构造反转
- [0047. Permutations II](../0047-permutations-ii/README.md) — 排列 II, 同款 sort + `!used[i-1]` 去重模板
- [0046. Permutations](../0046-permutations/README.md) — 全排列 (无重复元素)
- 0031\. Next Permutation (待补) — 下一个字典序排列
- 0784\. Letter Case Permutation (待补) — 字母大小写排列
- 0022\. Generate Parentheses (待补) — 生成合法括号
