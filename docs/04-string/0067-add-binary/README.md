# 0067. Add Binary / 二进制求和

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: String, Math, Big Integer · 字符串, 数学, 大数
    - **Link**: [LeetCode](https://leetcode.com/problems/add-binary/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given two binary strings `a` and `b`, return their sum as a binary string.

**中文**: 给两个二进制字符串 `a` 和 `b`, 返回它们的和 (也是二进制字符串).

## 思路

### Core idea

**[0415 Add Strings](../0415-add-strings/README.md) 的二进制版**. 模板**一字不改**, 只换两个常数:

- `s % 10` → `s % 2`
- `s / 10` → `s / 2`

两指针从尾扫, 维护 `carry`, while 条件含 carry, 末尾追加 + 最后 reverse — 全部一样.

### Key Insights

1. **大数加法模板的"base 切换" / Same template, different base**

    | 题 | base | 末位运算 |
    |---|---|---|
    | [0415](../0415-add-strings/README.md) Add Strings | 10 | `s % 10`, `s / 10` |
    | **0067 (本题)** | **2** | `s % 2`, `s / 2` |
    | 0989 Add to Array-Form (待补) | 10 但输入是 `vector<int>` | 同 0415, 仅容器换 |

    把这个模板背下来, 之后遇到 base 16 (hex), base k 的题改两个常数就行.

2. **`s % 2` 最多是 1, carry 最多是 1 / Small numbers, no overflow risk**

    二进制下每位只能是 0 或 1, 两位加上 carry 最多 = 3, carry 最多 = 1. 比十进制更简单, 没有任何溢出风险 — `int` 都够用.

3. **同款 `while (i >= 0 || j >= 0 || carry)` / Three-condition or**

    跟 0415 完全相同的尾循环条件: 任一指针没走完, 或 carry 还有, 就继续. 漏 carry → "11" + "1" 错算成 "0" 而不是 "100".

4. **如果允许 int / long long 也对, 但题目要 string** / Why we don't convert to int

    题目输入字符串可能很长 (n ≤ 10⁴), 转 `long long` 会溢出. 必须 string 加法. 这跟 [0306](../../08-backtracking/0306-additive-number/README.md) 是同样的原因.

### 一句话总结

**0415 模板, `% 10 → % 2`, `/ 10 → / 2`. 其它一字不改.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string addBinary(string a, string b) {
            string res;
            int i = a.size() - 1, j = b.size() - 1, carry = 0;
            while (i >= 0 || j >= 0 || carry) {
                int x = (i >= 0) ? a[i--] - '0' : 0;
                int y = (j >= 0) ? b[j--] - '0' : 0;
                int s = x + y + carry;
                res += ('0' + s % 2);                     // 末位: s mod 2
                carry = s / 2;                            // 进位: s div 2
            }
            reverse(res.begin(), res.end());
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def addBinary(self, a: str, b: str) -> str:
            # 一行 Pythonic: 用 int(_, 2) 转 base-2, 加完用 bin(...)[2:] 截掉 '0b' 前缀
            # 这是 Python 大整数原生支持 + 内建 base 转换的优势 (C++ / JS 没这么方便)
            return bin(int(a, 2) + int(b, 2))[2:]
        # 手写版 (跟 C++ 一字对应, 给学模板的人留着):
        # def addBinary(self, a, b):
        #     i, j, carry = len(a) - 1, len(b) - 1, 0
        #     digits = []
        #     while i >= 0 or j >= 0 or carry:
        #         x = int(a[i]) if i >= 0 else 0
        #         y = int(b[j]) if j >= 0 else 0
        #         s = x + y + carry
        #         digits.append(str(s % 2))
        #         carry = s // 2
        #         i -= 1; j -= 1
        #     return ''.join(reversed(digits))
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} a
     * @param {string} b
     * @return {string}
     */
    var addBinary = function(a, b) {
        // BigInt 一行: BigInt('0b' + a) parse 二进制字符串, 加完 toString(2) 转回
        // (Number 在 a.length > 53 时会精度丢失, 必须 BigInt)
        return (BigInt('0b' + a) + BigInt('0b' + b)).toString(2);
        /* 手写版:
        let i = a.length - 1, j = b.length - 1, carry = 0;
        const digits = [];
        while (i >= 0 || j >= 0 || carry) {
            const x = i >= 0 ? +a[i--] : 0;
            const y = j >= 0 ? +b[j--] : 0;
            const s = x + y + carry;
            digits.push(s % 2);
            carry = (s / 2) | 0;
        }
        return digits.reverse().join('');
        */
    };
    ```

## Complexity

- **Time**: O(max(m, n)).
- **Space**: O(max(m, n)) 输出.

## 易错点

- **`% 2` 不是 `% 10`**: 抄 0415 模板时最容易忘改的两处. 写错 base 直接输出十进制位字符 (像 "9" + "9" 出 "18" 而不是 "10010"), 测试用例秒挂.
- **`while` 必须含 `carry`**: 同 0415. 漏 carry → "11" + "1" 算成 "00" 而不是 "100".

## 相关题目

- [0415. Add Strings](../0415-add-strings/README.md) — 十进制版, 本题去掉 base 切换就是它
- [0306. Additive Number](../../08-backtracking/0306-additive-number/README.md) — 同款大数加法模板, 嵌在切割回溯里复用
- 0989\. Add to Array-Form of Integer (待补) — base 10 但一边输入是 `vector<int>`
- 0002\. Add Two Numbers (待补) — 链表版的大数加法
- 0043\. Multiply Strings (待补) — 大数乘法, 内部用 0415 当子例程
