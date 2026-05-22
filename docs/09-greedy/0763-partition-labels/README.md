# 0763. Partition Labels / 划分字母区间

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Hash Table, Two Pointers, String · 贪心, 哈希表, 双指针, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/partition-labels/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Partition string `s` into as many parts as possible so each letter appears in **at most one** part. Return the list of part lengths.

**中文**: 把字符串 `s` 划分成尽可能多的片段, 使每个字母**只出现在一个**片段里. 返回每段长度.

## Key Insights

1. **预处理: 记每个字母最后出现的位置 / Precompute last-seen index per letter**

    一次扫记下 `lastPos[c] = `字母 c 最后出现的 index. 这是后面贪心切刀的依据 — **任何一段必须包含段内每个字母的"最后一次出现"**, 否则后面还会出现该字母, 段就不闭合.

    用 `vector<int>(26, 0)` (小写字母固定 26 个), 一次 O(n) 扫完.

2. **贪心扫描: 维护当前段的"最远必须延伸到" 边界 / Greedy: extend the running right edge**

    扫第二遍, 维护 `end = max(end, lastPos[s[i]])` — 当前段必须延伸到的最右位置. 当 `i == end`, 说明**段内所有字母的最后一次出现都已经被走过** → 切一刀, 长度 = `end - start + 1`, 然后 `start = i + 1` 开新段.

    > 一句话: **段内每个字母的"最后位置" 的最大值** = 段的右边界. 走到这就闭合.

3. **跟 [0452](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) / [0435](../0435-non-overlapping-intervals/README.md) 是同一族 / Same "running right edge" family**

    都是"区间贪心 + 维护当前组右端" 模板. 区别只在右端怎么来:

    | 题 | 右端来自 | 切分条件 |
    |---|---|---|
    | **0763 (本题)** | `max(lastPos[char])` | `i == end` |
    | 0452 气球 | 排序后的 `points[0][1]` | `next.left > end` |
    | 0435 区间 | 排序后的 `intervals[0][1]` | `next.left < end` (重叠) |
    | 0056 合并 (待补) | 排序后的 `intervals[0][1]` | `next.left > end` |

    本题不需要 sort, 因为"字母只有 26 个", 用 lastPos 数组直接拿到右边界.

4. **为什么贪心一定最优 / Why greedy works**

    每一段都"必须" 至少这么长 (要包含段首字母的最后出现). 同时段一旦能闭合就**立刻闭合** — 不闭合就只能更长, 段数更少. 所以"能切就切" = 最多段数. 局部最优 ⇒ 全局最优.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> partitionLabels(string s) {
            vector<int> lastPos(26, 0);
            for (int i = 0; i < (int)s.size(); i++) {
                lastPos[s[i] - 'a'] = i;                           // 覆盖式, 最后一次写入 = 最后出现的 index
            }
            vector<int> res;
            int start = 0, end = 0;
            for (int i = 0; i < (int)s.size(); i++) {
                end = max(end, lastPos[s[i] - 'a']);               // 把当前段右端往后推
                if (end == i) {                                    // 走到了右端 → 段闭合
                    res.push_back(end - start + 1);
                    start = i + 1;
                }
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def partitionLabels(self, s: str) -> list[int]:
            # 字典推导一次扫: 同 key 重复写入, 最后留下的就是最后出现的 index
            # 等价 C++ 的 vector<int>(26) + 覆盖式赋值, 但用 dict 不受字符集限制
            last_pos = {c: i for i, c in enumerate(s)}
            res = []
            start = end = 0
            for i, c in enumerate(s):
                end = max(end, last_pos[c])
                if i == end:
                    res.append(end - start + 1)
                    start = i + 1
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} s
     * @return {number[]}
     */
    var partitionLabels = function(s) {
        // Map 比 plain object 更适合"任意字符做 key" 的场景, 也保留插入顺序
        // 这里用 plain object 也行, 因为 key 都是单字符, 不会有冲突
        const lastPos = {};
        for (let i = 0; i < s.length; i++) {
            lastPos[s[i]] = i;                                     // 覆盖式: 最后写入 = 最后出现位置
        }
        const res = [];
        let start = 0, end = 0;
        for (let i = 0; i < s.length; i++) {
            end = Math.max(end, lastPos[s[i]]);
            if (i === end) {
                res.push(end - start + 1);
                start = i + 1;
            }
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 两遍线性扫.
- **Space**: O(1) 额外 (lastPos 固定 26 / 字符集大小有限).

## 相关题目

- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 同款"维护当前组右端" 贪心 (sort 后)
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 同款"右端 + 切分" 区间贪心
- 0056\. Merge Intervals (待补) — 区间合并版, 按左端 sort + 维护右端
- 0758\. Bold Words in String (待补) — 同款"合并被覆盖的区间"
- 1024\. Video Stitching (待补) — 区间覆盖 + 贪心
