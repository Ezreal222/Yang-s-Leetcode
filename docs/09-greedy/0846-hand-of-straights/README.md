# 0846. Hand of Straights / 一手顺子

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Hash Table, Sort · 贪心, 哈希表, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/hand-of-straights/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given int array `hand` and `groupSize`, return whether `hand` can be partitioned into groups of exactly `groupSize` cards, each group **consecutive** (e.g. `[2,3,4]`).

**中文**: 给整数数组 `hand` 和 `groupSize`, 判断能否分成若干组, 每组恰好 `groupSize` 张**连续递增**的牌.

## Key Insights

1. **核心贪心: "最小未消耗的牌一定是某组的开头" / Smallest leftover starts a group**

    任何时刻, 剩余牌里**最小的那张** `x` **不可能在某组的中间或末尾** — 因为那要求有更小的 `x-1` 参与, 但 `x` 已是最小. 所以 `x` 必须是某组的**起点**, 那组就是 `[x, x+1, ..., x+groupSize-1]`.

    决策被锁死 → 没有"选什么" 的纠结 → 直接消耗.

2. **数据结构: 有序结构 + 计数 / Ordered counter for O(log n) "next smallest"**

    需要反复拿"最小未消耗" → 用 **`std::map<int,int>`** (有序 + 计数 二合一) 或 **`sort + unordered_map`**.

    - **v1 (Yang 初版)**: `sort + unordered_map`. 顺序扫排序后的数组, 跳过已消耗的, 每次消耗 1 组. O(n log n + n·groupSize).
    - **v2 (推荐)**: `map<int,int>` 自动按 key 排序. 对每个还有剩的最小 key, **一次批量消耗 `cnt` 组** ( `cnt = count[card]` 张同点必须各自开一组). 代码更紧.

3. **v2 的关键: 一次性消耗"以 card 为起点的所有组" / Batch-consume all groups starting at card**

    `need = count[card]` — 因为 card 是当前最小未消耗, 这 `need` 张**必须都是组的起点**. 那就同时减 `need` 张连续: `count[card..card+groupSize-1] -= need`.

    一次批量 vs 一次一组, 复杂度从 O(n·groupSize) 降到 O((n/groupSize)·groupSize) = O(n) (摊销).

4. **早返: `hand.size() % groupSize != 0` / Quick fail**

    总牌数不能整除 groupSize → 直接 false, 省后续操作.

5. **跟 [0659 分割连续子序列](../0659-split-array-into-consecutive-subsequences/README.md) 的对比 / vs 0659**

    | 题 | 段长 | 决策灵活度 | 数据结构 |
    |---|---|---|---|
    | **0846 (本题)** | **固定** `groupSize` | 决策被锁死 (最小必为起点) | 有序 map |
    | 0659 | **≥ 3** (可变) | 两种选择 (接龙 / 开新) + 贪心选 | count + tails |

    > 一句话: **段长固定 → 决策刚性, 一个哈希表搞定**; **段长可变 → 决策有选择, 需要 tails 追踪进行中的序列**.

6. **1296 是孪生重复题 / 1296 is a duplicate**

    1296\. Divide Array in Sets of K Consecutive Numbers (待补) 跟本题完全等价 — 同代码一份过两题.

## Solution

=== "C++"
    === "v2 推荐: 有序 map + 批量消耗"
        ```cpp
        class Solution {
        public:
            bool isNStraightHand(vector<int>& hand, int groupSize) {
                if (hand.size() % groupSize != 0) return false;    // 早返
                map<int, int> count;                               // map 自动按 key 升序
                for (int card : hand) count[card]++;
                for (auto& [card, cnt] : count) {
                    if (cnt > 0) {
                        int need = cnt;                            // 以 card 为起点要凑 need 个组
                        for (int i = card; i < card + groupSize; i++) {
                            if (count[i] < need) return false;     // 这张牌不够 need 张 → 凑不齐
                            count[i] -= need;
                        }
                    }
                }
                return true;
            }
        };
        ```

    === "v1 (sort + 逐组消耗)"
        ```cpp
        // 思路对, 但逐组消耗代码长且 O(n·groupSize)
        class Solution {
        public:
            bool isNStraightHand(vector<int>& hand, int groupSize) {
                unordered_map<int, int> count;
                sort(hand.begin(), hand.end());                    // 没有有序结构, 靠 sort 拿"最小"
                for (int x : hand) count[x]++;
                for (int x : hand) {
                    if (count[x] == 0) continue;
                    for (int i = 1; i < groupSize; i++) {
                        if (count[x + i]-- == 0) return false;     // 后置 --: 先判 0 再减
                    }
                    count[x]--;
                }
                return true;
            }
        };
        ```

=== "Python"
    ```python
    from collections import Counter

    class Solution:
        def isNStraightHand(self, hand: list[int], groupSize: int) -> bool:
            if len(hand) % groupSize:
                return False
            # Counter 是 dict 子类, sorted(count) 给升序 key 列表 — 等价 C++ 的 map
            # 没有内建有序 dict, 但本题"按升序遍历 key" 用 sorted 即可
            count = Counter(hand)
            for card in sorted(count):
                need = count[card]
                if need > 0:
                    for i in range(card, card + groupSize):
                        if count[i] < need:
                            return False
                        count[i] -= need
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} hand
     * @param {number} groupSize
     * @return {boolean}
     */
    var isNStraightHand = function(hand, groupSize) {
        if (hand.length % groupSize !== 0) return false;
        // Map 保留插入顺序; 想要按 key 升序就先拿 keys, sort 一下
        const count = new Map();
        for (const card of hand) count.set(card, (count.get(card) || 0) + 1);
        // [...count.keys()].sort((a,b) => a-b): 数字升序; 别忘 compareFn (默认字典序错)
        const keys = [...count.keys()].sort((a, b) => a - b);
        for (const card of keys) {
            const need = count.get(card);
            if (need > 0) {
                for (let i = card; i < card + groupSize; i++) {
                    if ((count.get(i) || 0) < need) return false;
                    count.set(i, count.get(i) - need);
                }
            }
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n log n) — 排序 / map 插入主导. 批量消耗内层均摊 O(n).
- **Space**: O(n) — count / map.

## 相关题目

- [0659. Split Array into Consecutive Subsequences](../0659-split-array-into-consecutive-subsequences/README.md) — 段长可变版, 需要 tails 追踪
- 1296\. Divide Array in Sets of K Consecutive Numbers (待补) — 本题完全重复, 同代码
- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同框架"最小未处理元素决策被锁死"
- [0763. Partition Labels](../0763-partition-labels/README.md) — 同款"按最小起点贪心切段"
- 1953\. Maximum Number of Weeks for Which You Can Work (待补) — 同款"频次驱动" 贪心
