# 0017. Letter Combinations of a Phone Number / 电话号码的字母组合

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, String, Hash Table · 回溯, 字符串, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a string of digits (`2-9`), return all possible letter combinations the number could represent on a standard phone keypad. Each digit maps to 3-4 letters (`2→abc`, `7→pqrs`, etc.).

**中文**: 给一个 `2-9` 数字串, 按手机九宫格键位 (`2→abc`, `7→pqrs` 等) 返回所有可能的字母组合.

## 思路

### Core idea

每个 digit 对应一个字母集合, **每层 (= 一个 digit) 从它的集合里选一个字符**, 选完所有 digits 就拿到一条答案. 用 `idx` 推进当前在第几位.

回溯三件套 (push → recurse → pop) 跟 [0077](../0077-combinations/README.md) 一样, 但**不再需要 `startIndex`** — 因为这题是"按 digit 顺序逐层", idx + 1 自然就是下一层.

### Key Insights

1. **多集合每位选一个 ≠ 单集合多次选 / Multi-set per-level vs single-set with startIndex**

    | 题型 | 候选集合 | 防重机制 |
    |---|---|---|
    | [0077](../0077-combinations/README.md) / [0216](../0216-combination-sum-iii/README.md) | **同一集合** `[1,n]` 多次抽 | `startIndex` 单向往后 |
    | **0017 (本题)** | **N 个不同集合**, 每层一个 | 不需要 — `idx` 推进自带顺序 |
    | 0046 排列 (待补) | **同一集合**, 顺序敏感 | `used[]` 数组 |

    本题是 "N 个集合的笛卡尔积". 这是回溯系列里**最简单**的一种形态, 没有去重压力.

2. **`mapping` 用 vector 而不是 unordered_map / Index-based dispatch**

    `digit - '0'` 直接当下标查 `mapping[2..9]`. 比 `unordered_map<char, string>` 快 (避免 hash) 且代码更短. 哈希表只在 key 是任意字符串时才需要 — 这里 key 是 0-9 数字, 数组就行.

    `mapping[0]` 和 `mapping[1]` 给空串占位, 因为标准九宫格 0 和 1 没字母, 题目也保证输入不含 0/1.

3. **终止条件: `idx == digits.size()` / Path length is input-determined**

    跟 0077 的 `path.size() == k` 等价 — 都是"路径攒满了就收". 区别是 k 来自参数, 这题的"k" 就是 digits 的长度, 由 idx 比较即可.

4. **空输入特判 / Empty input must return `[]` not `[""]`**

    `digits == ""` 时不能让 backtrack 跑一遍 — 那会把空 path 直接 push 进 res, 得到 `[""]`, 题目要的是 `[]`. 入口判一下 `if (digits.empty()) return {};` 是最干净的解决.

5. **`path` 用 `string` 而非 `vector<char>` / String as growable char buffer**

    C++ `string::push_back(char)` 和 `pop_back()` 都是 O(1) 摊销, 跟 vector 一样. 这里直接当 char 容器使. 最后收果实 `res.push_back(path)` 自动拷贝当前 string.

6. **不需要剪枝 / No pruning needed**

    没有 sum 约束, 也没有"凑齐 k 个" 的范围剪枝空间 (因为每层必须挑一个, 不能跳). 答案数刚好就是 ∏(每个 digit 的字母数), 时间下界注定是 O(3^n × 4^m).

### 一句话总结

**N 个集合的笛卡尔积. 每个 digit 对应一个字母集合, idx 推进当前位, 每层 push 一个字符 → 递归 → pop. 选完 digits 就收果实.**

### 图解

`digits = "23"` 的决策树:

```mermaid
graph TD
    R[""] --> A1["a"]
    R --> A2["b"]
    R --> A3["c"]
    A1 --> B1["ad"]
    A1 --> B2["ae"]
    A1 --> B3["af"]
    A2 --> B4["bd"]
    A2 --> B5["be"]
    A2 --> B6["bf"]
    A3 --> B7["cd"]
    A3 --> B8["ce"]
    A3 --> B9["cf"]
```

第 0 层选 "2" 的字母 `a/b/c`, 第 1 层选 "3" 的字母 `d/e/f`. 共 3×3 = 9 条.

## Solution

=== "C++"
    ```cpp
    class Solution {
        vector<string> res;
        string path;
        const vector<string> mapping = {
            "",     "",     "abc",  "def",
            "ghi",  "jkl",  "mno",  "pqrs",
            "tuv",  "wxyz"
        };

        void backtrack(const string& digits, int idx) {
            if (idx == (int)digits.size()) {
                res.push_back(path);              // path 拷贝快照
                return;
            }
            const string& letters = mapping[digits[idx] - '0'];
            for (char c : letters) {
                path.push_back(c);
                backtrack(digits, idx + 1);       // idx + 1 而不是 startIndex
                path.pop_back();
            }
        }
    public:
        vector<string> letterCombinations(string digits) {
            if (digits.empty()) return {};        // 空输入返回 [], 不能让 backtrack 跑出 [""]
            backtrack(digits, 0);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # 类常量: list 当查找表用, 下标 = digit. 比 dict 稍快, 且代码短
        MAPPING = ["", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"]

        def letterCombinations(self, digits: str) -> list[str]:
            if not digits:
                return []
            res, path = [], []                    # path 用 list[str], 比 str 拼接快; 最后 ''.join
            def backtrack(idx: int):
                if idx == len(digits):
                    res.append(''.join(path))     # ''.join: 把 ['a','d'] 拼成 'ad', O(总长) 一次性
                    return
                for c in self.MAPPING[int(digits[idx])]:
                    path.append(c)
                    backtrack(idx + 1)
                    path.pop()
            backtrack(0)
            return res
    ```

    !!! tip "Pythonic 一行版"
        `itertools.product` 直接给笛卡尔积, 完全不用手写回溯:
        ```python
        from itertools import product
        def letterCombinations(self, digits: str) -> list[str]:
            if not digits:
                return []
            groups = [self.MAPPING[int(d)] for d in digits]
            # product(*groups): 等价 N 层嵌套 for 循环, 产出所有 (a,d), (a,e), ... tuple
            # ''.join(t): 把 tuple 拼成 string
            return [''.join(t) for t in product(*groups)]
        ```
        C++ 没原生 cartesian product, 只能手写回溯. Python 的 `itertools` 把这类"枚举所有组合"题压成一行 — 是面试可以亮一手的写法.

=== "JavaScript"
    ```javascript
    /**
     * @param {string} digits
     * @return {string[]}
     */
    var letterCombinations = function(digits) {
        if (!digits) return [];
        const mapping = ["", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"];
        const res = [], path = [];
        const backtrack = (idx) => {
            if (idx === digits.length) {
                res.push(path.join(''));          // path.join(''): list[string] → string, 等价 Python ''.join
                return;
            }
            for (const c of mapping[+digits[idx]]) {  // +digits[idx]: 字符串转数字 (unary plus), 等价 Number()
                path.push(c);
                backtrack(idx + 1);
                path.pop();
            }
        };
        backtrack(0);
        return res;
    };
    ```

## Complexity

- **Time**: O(3^a × 4^b × L) — a 个 3-letter digit, b 个 4-letter digit (7 和 9), L = digits.length. 每条路径 O(L) 拷贝.
- **Space**: O(L) recursion + path. 输出本身是 3^a × 4^b 条.

## 易错点

- **空 digits 必须返回 `[]`, 不是 `[""]`**: 入口特判. 不特判的话 backtrack(0) 立刻 idx == 0 == size, push 空 path 进 res, 得到 `[""]`. LC 判错.
- **mapping 索引从 0 开始, 包含 "" 占位**: 必须填上 mapping[0] = mapping[1] = "", 否则下标偏移. 题目保证 digits 不含 0/1, 所以这两位不会被实际读到, 但占位必须留.
- **`idx + 1` 不是 `idx`**: 每个 digit 只能贡献一位, 下一层必须看下一个 digit. 写成 `idx` 会无限递归同一个 digit.
- **不需要 `startIndex`**: 这题没有"同集合避免重复"问题. 强加 `startIndex` 会丢答案 (例如 "23" 应该有 9 种, 加 startIndex 后只剩 6 种).
- **`digits[idx] - '0'`** (C++) / `int(digits[idx])` (Python) / `+digits[idx]` (JS): 字符转数字别忘. C++ 直接减 `'0'`; Python/JS 用 int/Number 转.
- **path 用 string + push_back/pop_back 不是 += 和 substr**: 后者每次创建新 string, O(n) 拷贝, 总 O(n²) 慢. push_back / pop_back 是 O(1) 摊销.

## 相关题目

- [0077. Combinations](../0077-combinations/README.md) — 单集合 + startIndex 的对照组
- [0216. Combination Sum III](../0216-combination-sum-iii/README.md) — 单集合 + startIndex + 和约束
- [0113. Path Sum II](../../07-binary-tree/0113-path-sum-ii/README.md) — 同款显式 push/pop, 走二叉树
- [0039. Combination Sum](../0039-combination-sum/README.md) — 允许同元素重复, recurse(i)
- 0046. Permutations (待补) — 同集合 + 顺序敏感, 用 `used[]`
- [0093. Restore IP Addresses](../0093-restore-ip-addresses/README.md) — 字符串切分 + 每段约束, 同款 idx 推进
- 0784. Letter Case Permutation (待补) — 同款每位选一个 (大小写两种), 简化版
