# 0344. Reverse String / 反转字符串

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, In-place, Array · 双指针, 原地, 数组
    - **Link**: [LeetCode](https://leetcode.com/problems/reverse-string/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Reverse array in-place** → **opposite two pointers**: swap `s[i]` and `s[j]`, march `i++, j--` until `i >= j`.
>
> **中文**: **原地反转数组** → **对撞双指针**: 两端交换 `s[i]` 和 `s[j]`, 往中间走直到相遇.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符数组 `s`, **原地** 反转. **O(1) 额外空间**.

**中文**: 原地反转字符数组.

## Key Insights

1. **🔑 对撞双指针的最简版本 / Simplest opposite-two-pointers**

    数组反转 = **首尾对调**, 从两端向中间走. 每步:

    ```
    swap(s[i], s[j])
    i++; j--
    ```

    - **循环条件 `i < j`**: 相遇时 (i == j) 无需交换 (自己跟自己). 可写 `<=` 但**多一次空操作**.
    - **无浪费**: 每步交换一对, n/2 次搞定.

    > **对撞双指针家族最基础的一员**. [0977 Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) / [0015 3Sum](../0015-3sum/README.md) / [0246 Strobogrammatic](../0246-strobogrammatic-number/README.md) 都是它的推广.

2. **🔑 swap 三行 vs `std::swap` 一行 / Swap idioms**

    Yang 的**手动 swap** (对齐 C 风格):

    ```cpp
    char tmp = s[i];
    s[i] = s[j];
    s[j] = tmp;
    ```

    等价 STL 一行:

    ```cpp
    swap(s[i], s[j]);        // <utility>, O(1)
    ```

    > **面试写哪个都行**. `std::swap` 更简洁, 手动 swap 更教学. 内容一样.

3. **🔑 对比 [0206 Reverse Linked List](../../02-linked-list/0206-reverse-linked-list/README.md): 数组 vs 链表 / Array vs linked list**

    | | 数组 (本题) | 链表 (0206) |
    |---|---|---|
    | 随机访问 | ✅ (O(1)) | ❌ (只能顺序走) |
    | 反转策略 | **对撞双指针交换** | **三指针翻转 next** |
    | 时间 | O(n) | O(n) |
    | 空间 | O(1) 原地 | O(1) 迭代版 |

    > **同一"反转" 问题**, 数据结构不同 → **算法完全不同**. 是"数据结构决定算法" 的漂亮例子.

4. **🔑 备选: STL `reverse` / STL alternative**

    ```cpp
    reverse(s.begin(), s.end());     // 一行, 内部就是双指针 swap
    ```

    面试**主动提** "**若不要求手写**, STL 一行". 但面试题多数考手写 — Yang 版正确.

5. **🔑 递归版 (教学向, 不推荐) / Recursive (educational only)**

    ```cpp
    void helper(vector<char>& s, int i, int j) {
        if (i >= j) return;
        swap(s[i], s[j]);
        helper(s, i + 1, j - 1);
    }
    ```

    - **递归 = 迭代**这个循环, 语义相同.
    - **代价**: O(n) 栈空间, 长数组可能 stack overflow. **不推荐生产用**.

6. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    - Time: n/2 次交换.
    - Space: 1 个 tmp.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        void reverseString(vector<char>& s) {
            int i = 0, j = (int)s.size() - 1;
            while (i < j) {
                char tmp = s[i];
                s[i] = s[j];
                s[j] = tmp;
                i++; j--;
            }
            // 或一行: while (i < j) swap(s[i++], s[j--]);
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def reverseString(self, s: list[str]) -> None:
            # Python 元组解包一步交换 — 右侧先算完, 再赋左侧, 完全无需 tmp
            # 等价 C++ swap(s[i], s[j])
            i, j = 0, len(s) - 1
            while i < j:
                s[i], s[j] = s[j], s[i]
                i += 1
                j -= 1
            # Pythonic 一行: s[:] = s[::-1] (原地切片赋值, 满足"原地" 要求)
            # 但那样跟"手写对撞双指针" 意图不符, 教学时 Yang 版更好
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {character[]} s
     * @return {void} Do not return anything, modify s in-place instead.
     */
    var reverseString = function(s) {
        // JS 无 tuple swap 语法. 用解构赋值: [a, b] = [b, a] 一步交换
        // 等价 C++ swap 的三行. 内部 JS 会创建临时数组 — 常数开销
        let i = 0, j = s.length - 1;
        while (i < j) {
            [s[i], s[j]] = [s[j], s[i]];
            i++;
            j--;
        }
        // 也可 s.reverse() — 原地反转, 一行. 但那是内建, 不是"手写".
    };
    ```

## Complexity

- **Time**: O(n) — n/2 次交换.
- **Space**: O(1) — 一个 tmp.

## 相关题目

- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针母题
- [0015. 3Sum](../0015-3sum/README.md) — 排序 + 对撞 + 3 层去重
- [0246. Strobogrammatic Number](../0246-strobogrammatic-number/README.md) — 对撞 + 旋转映射
- [0206. Reverse Linked List](../../02-linked-list/0206-reverse-linked-list/README.md) — 链表反转 (三指针)
- [0025. Reverse Nodes in k-Group](../../02-linked-list/0025-reverse-nodes-in-k-group/README.md) — 链表 k 组反转
- 0345\. Reverse Vowels of a String (待补) — 对撞 + 只交换元音
- [0541. Reverse String II](../0541-reverse-string-ii/README.md) — 每 2k 反转前 k 个
- 0125\. Valid Palindrome (待补) — 对撞判回文
- 0917\. Reverse Only Letters (待补) — 对撞 + 只交换字母
- [0151. Reverse Words in a String](../0151-reverse-words-in-a-string/README.md) — 整串 + 每词反转 trick, 或 stringstream 分词
