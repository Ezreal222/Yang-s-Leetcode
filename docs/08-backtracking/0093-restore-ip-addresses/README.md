# 0093. Restore IP Addresses / 复原 IP 地址

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Backtracking, String · 回溯, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/restore-ip-addresses/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a string of digits, return **all** valid IPv4 addresses formed by inserting three dots. Each segment must be 0-255, no leading zeros (except "0" itself).

**中文**: 给一个数字字符串, 返回所有合法的 IPv4 地址 (插 3 个点). 每段范围 0-255, 不能有前导 0 (除非段就是 "0").

## 思路

### Core idea

**[0131](../0131-palindrome-partitioning/README.md) + 强制切 4 段 + 段内合法性变成 IP 校验**. 同款切割回溯 (`start` 推进, `end` 在 `[start, n-1]` 滑), 只是:

1. 终止条件 = 切了 **正好 4 段** 且 `start == s.size()`.
2. 每段 `isValid` 改成 IP 校验 (长度 ≤ 3, 无前导 0, 数值 ≤ 255).
3. 早停: `path.size() > 4` 直接 return.

### Key Insights

1. **0131 切割模板 + 两个约束 / Partition template, two extra constraints**

    | 项 | 0131 | 0093 |
    |---|---|---|
    | 段数 | 任意, 切完即收 | **必须 4 段** |
    | isValid | 回文 | IP 段: 1-3 字符, 无前导 0, ≤ 255 |
    | 终止 | `start == n` | `path.size() == 4 && start == n` (双条件) |
    | 早停 | 无 | `path.size() > 4` |

    其它一切 (push/pop, continue 跳非法, end+1 推进) 都跟 0131 一字不差.

2. **IP 段三道校验 / Three IP-octet checks**

    ```cpp
    if (r - l > 2) return false;          // 长度 > 3 不行
    if (s[l] == '0' && r > l) return false; // 前导 0 (但单独 "0" 合法 — 由 r > l 排除单字符)
    int num = stoi(s.substr(l, r - l + 1));
    return num <= 255;
    ```
    - **长度判断在最前**: 不仅是合法性, 也保护 stoi 不读超大数. 没这个判断 stoi("1234") = 1234, 还能比 255 大, 但读 "999999999" 就接近 int 边界.
    - **前导 0 的细节**: "0" 是合法 (一段就是 0), "01" 不合法. 用 `s[l] == '0' && r > l` 精准捕捉.

3. **双条件终止 / Two-part terminal check**

    `path.size() == 4 && start == s.size()` — 两条都满足才收. 单条都不行:

    - 只检 `path.size() == 4`: 可能 4 段了但字符串没切完, 漏字符 → 非法 IP.
    - 只检 `start == s.size()`: 可能正好切完但段数 ≠ 4.

4. **`path.size() > 4` 早停: soft prune / Defensive overflow stop**

    没这一行也对, 但探索会多走一两层无效递归才被 collect 判 false 拦下. 加了之后, 任何会让 path 超过 4 段的递归在入口就 return.

    **可以更紧**: 把它跟 collect 合并:
    ```cpp
    if ((int)path.size() == 4) {
        if (start == (int)s.size()) res.push_back(/*join*/);
        return;   // ← 4 段了无论如何都不该再切
    }
    ```
    更优雅, 一次判断同时处理两种 4 段情况.

5. **拼接策略: vector<string> 比 string + "."更简单 / Why join at the end**

    Yang 选 `vector<string> path`, 4 段都收齐了再 `path[0]+"."+...+path[3]` 拼. 另一种思路是 `string path`, push 时带 ".", 收果实时去掉末尾点 — 多一个边界处理, 不划算. `vector` 版可以直接拿 `.size()` 当段数, 收益还多.

6. **复杂度 / Bounded combinatorics**

    最多在 n-1 个间隔里选 3 个插点位置, 共 C(n-1, 3) 种切法. n ≤ 20 时是 C(19, 3) = 969 种, 每种 O(n) 校验 + 拼接 → 微秒级.

### 一句话总结

**0131 切割模板 + 4 段 + IP 段校验. `start` 推进, `end` 在 `[start, n-1]` 滑, 非法 segment continue, 4 段且消费完才收, 超 4 段早停.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<string> res;
        vector<string> path;

        bool isValidIP(const string& s, int l, int r) {
            if (r - l > 2) return false;                          // 长度 > 3 不行
            if (s[l] == '0' && r > l) return false;               // 前导 0 不行 (但单个 0 允许)
            int num = stoi(s.substr(l, r - l + 1));
            return num >= 0 && num <= 255;
        }

        void backtrack(const string& s, int start) {
            if ((int)path.size() == 4 && start == (int)s.size()) {
                res.push_back(path[0] + "." + path[1] + "." + path[2] + "." + path[3]);
                return;
            }
            if ((int)path.size() > 4) return;                     // 早停: 段数超了
            for (int end = start; end < (int)s.size(); end++) {
                if (!isValidIP(s, start, end)) continue;          // 同 0131: continue 让 push/pop 紧邻
                path.push_back(s.substr(start, end - start + 1));
                backtrack(s, end + 1);
                path.pop_back();
            }
        }
        vector<string> restoreIpAddresses(string s) {
            backtrack(s, 0);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def restoreIpAddresses(self, s: str) -> list[str]:
            res, path = [], []
            n = len(s)
            def is_valid(l: int, r: int) -> bool:
                if r - l > 2:
                    return False
                if s[l] == '0' and r > l:                        # 前导 0
                    return False
                return int(s[l:r + 1]) <= 255                    # Python int 无溢出, 不需要先判长度也行
            def backtrack(start: int):
                if len(path) == 4 and start == n:
                    res.append('.'.join(path))                    # str.join: 等价 C++ 拼接, 但简洁很多
                    return
                if len(path) > 4:
                    return
                for end in range(start, n):
                    if not is_valid(start, end):
                        continue
                    path.append(s[start:end + 1])
                    backtrack(end + 1)
                    path.pop()
            backtrack(0)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string[]}
     */
    var restoreIpAddresses = function(s) {
        const res = [], path = [];
        const isValid = (l, r) => {
            if (r - l > 2) return false;
            if (s[l] === '0' && r > l) return false;             // 前导 0
            const num = +s.slice(l, r + 1);                       // +str: 字符串转 number, 等价 Number()
            return num <= 255;
        };
        const backtrack = (start) => {
            if (path.length === 4 && start === s.length) {
                res.push(path.join('.'));                         // Array.join('.'): 等价 Python '.'.join(...)
                return;
            }
            if (path.length > 4) return;
            for (let end = start; end < s.length; end++) {
                if (!isValid(start, end)) continue;
                path.push(s.slice(start, end + 1));
                backtrack(end + 1);
                path.pop();
            }
        };
        backtrack(0);
        return res;
    };
    ```

## Complexity

- **Time**: O(C(n-1, 3) × n) ≤ O(n⁴) 上界 — 三个切点位置, 每种 O(n) 拼接.
- **Space**: O(1) recursion depth (最多 4 层) + path 4 段.

## 易错点

- **前导 0: "0" 合法, "01" 不合法**: 边界条件别想成"含 0 就不行". 单字符 "0" 是合法 IP 段. 判断写成 `s[l] == '0' && r > l` (长度 > 1 时才禁).
- **长度 > 3 必须先拦**: 不仅是合法性, 也保护 stoi 不读巨大数. 长度判断必须在 stoi 之前.
- **终止必须双条件**: `path.size() == 4` 和 `start == s.size()` 都要. 只一个会漏字符或漏段.
- **`path.size() > 4` 早停**: 不加也能过 (collect 时段数自动判), 但加了减少无效递归. Yang 加了 — 学回去.
- **`end + 1` 不是 `start + 1`**: 同 0131. 下一段从**当前段的结尾后**开始.
- **`isValidIP` 里 `r - l > 2` 是"长度大于 3"**: 长度 = `r - l + 1`. `r - l > 2` 等价 `length > 3`. 别看错.
- **拼接的 "." 数量**: 4 段 IP 用 3 个 ".". 用 join 比手写 `+ "."` 少错.
- **JS `+s.slice(...)` 是 unary plus 转 number**: 不要写成 `parseInt(s.slice(...))` — `parseInt("08")` 在某些环境会按八进制解析 (历史包袱), unary plus 始终十进制安全.

## 相关题目

- [0131. Palindrome Partitioning](../0131-palindrome-partitioning/README.md) — 同切割模板, segment 校验换成回文
- [0077. Combinations](../0077-combinations/README.md) — 同款 startIndex 推进
- [0017. Letter Combinations of a Phone Number](../0017-letter-combinations-of-a-phone-number/README.md) — 同回溯模板, 但多集合每位一选 (非切割)
- 0468. Validate IP Address (待补) — 只判合法性, 不分段; 这题的 `isValidIP` 的扩展版 (IPv4 + IPv6)
- 0306. Additive Number (待补) — 同款切割但每段要满足斐波那契式约束
- 0282. Expression Add Operators (待补) — 数字串里插运算符, 同切割回溯 + 表达式求值
