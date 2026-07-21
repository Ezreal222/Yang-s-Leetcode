# 0249. Group Shifted Strings / 移位字符串分组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Map, Canonical Key, String, Modular Arithmetic · 哈希表, 规范化, 字符串, 模运算
    - **Link**: [LeetCode](https://leetcode.com/problems/group-shifted-strings/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Group strings that are cyclic shifts of each other** → **canonical key = sequence of adjacent-char diffs `mod 26`**; strings share the shift group ⇔ same diff signature. Hash map `key → bucket`.
>
> **中文**: **按"整体移位等价" 分组** → **canonical key = 相邻字符差分 mod 26 的序列**; 同 shift group ⇔ 同差分签名. 哈希分桶.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串数组 `strings`. 把**可以通过整体循环移位** 得到的字符串分为一组. 例如 `"abc" → "bcd" → ... → "xyz"` 是同一组 (每个字符 +1 得到下一个).

**中文**: 把"能通过统一 shift 得到" 的字符串分组. shift = 每个字符 +k (mod 26).

## Key Insights

1. **🔑 灵魂洞察: 同 shift group ⇔ 相邻差分序列相同 / Key insight: same group ⇔ same diff sequence**

    观察:

    ```
    "abc" 的相邻差:   b-a=1, c-b=1     → (1, 1)
    "bcd" 的相邻差:   c-b=1, d-c=1     → (1, 1)   ✅ 同签名 → 同组
    "xyz" 的相邻差:   y-x=1, z-y=1     → (1, 1)   ✅ 同组

    "abd" 的相邻差:   b-a=1, d-b=2     → (1, 2)   ❌ 不同组
    ```

    **shift 等价 = 对每个字符加同一常量 k mod 26** → 相邻差**不变** (因为 (x+k) - (y+k) = x - y).

    > **"整体线性变换 → 保持相对差"** 是数学观察. **"绝对量" 换成"相对量" 就是 canonical**. 找到这一步就赢了 60%.

2. **🔑 canonical key = 差分序列, 编码为字符串 / Encode diff sequence as string**

    Yang 的 helper:

    ```cpp
    for i in 1..len-1:
        diff = (s[i] - s[i-1] + 26) % 26
        key += to_string(diff) + ","
    ```

    - 差分 len-1 个数.
    - 每个数编码 + `,` 分隔.
    - 最终 key 是形如 `"1,1,"` 或 `"1,2,"` 的字符串.

    > **哈希 key 必须是"可比较的原子类型"** — 字符串是万能选择. 对 map 来说, 字符串比 `vector<int>` 更快.

3. **🔑 `(diff + 26) % 26` 处理 wrap around / Modular wrap for cyclic shift**

    C++ 中 `char - char` 是 int, 可能是**负数**:

    ```
    "az" → z - a = 25
    "za" → a - z = -25
    ```

    但 `"az"` 移位 1 得 `"ba"`, 差是 `a - b = -1 = 25 (mod 26)`. **同组要求 mod 26 相等**, 直接 `%` 在 C++ 里对负数返负 → **必须先 `+ 26`** 再 `% 26` 归到 `[0, 25]`.

    ```
    (-1 + 26) % 26 = 25 ✅
    (-1) % 26 = -1 ❌ (C++ 语义, 用作 hash key 会跟正 -1 撞车)
    ```

    > **模运算处理负数** 是 C 语系的必踩坑. 记住 **`(x + M) % M`** 强制正化.

4. **🔑 分隔符 `,` 的必要性 / Delimiter is necessary**

    若不加分隔, `"1,10"` 和 `"11,0"` 都会编码成 `"110"` → 撞 key. **多位数字**必须分隔.

    > **序列化为 string 时**总要问: **不同输入会不会编码相同?** 加个分隔符是**便宜的隔离**.

5. **🔑 单字符 case: 空 key / Single char case**

    `s.size() == 1` → for 循环不进 → key = `""`. 所有单字符字符串**共享空 key**, 分到一组. ✅ 符合语义 ("a" 和 "z" 都是"shift 兼容的单字符").

    > **边界 case 自动 handle** 说明 canonical 设计**优雅**. 若单字符要特殊处理, 说明 key 设计不好.

6. **🔑 跟 [0049 Group Anagrams](../0049-group-anagrams/README.md) 是**同一模板** / Same template as 0049**

    | | 0049 | **0249 (本题)** |
    |---|---|---|
    | 等价关系 | 字母多集相同 | shift 等价 |
    | canonical key | 排序后字符串 / 频次签名 | **差分 mod 26 序列** |
    | 数据结构 | `unordered_map<string, vector<string>>` | 同 |
    | 展桶 | structured binding `for (auto& [_, g] : m)` | 同 |

    > **哈希分桶模式**: `for x: bucket[canonical(x)].push(x)`. 90% 的"分组" 题都是这套. **难点只在 canonical 怎么选**.

7. **复杂度 O(N × L) 时间, O(N × L) 空间 / Linear in total chars**

    - N = 词数, L = 平均长度.
    - 每词生成 key O(L), 插入 map 平均 O(L) (hash).
    - Space: map 存所有原串.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<string>> groupStrings(vector<string>& strings) {
            unordered_map<string, vector<string>> groups;
            for (auto& s : strings) {
                groups[getKey(s)].push_back(s);                              // operator[] 自动建桶
            }
            vector<vector<string>> res;
            for (auto& [_, group] : groups) res.push_back(group);            // C++17 structured binding
            return res;
        }
    private:
        string getKey(const string& s) {
            string key;
            for (int i = 1; i < (int)s.size(); i++) {
                int diff = (s[i] - s[i - 1] + 26) % 26;                     // +26 处理负数
                key += to_string(diff) + ",";                               // 分隔防撞
            }
            return key;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def groupStrings(self, strings: list[str]) -> list[list[str]]:
            groups = defaultdict(list)      # defaultdict(list) 缺 key 自动建空 list, 同 C++ operator[]
            for s in strings:
                # tuple 生成器直接当 key — tuple 可哈希, list 不行. 无需自己编码为 string
                # ord('a') = 97. 差分 + 26 处理负数, % 26 归 [0, 25]
                key = tuple((ord(s[i]) - ord(s[i - 1])) % 26 for i in range(1, len(s)))
                # Python 的 % 对负数返正数 (跟数学定义一致), 不用手动 + 26 也对!
                # 例: (-1) % 26 == 25 in Python (C++ 里是 -1)
                groups[key].append(s)
            return list(groups.values())
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} strings
     * @return {string[][]}
     */
    var groupStrings = function(strings) {
        // JS Map 允许任意 key 类型, 但字符串 key 检索更快. 用 join(',') 编码差分序列
        const groups = new Map();
        for (const s of strings) {
            let key = '';
            for (let i = 1; i < s.length; i++) {
                // JS 的 % 对负数返负数 (跟 C++ 一致, 跟 Python 不同) → 必须 + 26
                const diff = (s.charCodeAt(i) - s.charCodeAt(i - 1) + 26) % 26;
                key += diff + ',';
            }
            if (!groups.has(key)) groups.set(key, []);
            groups.get(key).push(s);
        }
        return Array.from(groups.values());
    };
    ```

## Complexity

- **Time**: O(N × L) — N 词, L 平均长度.
- **Space**: O(N × L) — map 存原串.

## 相关题目

- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 同款"canonical key + 分桶", canonical = 排序 / 频次
- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组母题
- [0383. Ransom Note](../0383-ransom-note/README.md) — subset 判定
- [0349. Intersection of Two Arrays](../0349-intersection-of-two-arrays/README.md) — hash set 用法
- 0169\. Majority Element (待补) — Boyer-Moore 或 hash 计数
- [0567. Permutation in String](../../05-two-pointers/0567-permutation-in-string/README.md) — 滑窗 + 频次签名
- 0350\. Intersection of Two Arrays II (待补) — Counter 求"多集交"
