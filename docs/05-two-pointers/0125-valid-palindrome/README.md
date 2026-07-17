# 0125. Valid Palindrome / 验证回文串

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Two Pointers, String, Palindrome · 双指针, 字符串, 回文
    - **Link**: [LeetCode](https://leetcode.com/problems/valid-palindrome/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Palindrome check ignoring non-alnum and case** → **opposite two pointers**: inner `while` skip non-alnum on both ends; compare `tolower(s[l])` vs `tolower(s[r])`; mismatch → false.
>
> **中文**: **判回文, 只看字母数字, 忽略大小写** → **对撞双指针**: 内层 while 跳非字母数字; `tolower` 后比较; 不等即 false.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断 `s` 是否是回文 — **只考虑字母数字**, 忽略**大小写** 和其他字符.

例: `"A man, a plan, a canal: Panama"` → true (`"amanaplanacanalpanama"`).

**中文**: 判回文, 过滤非字母数字, 忽略大小写.

## Key Insights

1. **🔑 对撞双指针 + 双内层 while 跳过 / Opposite two pointers + inner-skip pattern**

    最直接的思路: 先 `strip` 出所有 alnum, 全转小写, 再对撞判. 但**需要 O(n) 额外空间**.

    **对撞双指针 in-place**: 两端各自**边比较边跳非法字符**, O(1) 空间.

    ```cpp
    while (left < right) {
        while (left < right && !isalnum(s[left])) left++;      // 跳到左端下一个合法
        while (left < right && !isalnum(s[right])) right--;    // 跳到右端下一个合法
        if (tolower(s[left]) != tolower(s[right])) return false;
        left++; right--;
    }
    ```

    > **"过滤 + 比较" 融进对撞**: 少一次预处理. **面试首选**.

2. **🔑 `left < right` 守卫在**内层**也要写 / Guard `left < right` inside both while too**

    Yang 的两个内层 while 都写了 `left < right &&` — **必须**. 否则字符串全是非法字符时, `left` 会**冲过 `right`**, 后续 `s[left]` 越界或逻辑错.

    ```
    s = ".,!"           全非法
    外层 left=0, right=2
    内层跳: left → 1 → 2 → 3 (越界!) 若无 left < right 守卫
    ```

    > **"内层循环推进指针" 时**, **每个内层守卫都要检查跟外层同款的边界**. 少写就爆.

3. **🔑 `isalnum` / `tolower` 出自 `<cctype>` / C-style char classifiers**

    - **`isalnum(c)`**: 是不是字母或数字 (0-9 A-Z a-z). 返 int (非 0 = true).
    - **`tolower(c)`**: 转小写, 非字母原样返.
    - **返 int 而非 char**: C 遗产, 因 EOF 是 -1 需要覆盖. 用作 bool / char 比较**无害**.

    > **`<cctype>` 里的 `isalpha` / `isdigit` / `isspace` / `toupper`** 是一族. 记住 char 判定不用手写 `>= 'a'` 那套.

4. **🔑 空串 / 单字符自动 true / Empty & single-char auto true**

    - 空串: `left = 0, right = -1`. 外层 `left < right` 假 → 直接返 true. ✅
    - 单字符: `left = 0, right = 0`. 同上. ✅
    - 全非法字符: 内层跳完 `left == right`, 外层退. 返 true. ✅ ("回文 = 空是回文")

    > **边界自然处理**是好设计的体现. 不需要特判.

5. **🔑 对比"预处理版" / vs preprocess version**

    ```cpp
    string t;
    for (char c : s) if (isalnum(c)) t += tolower(c);
    // 判 t 是不是回文 (对撞或 t == reverse(t))
    ```

    - **O(n) 额外空间** 建 t.
    - **代码更短**, 教学友好.
    - Yang 的**原地版** O(1) 空间, 更精.

    > **面试 followup "能不能 O(1) 空间?"** → 上原地对撞.

6. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    - Time: 每字符被访问最多两次 (一次内层跳, 一次比较).
    - Space: 两个指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isPalindrome(string s) {
            int left = 0, right = (int)s.size() - 1;
            while (left < right) {
                while (left < right && !isalnum(s[left]))  left++;      // 跳非法
                while (left < right && !isalnum(s[right])) right--;
                if (tolower(s[left]) != tolower(s[right])) return false;
                left++; right--;
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def isPalindrome(self, s: str) -> bool:
            # Pythonic 一行版: filter alnum + lower + 切片比较. O(n) 额外空间
            # str.isalnum() 是 str 的方法; c.lower() 转小写
            # 生成器 → ''.join → 判自反
            # t = ''.join(c.lower() for c in s if c.isalnum())
            # return t == t[::-1]

            # 但**面试更愿要 O(1) 空间的对撞版** — 对齐 C++
            left, right = 0, len(s) - 1
            while left < right:
                while left < right and not s[left].isalnum():
                    left += 1
                while left < right and not s[right].isalnum():
                    right -= 1
                # str.lower() 处理单字符也 OK — 一步比较
                if s[left].lower() != s[right].lower():
                    return False
                left += 1
                right -= 1
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {boolean}
     */
    var isPalindrome = function(s) {
        // JS 无原生 isalnum, 用正则 /[a-z0-9]/i 或手判 char code
        // /^[a-z0-9]$/i 单字符检测 (i = 忽略大小写)
        const isAlnum = c => /[a-z0-9]/i.test(c);
        let left = 0, right = s.length - 1;
        while (left < right) {
            while (left < right && !isAlnum(s[left]))  left++;
            while (left < right && !isAlnum(s[right])) right--;
            // .toLowerCase() 对单字符也 OK
            if (s[left].toLowerCase() !== s[right].toLowerCase()) return false;
            left++;
            right--;
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n) — 每字符最多访问 2 次.
- **Space**: O(1) — 两指针.

## 相关题目

- [0344. Reverse String](../0344-reverse-string/README.md) — 对撞双指针最简
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针
- [0246. Strobogrammatic Number](../0246-strobogrammatic-number/README.md) — 对撞 + 旋转映射
- [0541. Reverse String II](../0541-reverse-string-ii/README.md) — 每 2k 反转
- [0266. Palindrome Permutation](../../03-hash-table/0266-palindrome-permutation/README.md) — 频次判可回文
- 0680\. Valid Palindrome II (待补) — 允许**删一字符**
- 1216\. Valid Palindrome III (待补) — 允许**删 ≤ k 字符**, DP
- 0009\. Palindrome Number (待补) — 数字回文
- 0234\. Palindrome Linked List (待补) — 链表回文 (快慢 + 后半反转)
- 0409\. Longest Palindrome (待补) — 频次构造最长回文
