# 0049. Group Anagrams / 字母异位词分组

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Table, Sorting, String · 哈希表, 排序, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/group-anagrams/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Group strings by anagram class** → each class shares a **canonical key** (sorted string or 26-letter count signature) → hash map `key → bucket`, push each string into its bucket.
>
> **中文**: **按异位词分桶** → 同组共享一个**规范 key** (排序后字符串 / 频次签名) → `map[key].push_back(s)` 即聚类.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串数组. 把**字母异位词** 分到同组返回. 顺序任意.

**中文**: 异位词分组, 顺序无所谓.

## Key Insights

1. **🔑 哈希分桶模式: 等价类 → canonical key / Bucket trick: equivalence class via canonical key**

    任何"按规则分组" 的问题 → 套这个模板:

    ```
    for each item:
        key = canonical(item)        // 同等价类必给出相同 key
        groups[key].push_back(item)
    return groups.values()
    ```

    **唯一问题就是 "canonical 怎么算"**. 本题等价类是"异位词组", canonical 可以是:

    - **排序后字符串** ("eat", "tea", "ate" → 都得 "aet")
    - **频次签名** (26 个 int 编码成定长字符串, 如 "a1e1t1...")

    > **核心思维**: "怎么判断两个东西等价" 想清楚后, 翻译成"怎么生成一个所有等价物都相同的 key" — 哈希分桶就成立.

2. **🔑 `groups[key].push_back(s)` 利用 `operator[]` 自动建桶 / `operator[]` auto-creates the bucket**

    Yang 的代码**不需要** `if (groups.count(key)) ... else ...`:

    ```cpp
    groups[key].push_back(s);
    ```

    `unordered_map::operator[]` 行为: **key 不存在 → 默认构造 value** (这里 `vector<string>()`) 然后返回引用. 所以第一次见到一个 key 时, 空 vector 已经在那等着 push_back 了.

    > **C++ map 的小坑**: 这个特性让"查 + 插" 一行搞定, 但**读多写少** 场景容易意外创建空条目. 只读用 `.find()` 或 `.at()` 更安全.

3. **🔑 排序签名 vs 频次签名 / Sort vs count signature**

    | | **排序签名** (Yang) | **频次签名** |
    |---|---|---|
    | 单个 key | `O(k log k)` (sort) | `O(k)` (count + encode) |
    | 总时间 | `O(n · k log k)` | **`O(n · k)`** |
    | key 长度 | k | 固定 ~52 字节 (26 个数字 + 分隔) |
    | 代码量 | **3 行** | ~6 行 |
    | 字符集 | 任意 | **仅小写英文 / 固定字符集** |
    | 何时选 | k 小 / 字符集不固定 | k 大 / 性能敏感 |

    > Yang 选了**排序签名** — 短而美, n / k 都不极端时是最佳实践. **频次签名**在 k 大 (几千) 时省一个 log 因子.

4. **🔑 C++17 结构化绑定: `for (auto& [_, group] : groups)` / Structured bindings**

    展开桶时, Yang 用了 C++17 的 **structured bindings**:

    ```cpp
    for (auto& [_, group] : groups) ans.push_back(group);
    //         ^    ^
    //         key  value (引用)
    ```

    `_` 只是个**普通变量名** (不像 Python 的 `_` 那样真"丢弃") — 但**惯例上** 代表"我不用". 比写 `for (auto& kv : groups) ans.push_back(kv.second);` 干净.

    > 等价的"老式" C++14 写法: `for (auto& kv : groups) ans.push_back(kv.second);`. 知道两种都行.

5. **跟 [0242 Valid Anagram](../0242-valid-anagram/README.md) 的关系 / Relationship to 0242**

    | | 0242 | **0049 (本题)** |
    |---|---|---|
    | 输入 | **两个** 字符串 | **N 个** 字符串 |
    | 问题 | 是否异位词 | **分组** |
    | 手段 | 直接比较两个频次表 | **生成 canonical key 入桶** |
    | 核心 | "判等价" | "**找等价类标识符**" |

    > **同根问题**: 都关心"字母多重集". 0242 比一次, 0049 比 N² 次太慢 → 引入 canonical key 把比较降到 O(1).

6. **复杂度 / Complexity**

    设 n 是字符串数, k 是平均长度.

    - **时间**: `O(n · k log k)` — n 个串各排序一次. 频次签名版可降到 `O(n · k)`.
    - **空间**: `O(n · k)` — map 存所有原串.

## Solution

=== "C++"

    **排序签名 (Yang 风格, 简洁通用)**

    ```cpp
    class Solution {
    public:
        vector<vector<string>> groupAnagrams(vector<string>& strs) {
            unordered_map<string, vector<string>> groups;
            for (auto& s : strs) {
                string key = s;
                sort(key.begin(), key.end());                           // canonical: 排序后字符串
                groups[key].push_back(s);                               // operator[] 自动建空桶
            }
            vector<vector<string>> ans;
            for (auto& [_, group] : groups) ans.push_back(group);       // C++17 structured binding
            return ans;
        }
    };
    ```

    **频次签名 (省 log 因子, 仅小写英文)**

    ```cpp
    class Solution {
    public:
        vector<vector<string>> groupAnagrams(vector<string>& strs) {
            unordered_map<string, vector<string>> groups;
            for (auto& s : strs) {
                array<int, 26> cnt{};
                for (char c : s) cnt[c - 'a']++;
                // 编码成定长 key — 用 '#' 当分隔, 不同长度数字不会撞
                string key;
                for (int x : cnt) { key += to_string(x); key += '#'; }
                groups[key].push_back(s);
            }
            vector<vector<string>> ans;
            for (auto& [_, group] : groups) ans.push_back(group);
            return ans;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict

    class Solution:
        def groupAnagrams(self, strs: list[str]) -> list[list[str]]:
            # defaultdict(list): key 不存在时自动新建 list, 跟 C++ unordered_map::operator[] 的自动构造同款
            # 比 setdefault / dict.get 干净一截
            groups = defaultdict(list)
            for s in strs:
                # tuple(sorted(s)) 当 key — tuple 可哈希, list 不可
                # 等价 C++: sort(key.begin(), key.end()) + map<string,...>
                key = tuple(sorted(s))
                groups[key].append(s)
            # dict.values() 直接返回桶集合, list() 包成最终答案
            return list(groups.values())
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string[]} strs
     * @return {string[][]}
     */
    var groupAnagrams = function(strs) {
        // Map vs Object: Map 接受任意类型 key 且保留插入顺序; 这里 key 是 string, 用哪个都行
        // 但 Map 的 .get/.set API 更明确, 也更接近 C++ unordered_map 思维
        const groups = new Map();
        for (const s of strs) {
            // [...s] spread 字符串成字符数组, .sort() 默认按 Unicode, .join('') 拼回字符串
            // 等价 C++: string key = s; sort(key.begin(), key.end());
            const key = [...s].sort().join('');
            // Map 没有"自动建桶". 必须先取再判, 或用 ?? 兜底
            if (!groups.has(key)) groups.set(key, []);
            groups.get(key).push(s);
        }
        // Map.prototype.values() 返回 iterator, Array.from 物化成数组
        return Array.from(groups.values());
    };
    ```

## Complexity

- **Time**: `O(n · k log k)` — 排序版. 频次签名版 `O(n · k)`.
- **Space**: `O(n · k)` — 存所有字符串 + key.

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 异位词母题, 判两个
- 0438\. Find All Anagrams in a String (待补) — 滑窗 + 频次签名
- [0567. Permutation in String](../../05-two-pointers/0567-permutation-in-string/README.md) — 同款滑窗
- [0383. Ransom Note](../0383-ransom-note/README.md) — 计数数组 "subset"
- [0249. Group Shifted Strings](../0249-group-shifted-strings/README.md) — 同款"分桶 by canonical key", canonical 改为"差分签名"
- 0451\. Sort Characters By Frequency (待补) — 计数 + 按频次排序
