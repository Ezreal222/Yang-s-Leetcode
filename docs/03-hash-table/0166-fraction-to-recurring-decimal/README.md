# 0166. Fraction to Recurring Decimal / 分数到小数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Map, Math, Simulation, Long Division · 哈希表, 数学, 模拟, 长除法
    - **Link**: [LeetCode](https://leetcode.com/problems/fraction-to-recurring-decimal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Convert `num/den` to decimal string with `(...)` around the repeating part** → **simulate long division**; hash map `remainder → position in result`; when a remainder repeats, insert `(` at the recorded position and `)` at end.
>
> **中文**: **`num/den` 转小数串, 循环节用 `(...)` 包起来** → **模拟长除法**; 哈希 `余数 → 结果串位置`; 余数重复出现 = 循环开始, 在记录位置插 `(`, 末尾加 `)`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给整数 `numerator` 和 `denominator`. 返回**其小数表示的字符串**. 若小数部分是**循环小数**, 循环节用**括号** 包起来 (如 `0.1(6)` 表示 1/6).

**中文**: 分数变小数字符串, 循环节包括号.

## Key Insights

1. **🔑 灵魂洞察: 循环节 ⇔ **余数重复** / Recurring digits ⇔ repeating remainder**

    长除法: 每步 `remainder × 10`, 得下一位, 得新余数. 关键事实:

    > **同样的余数**出现两次 → 后续所有步骤**必然重复上次的过程** → 循环节从上次出现开始.

    因为余数 ∈ `[0, den - 1]`, 只有 den 种可能. 若无限扩展, **鸽巢原理**保证余数**必然重复** → 无理数不可能 (有理分母都有循环节).

    > **这跟 [0202 Happy Number](../0202-happy-number/README.md) / [0142 Cycle II](../../02-linked-list/0142-linked-list-cycle-ii/README.md) 同源**: "有限状态 + 确定性下一步" 必有环. **算法通用性再体现**.

2. **🔑 hash map `余数 → 结果串位置` / Hash map: remainder → position**

    Yang 的关键设计: value 不是"次数" 或"下一位", 是**"这个余数第一次出现时, 结果串已写到哪一位"**.

    ```
    seen[rem] = res.size()      // 当前 res 长度 = 这一位小数即将写入的位置
    ```

    当再次遇到同 rem, `seen[rem]` 就是**循环节起点**位置. 直接 `res.insert(seen[rem], "(")` 插左括号 + 末尾 append `)`.

    > **不能只记"看过没" 用 set** — 我们要**位置**才能插括号. **`unordered_set → unordered_map` 的一步升级** 靠"要不要记位置".

3. **🔑 符号处理: XOR 判异号 / XOR for sign detection**

    ```cpp
    if ((numerator < 0) ^ (denominator < 0)) res += "-";
    ```

    - `numerator < 0` 是 bool → 0/1.
    - **XOR = "异号"**: 恰好一个为负 → true → 结果负.
    - **优雅**替代 `(a < 0 && b > 0) || (a > 0 && b < 0)`.

    > **两 bool 的 XOR = 异或 = "不同"** — 记住这个符号处理套路.

4. **🔑 `abs(INT_MIN)` 溢出 → cast 到 long long / `abs(INT_MIN)` overflows**

    C 里 `INT_MIN = -2^31`, `-INT_MIN = 2^31` 超过 `INT_MAX = 2^31 - 1` → **UB**. Yang 的防御:

    ```cpp
    long long num = abs((long long)numerator);      // 先 cast 再 abs
    long long den = abs((long long)denominator);
    ```

    - `(long long)numerator` 把 int 提升到 64 位 → abs 就在 64 位下算 → 无溢出.
    - 后续所有运算都用 long long → 中间 `rem *= 10` 也不溢出.

    > **`abs(INT_MIN)` 是 C++ 的经典陷阱**. 涉及 abs 时**先提升类型**. 老手必踩过这船.

5. **🔑 长除法模拟 / Long-division simulation**

    ```
    while rem != 0:
        rem *= 10
        digit = rem / den
        res += digit
        rem %= den
    ```

    "余数乘 10 → 除 den 得下一位 → 新余数" — 就是小学数学的长除法算法.

    整合循环检测:

    ```cpp
    while (rem != 0) {
        if (seen.count(rem)) {                // 环!
            res.insert(seen[rem], "(");
            res += ")";
            break;
        }
        seen[rem] = res.size();                // 记位置
        rem *= 10;
        res += to_string(rem / den);
        rem %= den;
    }
    ```

    > **注意 `seen[rem] = res.size()`** 是在**下一位写入前** 记录. 位置刚好指向"下一位要写入的地方" = "循环节开始位置". 想清楚这个 off-by-one.

6. **🔑 边界: 0 / 整除 / 早退 / Edge cases**

    - `numerator == 0` → 直接 "0" (不管 den).
    - 整除后 `rem == 0` → 无小数部分, 只返整数部分.
    - 负号处理**在 abs 之前** 才不影响后续.

7. **🔑 跟 [0202 Happy Number](../0202-happy-number/README.md) 关系 / vs 0202**

    | | 0202 Happy Number | **0166 (本题)** |
    |---|---|---|
    | 迭代 | `n = squareSum(n)` | `rem = (rem * 10) % den` |
    | 状态空间 | ~ [1, 999] | [0, den-1] |
    | 判环 | hash set 存 seen 数 | **hash map 存 (seen 余数 → 位置)** |
    | 目的 | 判"到 1 没" | **找循环节位置** |

    > **hash 判环** 家族: 判存在用 set, 需要位置/上一步用 map. 一族问题.

8. **复杂度 / Complexity**

    - **Time**: O(den) 最坏 — 至多 den 个不同余数, 无穷循环也在 den 步内进环.
    - **Space**: O(den) — hash map.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        string fractionToDecimal(int numerator, int denominator) {
            if (numerator == 0) return "0";
            string res;
            // XOR 判异号
            if ((numerator < 0) ^ (denominator < 0)) res += "-";
            // 先 cast 再 abs, 防 INT_MIN 溢出
            long long num = abs((long long)numerator);
            long long den = abs((long long)denominator);
            // 整数部分
            res += to_string(num / den);
            long long rem = num % den;
            if (rem == 0) return res;                                 // 整除, 无小数
            res += ".";
            // hash: 余数 → 该余数首次出现时, res 已写到哪
            unordered_map<long long, int> seen;
            while (rem != 0) {
                if (seen.count(rem)) {                                // 余数重复 = 循环
                    res.insert(seen[rem], "(");                       // 循环起点插 (
                    res += ")";
                    break;
                }
                seen[rem] = res.size();                               // 记位置 (下一位写入前)
                rem *= 10;
                res += to_string(rem / den);                          // 下一位
                rem %= den;                                           // 新余数
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def fractionToDecimal(self, numerator: int, denominator: int) -> str:
            if numerator == 0: return "0"
            # Python int 是任意精度, 无 INT_MIN 溢出隐患 — 不用手动 cast
            # ^ 对 bool 也是 XOR (True/False 会自动转 int 参与运算)
            sign = "-" if (numerator < 0) ^ (denominator < 0) else ""
            n, d = abs(numerator), abs(denominator)
            # divmod(n, d) 一步返 (商, 余) — 比分开写 n // d 和 n % d 简洁
            q, r = divmod(n, d)
            res = sign + str(q)
            if r == 0: return res
            res += "."
            seen: dict[int, int] = {}
            # Python 的 list 拼接不如 C++ string += 高效, 用 list + join 更 Pythonic
            # 这里保持跟 C++ 一致的 string 累加 (可读) — 长度都是 O(den) 无所谓
            while r:
                if r in seen:
                    idx = seen[r]
                    res = res[:idx] + "(" + res[idx:] + ")"
                    break
                seen[r] = len(res)
                r *= 10
                res += str(r // d)
                r %= d
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} numerator
     * @param {number} denominator
     * @return {string}
     */
    var fractionToDecimal = function(numerator, denominator) {
        if (numerator === 0) return "0";
        // JS 数字是 double, 安全整数 2^53, 覆盖 32-bit int × 10 空间 — 无溢出
        // ^ 在 JS 是位运算, 对 bool 会先转 int (false=0, true=1), 用于异号判定 OK
        let res = (numerator < 0) ^ (denominator < 0) ? "-" : "";
        // Math.abs 对负数取正. JS 里 Math.trunc 更接近 C++ 的整数除 (对 int/int 都行)
        const n = Math.abs(numerator), d = Math.abs(denominator);
        res += Math.trunc(n / d).toString();
        let r = n % d;
        if (r === 0) return res;
        res += ".";
        const seen = new Map();
        while (r !== 0) {
            if (seen.has(r)) {
                const idx = seen.get(r);
                res = res.slice(0, idx) + "(" + res.slice(idx) + ")";
                break;
            }
            seen.set(r, res.length);
            r *= 10;
            res += Math.trunc(r / d).toString();
            r %= d;
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(den) 最坏 — 循环节 ≤ den 步.
- **Space**: O(den) — hash map.

## 相关题目

- [0202. Happy Number](../0202-happy-number/README.md) — 同族"数字迭代 + 判环"
- [0142. Linked List Cycle II](../../02-linked-list/0142-linked-list-cycle-ii/README.md) — Floyd 判环
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 哈希分桶
- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组
- 0043\. Multiply Strings (待补) — 大数乘法模拟
- 0415\. Add Strings (已存) — 大数加法
- 0592\. Fraction Addition and Subtraction (待补) — 分数运算
