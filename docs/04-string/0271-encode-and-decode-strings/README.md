# 0271. Encode and Decode Strings / 字符串的编码与解码

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: String, Design, Serialization · 字符串, 设计, 序列化
    - **Link**: [LeetCode](https://leetcode.com/problems/encode-and-decode-strings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Encode list of arbitrary strings into one** → **length-prefix protocol** `len#str` per item. Decode = read digits till `#`, skip `#`, take exactly `len` chars, repeat.
>
> **中文**: **任意字符串列表 → 单串** → **长度前缀协议** `长度#内容`. 解码先读到 `#` 取长度, 再读 len 个字符, 循环.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 设计 `encode(vector<string>) → string` 和 `decode(string) → vector<string>`. 字符串可含**任意字符** (包括分隔符 / 数字 / Unicode). 要求 `decode(encode(strs)) == strs`.

**中文**: 设计编解码, 内容任意.

## Key Insights

1. **🔑 关键困难: 字符串可含任意字符, **没有"安全分隔符"** / The hard part: no universally safe delimiter**

    朴素思路 "用 `,` / `|` / `#` 分隔" **必错** — 因为内容里可能含这些字符. 比如:

    ```
    strs = ["hello,world", "foo"]
    encoded = "hello,world,foo"      // 拆回时变成 3 个串, 错!
    ```

    > **任何"分隔符" 方案都假设内容里没有该字符 — 这是脆弱的契约**. 工程上要么转义 (`\,`), 要么换思路.

2. **🔑 工业标准: 长度前缀 (length-prefix) / Length-prefix encoding**

    每个串前面**先告诉解码器它有多长**, 解码器读完那么多字节就停, **不管内容是什么**:

    ```
    "abc" → "3#abc"
    "ab#cd" → "5#ab#cd"            // 内容里的 # 完全无害
    ["abc", "ab#cd"] → "3#abc5#ab#cd"
    ```

    解码:
    1. 从 `i` 开始往后扫到第一个 `#` → 数字部分
    2. parse 出 `len`
    3. 跳过 `#`, 取 `len` 个字符 → 一个串
    4. `i` 跳到下一个串的起点, 循环

    > **TCP / Protocol Buffers / HTTP Content-Length** 都用这套思路. **数据长度独立于内容含义** — 这是优雅的隔离.

3. **🔑 为啥"扫到第一个 `#`" 安全? / Why scanning for the first `#` is safe**

    内容里可能含 `#`, 但 **"长度前缀的数字部分"** 一定都是 ASCII 数字 `'0'..'9'`. **数字里不会有 `#`** → 第一个 `#` 必是分隔符, 不会误判.

    > 分隔符在**有结构约束的位置** 用就是安全的. **隔离思想**: 数字段绝不重叠内容.

4. **🔑 解码的索引推进: `i = j + 1 + len` / Index arithmetic**

    Yang 的核心一行:

    ```
    i = j + 1 + len
    //  ^   ^   ^
    //  #位置  跳#  跳内容
    ```

    - `j` 指向 `#`
    - `j + 1` 跳过 `#` (= 内容起点)
    - `+ len` 跳过整个内容 → 下一个长度数字的起点

    > **指针推进的一行**是这题的灵魂. 想清楚 `i` / `j` 的语义, 别 ± 1 错位.

5. **C++ 写法细节 / C++ idioms**

    - **`to_string(int)`** ↔ **`stoi(string)`**: 数字 ↔ 字符串. `<string>` 头文件.
    - **`s.substr(pos, len)`**: 从 `pos` 开始取 `len` 个字符. 跟 `pos..pos+len-1` 闭区间等价.
    - **`encoded += ...`**: `string` 的 `+=` 复用底层 buffer (类似 C++ `std::vector`), 摊销 O(1).

6. **复杂度 / Complexity**

    - **encode**: O(L) where L = 总字符数 — 一遍写完.
    - **decode**: O(L) — 每个字符最多被访问 O(1) 次 (扫数字一次, 拷内容一次).
    - **空间**: O(L) — 输出.

## Solution

=== "C++"
    ```cpp
    class Codec {
    public:
        // Encodes a list of strings to a single string.
        string encode(vector<string>& strs) {
            string encoded;
            for (auto& s : strs) {
                encoded += to_string(s.size()) + '#' + s;            // 长度 # 内容
            }
            return encoded;
        }

        // Decodes a single string to a list of strings.
        vector<string> decode(string s) {
            vector<string> ans;
            int i = 0;
            while (i < (int)s.size()) {
                int j = i;
                while (s[j] != '#') j++;                             // 扫到分隔符
                int len = stoi(s.substr(i, j - i));                  // [i, j) 是数字
                ans.push_back(s.substr(j + 1, len));                 // [j+1, j+1+len) 是内容
                i = j + 1 + len;                                     // 跳到下一段
            }
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    class Codec:
        def encode(self, strs: list[str]) -> str:
            # f-string 拼长度 # 内容. ''.join(...) 一次性拼接, 比 += 累加更快
            # (Python string 不可变, += 每次拷贝整个串, O(n²); join 是 O(n))
            return ''.join(f'{len(s)}#{s}' for s in strs)

        def decode(self, s: str) -> list[str]:
            ans = []
            i, n = 0, len(s)
            while i < n:
                j = s.index('#', i)             # 从 i 开始找第一个 '#'. C++ 等价: 手写 while 扫
                length = int(s[i:j])            # [i:j) — Python 切片同 C++ substr 半开
                ans.append(s[j + 1 : j + 1 + length])
                i = j + 1 + length              # 同 C++ 推进逻辑
            return ans
    ```

=== "JavaScript"
    ```javascript
    var Codec = function() {};

    /**
     * @param {string[]} strs
     * @return {string}
     */
    Codec.prototype.encode = function(strs) {
        // map + join 比 for 循环 += 干净. JS string 也是不可变, += 是 O(n²) 隐患
        // 模板字面量 `${len}#${s}` 拼长度 # 内容
        return strs.map(s => `${s.length}#${s}`).join('');
    };

    /**
     * @param {string} s
     * @return {string[]}
     */
    Codec.prototype.decode = function(s) {
        const ans = [];
        let i = 0;
        while (i < s.length) {
            // .indexOf('#', i) 从 i 开始找 — 等价 Python s.index('#', i) / C++ 手写扫
            const j = s.indexOf('#', i);
            const len = parseInt(s.substring(i, j), 10);    // [i, j) 数字段
            ans.push(s.substring(j + 1, j + 1 + len));      // [j+1, j+1+len) 内容
            i = j + 1 + len;
        }
        return ans;
    };
    ```

## Complexity

- **Time**: O(L) — 编码解码各扫一遍, L = 总字符数.
- **Space**: O(L) — 输出.

## 相关题目

- 0297\. Serialize and Deserialize Binary Tree (待补) — 树结构序列化, 同款"自定义协议"
- 0428\. Serialize and Deserialize N-ary Tree (待补) — 多叉树版
- 0449\. Serialize and Deserialize BST (待补) — BST 利用性质压缩
- [0049. Group Anagrams](../../03-hash-table/0049-group-anagrams/README.md) — 频次签名也是"自定义编码"
- 0535\. Encode and Decode TinyURL (待补) — 设计题, 不同 trade-off (hash vs 计数 vs 随机)
