# 0541. Reverse String II / 反转字符串 II

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, In-place, String, Simulation · 双指针, 原地, 字符串, 模拟
    - **Link**: [LeetCode](https://leetcode.com/problems/reverse-string-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Every 2k chars, reverse the first k** → step index by **2k**, reverse `[i, min(i + k, n))`. The `min` handles the tail case (< k chars left → reverse all remaining).
>
> **中文**: **每 2k 反转前 k 个** → 步长 **2k**, reverse `[i, min(i + k, n))`. `min` 自动兜末尾 (剩余 < k 就全反).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s` 和 `k`. **每 2k 个字符** 反转前 k 个. 若剩余:

- < k 个 → 全部反转.
- 在 [k, 2k) 之间 → 反转前 k, 后面不动.

**中文**: 每 2k 段反转前 k 个, 末尾按剩余长度处理.

## Key Insights

1. **🔑 灵魂招式: 步长 2k, 反转前 k / Step by 2k, reverse first k**

    循环变量 `i` 每次跳 **2k** — 因为每一段处理 2k 个字符, 只反转前 k, 后 k 保持. 于是 `i` 直接跳过后 k, 进入下一段.

    ```
    段 0: [0, 2k)     reverse [0, k)
    段 1: [2k, 4k)    reverse [2k, 3k)
    段 2: [4k, 6k)    reverse [4k, 5k)
    ...
    ```

    > **"每 M 处理一次" 的题** → 循环步长 M. **步长选对了, 边界跟着简单**.

2. **🔑 三种末尾情况 → 一个 `min` 通吃 / One `min` handles all 3 tail cases**

    | 剩余长度 | 该做什么 | reverse 范围 |
    |---|---|---|
    | ≥ 2k | 反转前 k | `[i, i + k)` |
    | ∈ [k, 2k) | 反转前 k | `[i, i + k)` |
    | < k | 全部反转 | `[i, n)` |

    合成: **`reverse(s + i, s + min(i + k, n))`**. Yang 用 if-else 明写, 也可写成一行:

    ```cpp
    reverse(s.begin() + i, s.begin() + min((int)s.size(), i + k));
    ```

    > **合并三 case 靠 `min` / `max` 兜边界** — 是写"少特判" 代码的通用技巧.

3. **🔑 STL `reverse` 半开区间 `[first, last)` / STL half-open**

    ```cpp
    reverse(s.begin() + i, s.begin() + i + k);
    ```

    - 反转的是**下标 `[i, i + k - 1]` 的字符**.
    - 半开约定跟 [0025 Reverse Nodes in k-Group](../../02-linked-list/0025-reverse-nodes-in-k-group/README.md) 的 `[start, end)` 同源 — 一致的 STL 风格.

    > **半开 = STL 通用约定**. 记熟了跟 begin/end 迭代器的思维模式一致.

4. **🔑 数组下标 → 迭代器 / Index to iterator**

    STL 算法要**迭代器** 不是下标. `s.begin() + i` 就是转换. 每次生成一个新迭代器, 常数级开销.

    > **想少写 `+ begin()`**? C++20 支持 `ranges::reverse(s | views::drop(i) | views::take(k))` — 更炫但少人用.

5. **🔑 每字符最多被 reverse 一次 → O(n) / Amortized O(n)**

    虽然 `reverse` 内部是 O(k), 但**外循环走 n/(2k) 次**, 相乘还是 O(n). 每字符要么在"反转前 k" 里被交换一次, 要么在"保持后 k" 里没动. **总操作 O(n)**.

6. **🔑 跟 [0344 Reverse String](../0344-reverse-string/README.md) / [0151 Reverse Words](../0151-reverse-words-in-a-string/README.md) 的关系 / vs 0344 / 0151**

    | 题 | 反转粒度 |
    |---|---|
    | 0344 | **整个** 数组 |
    | 0541 (本题) | **每 2k 前 k** — 定长段 |
    | 0557 (待补) | **每个单词** |
    | 0151 | 单词**顺序** (整反 + 每词再反 trick) |

    > **"反转" 一族** 按粒度分四种. 记住一个"通用招 = STL reverse (半开区间)".

7. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear**

    - Time: 一遍扫, 每字符最多被反转一次.
    - Space: O(1) 原地.

## Solution

=== "C++"

    **Yang 原版 (if-else 显式)**

    ```cpp
    class Solution {
    public:
        string reverseStr(string s, int k) {
            for (int i = 0; i < (int)s.size(); i += 2 * k) {
                if (i + k <= (int)s.size()) {
                    reverse(s.begin() + i, s.begin() + i + k);           // 剩余 ≥ k
                } else {
                    reverse(s.begin() + i, s.end());                     // 剩余 < k, 全反
                }
            }
            return s;
        }
    };
    ```

    **一行合并 (min 兜底)**

    ```cpp
    class Solution {
    public:
        string reverseStr(string s, int k) {
            int n = s.size();
            for (int i = 0; i < n; i += 2 * k) {
                reverse(s.begin() + i, s.begin() + min(n, i + k));       // 一行, 三 case 通吃
            }
            return s;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def reverseStr(self, s: str, k: int) -> str:
            # Python str 不可变 → 用 list 模拟原地
            chars = list(s)
            n = len(chars)
            for i in range(0, n, 2 * k):
                # 切片赋值 + [::-1] 反转: 语义化 + 一步搞定
                # min(i + k, n) 处理末尾 < k 的情况, 同 C++ 一行版
                chars[i:i + k] = chars[i:i + k][::-1]
                # 注意: Python 切片自动处理越界 — s[i:i+k] 若 i+k > n 会自动截, 不用 min
                # 也就是说这行代码本身就自动兜末尾 case! 比 C++ 还省心
            return ''.join(chars)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {number} k
     * @return {string}
     */
    var reverseStr = function(s, k) {
        // JS string 不可变, 转 array 处理
        const arr = s.split('');
        const n = arr.length;
        for (let i = 0; i < n; i += 2 * k) {
            // slice 半开 [i, min(i+k, n)) — .slice 越界自动截, 跟 Python 切片一致
            // splice 会原地替换; 或用手动 for + swap
            const chunk = arr.slice(i, i + k).reverse();
            for (let j = 0; j < chunk.length; j++) arr[i + j] = chunk[j];
        }
        return arr.join('');
    };
    ```

## Complexity

- **Time**: O(n) — 每字符最多被反转一次.
- **Space**: O(1) 额外 (C++). Python/JS 因 string 不可变需 O(n) 临时数组.

## 相关题目

- [0344. Reverse String](../0344-reverse-string/README.md) — 反转母题
- [0151. Reverse Words in a String](../0151-reverse-words-in-a-string/README.md) — 反转单词顺序
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针
- [0025. Reverse Nodes in k-Group](../../02-linked-list/0025-reverse-nodes-in-k-group/README.md) — 链表 k 组反转, 半开区间同源
- 0557\. Reverse Words in a String III (待补) — 每词反转, 保序
- 0917\. Reverse Only Letters (待补) — 对撞 + 只交换字母
- 0345\. Reverse Vowels of a String (待补) — 对撞 + 只交换元音
- 0189\. Rotate Array (待补) — "三次反转" 数组旋转
