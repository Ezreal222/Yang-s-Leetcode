# 0415. Add Strings / 字符串相加

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: String, Math, Simulation, Big Integer · 字符串, 数学, 模拟, 大数
    - **Link**: [LeetCode](https://leetcode.com/problems/add-strings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given two non-negative integers as strings, return their sum as a string. No conversion to BigInteger/`long long` allowed — must do it digit-by-digit.

**中文**: 给两个用 string 表示的非负整数, 返回它们的和 (也是 string). 不允许直接转 BigInteger / `long long`, 必须按位模拟.

## 思路

### Core idea

**大数加法模板** (必背). 两指针从两个 string 的**末尾**往前走, 维护一个 `carry`:

1. 取当前位数字 (越界补 0).
2. 三者相加: `s = x + y + carry`.
3. 当前位 `s % 10` 末尾追加到 res, `carry = s / 10`.
4. while 条件三合一: 两个指针都没走完, **或** carry 还有.
5. 最后 `reverse(res)` 一次性 O(n) 翻转.

### Key Insights

1. **`- '0'` 是字符→数字的核心 / Char-to-digit IS the trick**

    Yang 自己标的关键: `num1[i] - '0'` 才拿到 0-9 整数. 漏了的话:

    - `num1[i]` 是 ASCII char ('0' = 48, '9' = 57). 直接加会得到 100+ 的乱码.
    - 反向: 写回 char 用 `'0' + digit`. 同一原理.

    口诀: **读字符就 `- '0'`, 写字符就 `+ '0'`**.

2. **while 条件必须包含 `carry` / Don't forget the final carry-out**

    `while (i >= 0 || j >= 0 || carry)`. 漏 carry → "99" + "1" 会算出 "00" 而不是 "100", 因为两指针走完后 carry=1 还没写进 res. 这是大数加法第二大坑.

3. **末尾追加 + 最后 reverse, O(n) / The string-builder pattern**

    每步 `res += char` 摊 O(1), 最后一次 `reverse` O(n), 合计 **O(n)**.

    若改 `res = char + res` 前插: 每次 O(n) 拷贝整个 string → 总 O(n²). [0306](../../08-backtracking/0306-additive-number/README.md) 用了这个相同模板, 那题输入 35 位差距明显.

4. **不需要补齐长度 / Two pointers handle the asymmetry**

    天真做法是把短的那个前面补 0 再逐位加. 不必 — 三元 `i >= 0 ? num1[i--] - '0' : 0` 一行就处理了"长度不同". 代码更短.

5. **同模板的"周边题" / Reusable across base 2, base 10, mixed**

    把 base 换成 2 就是 [0067 Add Binary](待补), base 16 就是 hex 加法. 一边换成 `vector<int>` 就是 0989 Add to Array-Form. 模板不变, 只是 `% 10` 改成 `% base`.

### 一句话总结

**两指针从末尾扫, carry 进位. while 条件含 carry, 字符 `- '0'` 拿数字, 末尾追加 + 最后 reverse.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string addStrings(string num1, string num2) {
            string res;
            int i = num1.size() - 1, j = num2.size() - 1, carry = 0;
            while (i >= 0 || j >= 0 || carry) {
                int x = (i >= 0) ? num1[i--] - '0' : 0;          // 字符 → 数字, 关键的 -'0'
                int y = (j >= 0) ? num2[j--] - '0' : 0;
                int s = x + y + carry;
                res += ('0' + s % 10);                            // 数字 → 字符, 关键的 +'0'
                carry = s / 10;
            }
            reverse(res.begin(), res.end());
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def addStrings(self, num1: str, num2: str) -> str:
            # Python int 原生大数, 一行就能写 — 但题目精神是练大数加法模板, 所以手写
            i, j, carry = len(num1) - 1, len(num2) - 1, 0
            digits = []                                           # list 当 string buffer, 比 += 拼接快
            while i >= 0 or j >= 0 or carry:
                # ord(c) - ord('0') 等价 C++ c - '0'; 也可以 int(num1[i]) 但更慢一点
                x = int(num1[i]) if i >= 0 else 0
                y = int(num2[j]) if j >= 0 else 0
                s = x + y + carry
                digits.append(str(s % 10))
                carry = s // 10
                i -= 1
                j -= 1
            return ''.join(reversed(digits))                      # 反向迭代 + join, 一次性拼接 O(n)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} num1
     * @param {string} num2
     * @return {string}
     */
    var addStrings = function(num1, num2) {
        let i = num1.length - 1, j = num2.length - 1, carry = 0;
        const digits = [];                                        // 用数组 + join 比 string += 快
        while (i >= 0 || j >= 0 || carry) {
            // num1.charCodeAt(i) - 48 等价 C++ - '0' (48 = '0' 的 ASCII)
            // 也可以 +num1[i] (unary plus 转 number), 二选一
            const x = i >= 0 ? num1.charCodeAt(i--) - 48 : 0;
            const y = j >= 0 ? num2.charCodeAt(j--) - 48 : 0;
            const s = x + y + carry;
            digits.push(s % 10);
            carry = (s / 10) | 0;                                 // | 0 强制整数化, 等价 Math.floor (正数情况)
        }
        return digits.reverse().join('');                         // Array.reverse 原地, join 拼接
    };
    ```

## Complexity

- **Time**: O(max(m, n)) — 一次扫两个串.
- **Space**: O(max(m, n)) 输出本身.

## 易错点

- **`- '0'` 别漏**: Yang 自己 flag 的. 漏了得到 ASCII 值, 整个结果错位. 写字符回 `+ '0'` 同理.
- **while 条件含 `carry`**: 漏 carry 会让最高位的进位丢失. "99" + "1" 错算成 "0".

## 相关题目

- [0306. Additive Number](../../08-backtracking/0306-additive-number/README.md) — 把这题的 addStrings 当子例程用
- 0067. Add Binary (待补) — 同模板, base 2
- 0989. Add to Array-Form of Integer (待补) — 一边换成 vector<int>
- 0043. Multiply Strings (待补) — 大数乘法, 用大数加法当子例程
- 0002. Add Two Numbers (待补) — 链表版的大数加法
