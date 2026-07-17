# 0151. Reverse Words in a String / 反转字符串中的单词

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Two Pointers, In-place, String, Reverse · 双指针, 原地, 字符串, 反转
    - **Link**: [LeetCode](https://leetcode.com/problems/reverse-words-in-a-string/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Reverse word order, trim/collapse spaces** → **v1 (O(1) extra)**: three passes: fast/slow pack + trim → reverse whole → reverse each word. **v2 (clean)**: `stringstream >> word` collect + iterate backwards to build result.
>
> **中文**: **反转单词顺序 + 处理多余空格** → **v1 (原地)**: 三遍扫: 快慢打包去空格 → 整串反转 → 每词再反转. **v2 (清爽)**: stringstream 读词 + 反向拼接.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给字符串 `s`. 反转其中的**单词顺序**. 要求:

- 处理**多余空格** (前后 / 中间连续).
- 输出**单词间只有 1 个空格**, 无前后空格.

**中文**: 反转单词顺序, 清理多余空格.

## Key Insights

1. **🔑 灵魂 trick: 整反 + 每词再反 = 单词顺序颠倒 / Whole-reverse + word-reverse = word order flipped**

    数学观察: 字符串 = `a + b + c` (三个词). 整体反转 → `c' + b' + a'` (每词字符也反了). 再**每个词内反转** → `c + b + a`.

    ```
    "the sky is blue"        原
    "eulb si yks eht"        整反后
    "blue is sky the"        每词再反
    ```

    → **两次反转**, 单词顺序颠倒, 每词内容不变. **无需额外存储**.

    > **"反转 + 再反转 = 位置颠倒"** 是**数组/字符串**里的通用技巧, 也用在**旋转数组**  (0189) 和**Reverse Only Letters** (0917).

2. **🔑 v1 Pass 1: 同向快慢双指针 pack 空格 / Fast-slow to trim spaces**

    Yang 的巧写:

    ```cpp
    while (fast < n) {
        while (fast < n && s[fast] == ' ') fast++;       // 跳到下一个词的开头
        if (fast == n) break;                             // 尾部只有空格 → 结束
        if (slow != 0) s[slow++] = ' ';                   // 不是第一个词 → 前面加分隔空格
        while (fast < n && s[fast] != ' ') s[slow++] = s[fast++];  // 拷贝整个词
    }
    s.resize(slow);                                       // 截断
    ```

    - **`slow` 是"写位置", `fast` 是"读位置"** — 快慢**同向**双指针的标准模板 (跟 [0027 Remove Element](../0027-remove-element/README.md) 同款).
    - **`if (slow != 0) s[slow++] = ' ';`** 是漂亮设计: **除了第一个词, 每个词前面加分隔空格**. 保证末尾无 trailing space.

    > **"前置分隔符" 而非"后置"** 是 join 的通用模板 — 少一次尾部特判. 之前 [0648 Replace Words](../../03-hash-table/0648-replace-words/README.md) 里也用过.

3. **🔑 v1 Pass 2: 整串反转 (STL reverse) / Reverse the whole string**

    ```cpp
    reverse(s.begin(), s.end());
    ```

    整体反转. **词内字符**也倒了 — 靠 Pass 3 修回来.

4. **🔑 v1 Pass 3: 分词, 每词反转 / Segment + reverse each word**

    ```cpp
    int start = 0;
    for (int i = 0; i <= s.size(); i++) {
        if (i == s.size() || s[i] == ' ') {
            reverse(s.begin() + start, s.begin() + i);
            start = i + 1;
        }
    }
    ```

    - **`i <= s.size()` 而不是 `<`**: 因为最后一个词后面**没有空格**, 靠 `i == size` 触发最后一次 reverse.
    - `reverse(begin + start, begin + i)` — **半开区间 `[start, i)`** — STL reverse 的语义, 跟 [0025 Reverse Nodes in k-Group](../../02-linked-list/0025-reverse-nodes-in-k-group/README.md) 的半开约定同源.

5. **🔑 v2: stringstream 分词 + 反向拼 / stringstream + reverse-collect**

    ```cpp
    stringstream ss(s);
    string w;
    vector<string> words;
    while (ss >> w) words.push_back(w);         // 自动跳空格
    string res;
    for (int i = words.size() - 1; i >= 0; i--) {
        if (!res.empty()) res += " ";
        res += words[i];
    }
    ```

    - **`ss >> word` 自动按空白切**, 顺便**过滤多余空格 + 前后空格** — 一步到位.
    - **反向 for + 前置空格 join** — 又是"非空才加分隔符" 模板.

    > v2 优雅 & 短一半, 但 **O(n) 额外空间** (存 words 数组). v1 是 **O(1) 额外**.

6. **🔑 v1 vs v2 对比 / v1 vs v2**

    | | **v1 三遍扫** | **v2 stringstream** |
    |---|---|---|
    | Time | O(n) | O(n) |
    | **Extra Space** | **O(1)** | O(n) (words 数组) |
    | 代码量 | 长 (三 pass 逻辑) | **短一半** |
    | 面试推荐 | **Follow-up "O(1) 空间"** | 首选 |
    | 展示技术 | 快慢 + 反转 trick | 分词 API |

    > **面试思路**: 先写 v2 展示能读题, 再问 "能不能原地 O(1)?" → v1.

7. **🔑 快慢同向双指针 = §05 的第三大流派 / Fast-slow: third TP family**

    回顾双指针三大模式 (跟 [0209 Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) 里的对比表呼应):

    | 模式 | 代表 | 特点 |
    |---|---|---|
    | 同向快慢 | [0027](../0027-remove-element/README.md) / **本题 v1 Pass 1** | slow 写, fast 读 |
    | 对撞合拢 | [0344](../0344-reverse-string/README.md) / [0977](../0977-squares-of-a-sorted-array/README.md) | 两端往中间 |
    | 不定长滑窗 | [0209](../0209-minimum-size-subarray-sum/README.md) | 右扩左缩 |

    > 本题 v1 里**pack 空格用 "同向", reverse 用 "对撞" 的思想**. 一题两模式.

8. **🔑 复杂度 / Complexity**

    - **v1**: Time O(n), Space **O(1)** extra (原地).
    - **v2**: Time O(n), Space O(n) (words 数组).

## Solution

=== "C++"

    **v1: 三遍扫 (O(1) 额外空间)**

    ```cpp
    class Solution {
    public:
        string reverseWords(string s) {
            int slow = 0, fast = 0, n = s.size();
            // Pass 1: 快慢 pack, 去多余空格, 词间加单空格
            while (fast < n) {
                while (fast < n && s[fast] == ' ') fast++;
                if (fast == n) break;
                if (slow != 0) s[slow++] = ' ';                      // 非首词, 前置空格
                while (fast < n && s[fast] != ' ') s[slow++] = s[fast++];
            }
            s.resize(slow);

            // Pass 2: 整串反转
            reverse(s.begin(), s.end());

            // Pass 3: 每词反转
            int start = 0;
            for (int i = 0; i <= (int)s.size(); i++) {
                if (i == (int)s.size() || s[i] == ' ') {
                    reverse(s.begin() + start, s.begin() + i);       // 半开 [start, i)
                    start = i + 1;
                }
            }
            return s;
        }
    };
    ```

    **v2: stringstream + 反向拼 (清爽, O(n) 额外)**

    ```cpp
    class Solution {
    public:
        string reverseWords(string s) {
            stringstream ss(s);
            vector<string> words;
            string w;
            while (ss >> w) words.push_back(w);                       // 自动跳空格
            string res;
            for (int i = words.size() - 1; i >= 0; i--) {
                if (!res.empty()) res += " ";                          // 前置空格
                res += words[i];
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v2 — Python 最短
        def reverseWords(self, s: str) -> str:
            # str.split() 无参数默认按连续空白切 + 自动过滤空串 (包括前后空格)
            # [::-1] 切片反转. ' '.join(...) 用单空格拼接
            # 三个 idiom 合起来一行
            return ' '.join(s.split()[::-1])

        # v1 手动版 (对齐 C++ 教学)
        def reverseWords_manual(self, s: str) -> str:
            # Python str 不可变, "原地" 严格意义上无法做. 但用 list 模拟
            chars = list(s)
            n = len(chars)
            slow, fast = 0, 0
            while fast < n:
                while fast < n and chars[fast] == ' ': fast += 1
                if fast == n: break
                if slow != 0:
                    chars[slow] = ' '; slow += 1
                while fast < n and chars[fast] != ' ':
                    chars[slow] = chars[fast]; slow += 1; fast += 1
            chars = chars[:slow]
            chars.reverse()
            # 分词反转
            start = 0
            for i in range(len(chars) + 1):
                if i == len(chars) or chars[i] == ' ':
                    chars[start:i] = chars[start:i][::-1]
                    start = i + 1
            return ''.join(chars)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {string}
     */
    var reverseWords = function(s) {
        // v2: JS 也有一行版
        // s.trim() 去前后空格; .split(/\s+/) 按连续空白切成词数组 (正则 \s+); .reverse().join(' ')
        // 三步链式, 干净. O(n) 空间跟 C++ v2 一致
        return s.trim().split(/\s+/).reverse().join(' ');
    };
    ```

## Complexity

| Version | Time | Extra Space |
|---|---|---|
| v1 (三遍原地) | O(n) | **O(1)** |
| v2 (stringstream) | O(n) | O(n) |

## 相关题目

- [0344. Reverse String](../0344-reverse-string/README.md) — 对撞双指针最简
- [0027. Remove Element](../0027-remove-element/README.md) — 同向快慢母题
- [0977. Squares of a Sorted Array](../0977-squares-of-a-sorted-array/README.md) — 对撞双指针
- [0209. Minimum Size Subarray Sum](../0209-minimum-size-subarray-sum/README.md) — 滑窗
- [0648. Replace Words](../../03-hash-table/0648-replace-words/README.md) — stringstream 分词
- [0271. Encode and Decode Strings](../../04-string/0271-encode-and-decode-strings/README.md) — 字符串编码
- 0189\. Rotate Array (待补) — 数组"三次反转" 旋转
- 0917\. Reverse Only Letters (待补) — 对撞 + 只交换字母
- 0186\. Reverse Words in a String II (待补) — 原地版, 保证输入格式
- 0557\. Reverse Words in a String III (待补) — 保持单词顺序反转每词
