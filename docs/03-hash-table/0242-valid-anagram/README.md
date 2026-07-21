# 0242. Valid Anagram / 有效的字母异位词

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Table, Counting Array, String · 哈希表, 计数数组, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/valid-anagram/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Anagram iff same letter multiset** → **26-int counting array**: `++` for s, `--` for t, all zero ⇒ true.
>
> **中文**: **字母多重集相同** → **26 长计数数组**, s 加 t 减, 全 0 即异位词.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给两个小写字符串 `s, t`. 判断 t 是否为 s 的**字母异位词** (字母组成相同, 顺序不限).

**中文**: 判异位词. 仅小写英文.

## Key Insights

1. **🔑 哈希的轻量版 = 计数数组 (固定字符集) / Counting array = hash table's lite version**

    "哈希表" 不一定要 `unordered_map`. 当 **key 是固定小集合** (这里 26 个小写字母), **数组就是最快的哈希**:

    | | 计数数组 `vector<int>(26)` | `unordered_map<char,int>` |
    |---|---|---|
    | 访问 | 数组下标, **几个时钟周期** | 哈希 + 链表跳转, 慢一阶 |
    | 内存 | 26 × 4 = 104 字节 | 几百字节 + 哈希结构开销 |
    | 适用 | **字符集有限** (ASCII / 小写) | **任意 key** (Unicode / 复杂对象) |

    > **看到"仅小写字母" 或"仅 ASCII"** → 优先用 26 / 128 / 256 长数组. 性能 + 简洁双赢.

2. **🔑 +1/-1 trick: 一次过, 不用两遍比较 / Add-then-subtract — no second pass to compare**

    朴素思路: 给 s 算一个频次表, 给 t 算一个, 然后比较两个表. 用 **+1/-1 技巧** 只需要一个表:

    ```cpp
    for c in s: cnt[c]++;
    for c in t: cnt[c]--;
    全 0 ⇒ 异位词
    ```

    若 t 多了什么字符 → 对应位置变负; t 少了什么 → 对应位置仍为正. **任何非 0 都说明对不上**.

    > 这是**消去式哈希**思想: 让"相同" 用"相互抵消" 体现. 同样思路也用在 0136 Single Number (用 XOR 抵消).

3. **🔑 字符 → 下标: `c - 'a'` / Map char to index**

    `'a' - 'a' = 0`, `'b' - 'a' = 1`, ..., `'z' - 'a' = 25`. **零开销** 的字符 → 整数映射, 计数数组的标配.

    Python/JS 没有"字符" 类型, 等价: `ord(c) - ord('a')` / `c.charCodeAt(0) - 97`.

4. **🔑 早退优化: 长度不同直接 false / Early-return on length mismatch**

    Yang 当前代码没加, 但工程上常加: `if (s.size() != t.size()) return false;` — 长度不同**必不可能** 是异位词. 省一次扫.

    > 不加也对 (-- 后会有负值), 加上更快. 面试时**提一句** 显示考虑.

5. **复杂度 O(n) 时间, O(1) 空间 / Linear time, constant space**

    `n = |s| + |t|`. 计数数组**26 长固定** → O(1) 空间, 不随 n 变.

    对比"排序后比较" 的 O(n log n) 方法 — **慢一阶, 但 O(1) 空间**. 计数数组**全方位胜出** 当字符集小.

6. **扩展: 字符集变大怎么办? / What if charset grows?**

    | 字符集 | 数据结构 |
    |---|---|
    | 小写英文 (26) | `vector<int>(26)` |
    | 全 ASCII (128) | `vector<int>(128)` |
    | 完整 Unicode | **`unordered_map<int,int>`** (码点数千万 — 数组开销爆炸) |

    > 进阶题 0049 Group Anagrams: 按"频次签名" 当 key 分组, 同款思路.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isAnagram(string s, string t) {
            if (s.size() != t.size()) return false;                  // 早退 (可选)
            vector<int> cnt(26, 0);
            for (char c : s) cnt[c - 'a']++;                         // s 加
            for (char c : t) cnt[c - 'a']--;                         // t 减
            for (int x : cnt) if (x != 0) return false;              // 全 0 ⇒ 异位词
            return true;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def isAnagram(self, s: str, t: str) -> bool:
            # 最 Pythonic 的写法: Counter 直接重载了 ==
            # Counter 是 dict 的子类, 自动统计频次. C++ 等价: unordered_map<char,int> 然后比较两个 map
            # O(n) 时间 + O(k) 空间, k = 字符集大小
            return Counter(s) == Counter(t)

        # —— 备选: 手动计数数组, 跟 C++ 一比一对应 ——
        def isAnagram_manual(self, s: str, t: str) -> bool:
            if len(s) != len(t):
                return False
            cnt = [0] * 26
            for c in s:
                cnt[ord(c) - ord('a')] += 1     # ord('a') = 97. Python 没"字符 - 字符" 运算, 必须 ord()
            for c in t:
                cnt[ord(c) - ord('a')] -= 1
            return all(x == 0 for x in cnt)     # any 的反面. C++ 等价: 全 0 才 true
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @param {string} t
     * @return {boolean}
     */
    var isAnagram = function(s, t) {
        if (s.length !== t.length) return false;
        // JS 没有"原生 Counter". 用普通数组 + charCodeAt 模拟计数表
        const cnt = new Array(26).fill(0);
        // 'a'.charCodeAt(0) = 97 — 等价 C++ 'a'. 减掉得到 0..25 下标
        const a = 'a'.charCodeAt(0);
        for (const c of s) cnt[c.charCodeAt(0) - a]++;
        for (const c of t) cnt[c.charCodeAt(0) - a]--;
        // .every 是数组方法 — 全部满足返 true. 比手写 for 简洁
        return cnt.every(x => x === 0);
    };
    ```

## Complexity

- **Time**: O(n) — 两次扫字符串 + 一次扫 26 长数组.
- **Space**: O(1) — 26 长固定计数数组 (若 Unicode 则 O(k), k = 不同字符数).

## 相关题目

- [0383. Ransom Note](../0383-ransom-note/README.md) — 同款计数数组, "subset" 而非 "equal"
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — 频次签名 / 排序签名当 hash key 分组
- 0438\. Find All Anagrams in a String (待补) — 滑窗 + 计数数组, 实时维护 anagram
- [0567. Permutation in String](../../05-two-pointers/0567-permutation-in-string/README.md) — 滑窗 + 计数
- 0387\. First Unique Character in a String (待补) — 计数数组 + 一次扫
- 0136\. Single Number (待补) — XOR 抵消 (同款"消去思想", 不同操作)
