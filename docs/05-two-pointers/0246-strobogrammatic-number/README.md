# 0246. Strobogrammatic Number / 中心对称数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, Hash Map, String · 双指针, 哈希表, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/strobogrammatic-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Number reads same after 180° rotation?** → **rotation map** `{0:0, 1:1, 6:9, 8:8, 9:6}`; **opposite two pointers**: at each step check `mp[num[i]] == num[j]`; any digit outside the map → false.
>
> **中文**: **数字 180° 旋转后是否相同?** → **旋转映射** `{0:0, 1:1, 6:9, 8:8, 9:6}`; **对撞双指针**每步查 `mp[num[i]] == num[j]`; 出现映射外字符即 false.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断数字字符串 `num` 是否是**中心对称数** — 把它**旋转 180°** 后**看起来一样**.

例: "69" → "69" (对称 ✅), "962" → "296" 而非 "962" (❌).

**中文**: 判"180° 旋转后跟原来一样" 的数字.

## Key Insights

1. **🔑 灵魂事实: 只有 5 个数字 rotate 合法 / Only 5 digits are rotation-safe**

    数字 180° 旋转:

    | digit | rotated |
    |---|---|
    | **0** | 0 ✅ 自映射 |
    | **1** | 1 ✅ 自映射 |
    | 2, 3, 4, 5, 7 | 无对应数字 ❌ 直接 false |
    | **6** | 9 ✅ 互映射 |
    | **8** | 8 ✅ 自映射 |
    | **9** | 6 ✅ 互映射 |

    → **hash map: `{0:0, 1:1, 6:9, 8:8, 9:6}`**. 5 项常量.

    > **"字符集小 + 固定映射" 的题**都可以用 hash map 硬编码. 记住这 5 项 (0/1/8 自映射, 6↔9 互映射).

2. **🔑 结构观察: **对撞双指针** 天然匹配"两端对称" / Opposite two pointers naturally handle end-swap**

    旋转 180° 后, **原第 i 位 → 新第 (n-1-i) 位**. 判"跟原一样" =
    `rotated[i] == num[i]` 对所有 i.

    等价: `rotate(num[n-1-i]) == num[i]` — **两端字符, rotate 一个, 跟另一个比**. **对撞双指针**每步比一对.

    > **看到"回文 / 对称 / 两端配对"** → 反应对撞双指针. 跟 [0977 Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) / [0125 Valid Palindrome](../0125-valid-palindrome/README.md) 一族.

3. **🔑 `i <= j` 而不是 `i < j` — 处理中位 / `<=` for odd-length center**

    奇数长度时, **中位** 位置 `i == j`, 单字符**也要**判是不是自映射 (0/1/8).

    - "8" ✅ 单字符自映射.
    - "5" ❌ 5 不在 map.
    - "6" ❌ 6 → 9 ≠ 6, 单字符 rotate 变别的.

    `i <= j` 让 `i == j` 时进循环判一次. 若写 `i < j`, "6" 这种会漏判 → 返 true (错).

4. **🔑 两个 check 都必要 / Both checks are needed**

    Yang 的核心两行:

    ```cpp
    if (!mp.count(num[i])) return false;      // (1) 字符必须在 map 里
    if (mp[num[i]] != num[j]) return false;   // (2) rotation 跟对面相等
    ```

    - **(1) 不能省**: 若 num[i] 是 '2', `mp[num[i]]` 会**自动插入 key** ('2' → '\0'), 副作用 + 返 '\0'. 比较结果不定, 逻辑错.
    - **(2) 是主判定**: 两端 rotate 后必须相等.

    > **C++ `unordered_map::operator[]` 缺 key 会插入默认值** — 想只读用 `.count()` / `.find()` / `.at()`. 老手必踩过这船 (在 [0454 4Sum II](../../03-hash-table/0454-4sum-ii/README.md) 里我们**利用**了这特性; 这里要**避开**).

5. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear time, constant space**

    - Time: 一遍对撞扫, ≤ n/2 步.
    - Space: 5-item map 常量.

6. **🔑 备选写法: 只用数组不用 map / Array instead of map**

    ```cpp
    int mp[10] = {0, 1, -1, -1, -1, -1, 9, -1, 8, 6};
    // 用 mp[num[i]-'0'] == num[j]-'0', mp[...] == -1 → false
    ```

    更快 (数组随机访问 O(1)), 更省 (10 int vs unordered_map). 但**可读性略差**. Yang 用 map 更教学.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isStrobogrammatic(string num) {
            unordered_map<char, char> mp = {
                {'0', '0'}, {'1', '1'}, {'6', '9'}, {'8', '8'}, {'9', '6'}
            };
            int i = 0, j = (int)num.size() - 1;
            while (i <= j) {                                 // <= 处理中位
                if (!mp.count(num[i])) return false;         // 字符必须合法
                if (mp[num[i]] != num[j]) return false;      // rotation 跟对面相等
                i++; j--;
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isStrobogrammatic(self, num: str) -> bool:
            # 字典字面量声明 rotation map. 5 项常量, 一步搞定
            mp = {'0': '0', '1': '1', '6': '9', '8': '8', '9': '6'}
            i, j = 0, len(num) - 1
            while i <= j:
                # dict.get(k) 缺 key 返 None, 不会像 mp[k] 一样抛 KeyError — 更安全
                # None != num[j] 恒成立 → 一步覆盖两个 check
                if mp.get(num[i]) != num[j]:
                    return False
                i += 1; j -= 1
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} num
     * @return {boolean}
     */
    var isStrobogrammatic = function(num) {
        // Object 字面量当 map. JS 对 char 索引也 OK ('6' 是 string key)
        const mp = {'0': '0', '1': '1', '6': '9', '8': '8', '9': '6'};
        let i = 0, j = num.length - 1;
        while (i <= j) {
            // mp[num[i]] 缺 key 返 undefined, undefined !== 任何真值 → 一步覆盖两个 check
            // (跟 Python .get() 同款技巧)
            if (mp[num[i]] !== num[j]) return false;
            i++; j--;
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n) — 一遍对撞.
- **Space**: O(1) — 常量 5-项 map.

## 相关题目

- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针母题
- [0015. 3Sum](../0015-3sum/README.md) — 对撞双指针 + 3 层去重
- [0018. 4Sum](../0018-4sum/README.md) — 对撞双指针 + 4 层去重
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 不定长滑窗
- [0125. Valid Palindrome](../0125-valid-palindrome/README.md) — 对撞双指针判回文
- 0009\. Palindrome Number (待补) — 数字回文
- 0247\. Strobogrammatic Number II (待补) — 生成所有长度 n 的中心对称数
- 0248\. Strobogrammatic Number III (待补) — 范围内计数
- 0242\. Valid Anagram — 字符统计
