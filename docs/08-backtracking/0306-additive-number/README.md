# 0306. Additive Number / 累加数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, String, Big Integer · 回溯, 字符串, 大数
    - **Link**: [LeetCode](https://leetcode.com/problems/additive-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a string of digits, determine if it forms an "additive sequence" — at least **3** numbers where each is the sum of the previous two. No leading zeros except the value "0" itself.

**中文**: 给一个数字字符串, 判断是否能切成 ≥3 个数, 使每个数 = 前两个之和. 段不允许前导 0 (单独的 "0" 除外).

## 思路

### Core idea

**切割回溯 + 大数加法**. 前两段是种子, 一旦定下来整个序列就唯一确定:

1. **双层 for** 枚举第一段长度 i 和第二段终点 j → 拿到 `s1 = num[0..i-1]`, `s2 = num[i..j-1]`.
2. **递归 check**: 计算 `sum = s1 + s2` (string 加法), 检查 `num[j..]` 开头是否就是 `sum`. 若是, 滑动 (s1, s2) → (s2, sum), 从 `j + sum.size()` 继续.
3. 滑到 `i == num.size()` 表示整串切完且全合法 → 返 true.

### Key Insights

1. **切割问题里的"跨段约束" / Cross-segment constraint**

    | 题 | 段内约束 | 跨段约束 |
    |---|---|---|
    | [0131](../0131-palindrome-partitioning/README.md) | 每段必须回文 | 无 |
    | [0093](../0093-restore-ip-addresses/README.md) | 0-255 + 无前导 0 + 4 段 | 无 |
    | **0306 (本题)** | 无前导 0 | **第 k 段 = 第 k-1 + 第 k-2** |

    跨段约束让 "前两段定下来 = 整序列定下来", 所以**只需要 O(n²) 枚举前两段**, 不需要对每段都试.

2. **大数加法是 must-have, 不是 nice-to-have / String addition is mandatory**

    LC 输入长度 ≤ 35 位. 35 位十进制数远超 `long long` (≈19 位). 所以必须 string 表示 + string 加法. 即使用 Python (原生大整数) 或 JS (BigInt), **C++ 里这道题就是 string 加法练习题**.

3. **string addition 的末尾追加 + 最后 reverse / Why not front-insert**

    Yang 这段经验值得记成肌肉:

    - 末尾 `res += char`: 均摊 **O(1)** 每次, 整体 O(n).
    - 最后 `reverse(res)`: 一次性 O(n).
    - **合计 O(n)**.

    若改成 `res = char + res` 前插, 每次拷贝整个 res, 总 O(n²). 错的不是结果, 是性能 — 输入 35 位时差距明显.

4. **大数加法模板 (必背) / The big-int add template**

    ```cpp
    // 等价于人手算十进制加法: 从低位往高位扫, 维护 carry
    while (i >= 0 || j >= 0 || carry) {
        int x = (i >= 0) ? a[i--] - '0' : 0;
        int y = (j >= 0) ? b[j--] - '0' : 0;
        int s = x + y + carry;
        res += ('0' + s % 10);   // 末尾追加当前位
        carry = s / 10;
    }
    reverse(res.begin(), res.end());
    ```
    同款模板复用题: 0415 Add Strings (待补), 0067 Add Binary (待补), 0989 Add to Array-Form (待补). 把 base 10 换成 base 2 / 16 都不变.

5. **`check` 的参数滑动 / Sliding window in recursion**

    每次成功匹配后, 下一层的"前两段"变成 (**当前 s2**, **当前 sum**), 起点变成 `j + sum.size()`. 写错滑动顺序 (例如 pass `(sum, s2)` 反了) 是这题最容易调半天的 bug.

6. **Leading-zero 必须双层都拦 / Guard at both levels**

    - 第一段: `num[0] == '0' && i > 1` break.
    - 第二段: `num[i] == '0' && j > i + 1` break.

    单字符 "0" 是合法段 (例如 "101" → "1" + "0" + "1"), 但 "01", "02" 不合法.

### 一句话总结

**枚举前两段 (i, j) 双层 for; 递归 check 算 string 加法 + 滑动 (s2, sum). Leading-0 双层拦; n ≤ 35 必须 string 加法.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool check(const string& num, int i, string s1, string s2) {
            if (i == (int)num.size()) return true;
            string sum = addStrings(s1, s2);
            if (i + (int)sum.size() > (int)num.size()) return false;
            if (num.substr(i, sum.size()) != sum) return false;
            return check(num, i + sum.size(), s2, sum);          // 滑动: (s1, s2) → (s2, sum)
        }
        string addStrings(const string& a, const string& b) {
            string res;
            int i = a.size() - 1, j = b.size() - 1, carry = 0;
            while (i >= 0 || j >= 0 || carry) {
                int x = (i >= 0) ? a[i--] - '0' : 0;
                int y = (j >= 0) ? b[j--] - '0' : 0;
                int s = x + y + carry;
                res += ('0' + s % 10);                           // 末尾追加, 摊 O(1)
                carry = s / 10;
            }
            reverse(res.begin(), res.end());                     // 一次性 reverse, O(n)
            return res;
        }
        bool isAdditiveNumber(string num) {
            int n = num.size();
            for (int i = 1; i <= n / 2; i++) {
                if (num[0] == '0' && i > 1) break;               // 第一段 leading 0
                for (int j = i + 1; j <= (n + i) / 2; j++) {
                    if (num[i] == '0' && j > i + 1) break;       // 第二段 leading 0
                    string s1 = num.substr(0, i);
                    string s2 = num.substr(i, j - i);
                    if (check(num, j, s1, s2)) return true;
                }
            }
            return false;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isAdditiveNumber(self, num: str) -> bool:
            n = len(num)
            # Python int 原生大数, 不用手写 string addition;
            # 但 C++ 必须的 string 加法模板要记 (0415 等都同模板)
            def check(i: int, a: int, b: int) -> bool:
                if i == n:
                    return True
                s = str(a + b)                                   # int → str 拼接, 等价 C++ addStrings
                if not num.startswith(s, i):                     # str.startswith(prefix, start): 从 i 开始找前缀
                    return False
                return check(i + len(s), b, a + b)               # 滑动

            for i in range(1, n // 2 + 1):
                if num[0] == '0' and i > 1:
                    break
                for j in range(i + 1, (n + i) // 2 + 1):
                    if num[i] == '0' and j > i + 1:
                        break
                    if check(j, int(num[:i]), int(num[i:j])):
                        return True
            return False
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} num
     * @return {boolean}
     */
    var isAdditiveNumber = function(num) {
        const n = num.length;
        // BigInt 处理任意精度, 等价 Python int.
        // BigInt 字面量后缀 'n' 或者 BigInt(x) 转, 普通 + 不能跟 number 混
        const check = (i, a, b) => {
            if (i === n) return true;
            const s = (a + b).toString();                        // BigInt.toString(): 转成十进制 string
            if (num.slice(i, i + s.length) !== s) return false;
            return check(i + s.length, b, a + b);
        };
        for (let i = 1; i <= n / 2; i++) {
            if (num[0] === '0' && i > 1) break;
            for (let j = i + 1; j <= (n + i) / 2; j++) {
                if (num[i] === '0' && j > i + 1) break;
                if (check(j, BigInt(num.slice(0, i)), BigInt(num.slice(i, j)))) {
                    return true;
                }
            }
        }
        return false;
    };
    ```

## Complexity

- **Time**: O(n³) — 双 for 选种子 O(n²), 每次 check 沿链匹配 + 加法 O(n).
- **Space**: O(n) recursion depth + string 加法的 sum 长度.

## 易错点

- **Leading-0 双层都要拦**: 第一段 / 第二段都要查. 漏任一层都会接受 "01..." 这种非法序列. 单字符 "0" 必须放过 — 用 `i > 1` / `j > i + 1` 精准捕捉.
- **必须 string 加法 (C++)**: 输入 35 位远超 long long. 用 stoll 一定爆. Python/JS 原生大整数, 但移植到 C++ 必须手写大数加法.

## 相关题目

- [0131. Palindrome Partitioning](../0131-palindrome-partitioning/README.md) — 切割回溯, 段内约束 (回文)
- [0093. Restore IP Addresses](../0093-restore-ip-addresses/README.md) — 切割回溯, 段内约束 (IP) + 固定段数
- [0415. Add Strings](../../04-string/0415-add-strings/README.md) — 纯大数加法模板题, 这题的 addStrings 就是本题
- 0067\. Add Binary (待补) — 大数加法的 base-2 版本
- 0989\. Add to Array-Form of Integer (待补) — 大数加法但一边是 int, 一边是 vector<int>
- 0842\. Split Array into Fibonacci Sequence (待补) — 几乎是本题, 但要求返回切法不是判定; 同款模板
