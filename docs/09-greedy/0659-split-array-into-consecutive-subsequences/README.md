# 0659. Split Array into Consecutive Subsequences / 分割数组为连续子序列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Hash Table · 贪心, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/split-array-into-consecutive-subsequences/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a sorted int array `nums`, return whether it can be split into one or more disjoint subsequences such that each subsequence is **consecutive** (each next element exactly +1) and has length **≥ 3**.

**中文**: 给已排序的整数数组 `nums`, 判断能否分割成若干**互不相交**的子序列, 每个子序列是**连续递增** (相邻 +1) 且**长度 ≥ 3**.

## Key Insights

1. **视角缩小: 每个 x 只有两种去处 / Localize: each x has just 2 choices**

    不要想"整个数组怎么分", 想"**拿到一个 x 它能去哪**":

    - **接龙**: 接到某个结尾是 `x-1` 的现有序列 → 序列变长.
    - **开新**: 自己起一个新序列, 同时占用 `x+1` 和 `x+2` 凑长度 3.

    > 这是整道题的钥匙. 把"全局分组" 缩小成"单点决策" 是贪心/DP 通用的思维转换.

2. **决策反推状态: count + tails 两个哈希表 / Two hashmaps come from the two decisions**

    | 决策 | 需要什么信息 | 状态 |
    |---|---|---|
    | 接龙 | 有没有结尾是 `x-1` 的序列在等? | `tails[x-1]` 计数 |
    | 开新 | `x+1` 和 `x+2` 还有剩吗? | `count[x+1]`, `count[x+2]` |

    > 哈希表不是凭空冒出来 — 是"决策需要什么信息" 反推. **先列决策, 再决定要追踪什么**.

3. **贪心策略: 接龙优先 (省资源) / Prefer extending over starting**

    `x` 既能接龙又能开新时, **选接龙**. 资源分析:

    - 接龙: 只消耗 1 个 `x`, 不动 `x+1`/`x+2`.
    - 开新: 消耗 3 个 (`x, x+1, x+2`), 后面可能因缺料而崩.

    > 一句话: **能不动用未来资源就别动**. 贪心策略来自"哪个选择更省"的资源分析.

4. **交换论证: 贪心正确性 / Why this greedy works**

    假设某步既能接龙又能开新, 选了开新. 改成接龙 — 省下的 `x+1, x+2` 留给后面, 后续要么仍能凑出原本的开新序列 (因为那些数还在), 要么更灵活 — 总之**不会变差**. 反复交换 ⇒ "接龙优先" 不劣于任何替代方案.

5. **失败边界: 既不能接龙也不能开新 → false / Bail-out condition**

    `x` 落单 (没有等 `x` 的序列, 且 `x+1` 或 `x+2` 不够) → 它进不了任何长度 ≥ 3 的连续序列 → **直接 false**.

    > 任何"判定问题" 都要主动想"什么时候失败". 不写就漏.

## 思维过程 / How to derive this

```
1. 抓题型关键词    → "分组" + "连续序列" + "长度≥3" + "判定"
        ↓
2. 视角缩小        → "单个 x 能做什么?" → 列出两个选择 (接龙 / 开新)
        ↓
3. 反推状态        → "做这两个决策需要什么信息?"
                  → 接龙需要"结尾是 x-1 的序列数量" → tails
                  → 开新需要"x+1, x+2 还剩几个"     → count
        ↓
4. 解决冲突        → "都能选时选哪个?" → 资源分析: 接龙更省 → 优先接龙
        ↓
5. 验证            → 交换论证: 换成开新不会更好
        ↓
6. 边界            → "都不能?" → false
```

> **通用框架**: 单点选择 → 状态反推 → 资源分析定贪心 → 交换论证 → 失败边界. 对任何"分配/分组/选择"问题套同一流程 (e.g. [0055 跳跃游戏](../0055-jump-game/README.md), [0056 合并区间](./../0056-merge-intervals/README.md)).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        bool isPossible(vector<int>& nums) {
            unordered_map<int, int> count;                         // count[x] = x 还剩几个
            unordered_map<int, int> tails;                         // tails[x] = 结尾为 x 的现有序列条数
            for (int x : nums) count[x]++;
            for (int x : nums) {
                if (count[x] == 0) continue;                       // 已经用掉
                if (tails[x - 1] > 0) {                            // 优先接龙
                    count[x]--;
                    tails[x - 1]--;                                // 那条序列结尾从 x-1 推进到 x
                    tails[x]++;
                } else if (count[x + 1] > 0 && count[x + 2] > 0) { // 不能接龙再开新
                    count[x]--;
                    count[x + 1]--;
                    count[x + 2]--;
                    tails[x + 2]++;                                // 新序列结尾是 x+2
                } else {
                    return false;                                  // 落单
                }
            }
            return true;
        }
    };
    ```

=== "Python"
    ```python
    from collections import Counter, defaultdict

    class Solution:
        def isPossible(self, nums: list[int]) -> bool:
            # Counter(nums) 一行算每个数的频次, 等价 C++ 的 for x: count[x]++
            count = Counter(nums)
            # defaultdict(int) 让 tails[x] 默认 0, 不用每次 .get(x, 0); C++ unordered_map[int] 自带这个语义
            tails = defaultdict(int)
            for x in nums:
                if count[x] == 0:
                    continue
                if tails[x - 1] > 0:                               # 接龙
                    count[x] -= 1
                    tails[x - 1] -= 1
                    tails[x] += 1
                elif count[x + 1] > 0 and count[x + 2] > 0:        # 开新
                    count[x]     -= 1
                    count[x + 1] -= 1
                    count[x + 2] -= 1
                    tails[x + 2] += 1
                else:
                    return False
            return True
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {boolean}
     */
    var isPossible = function(nums) {
        // Map 比 plain object 更适合 int key (不会被强转字符串), 且保留插入顺序
        const count = new Map(), tails = new Map();
        const get = (m, k) => m.get(k) || 0;                       // 缺省 0 的 helper, 替代 defaultdict
        const inc = (m, k, d) => m.set(k, get(m, k) + d);
        for (const x of nums) inc(count, x, 1);
        for (const x of nums) {
            if (get(count, x) === 0) continue;
            if (get(tails, x - 1) > 0) {                           // 接龙
                inc(count, x, -1);
                inc(tails, x - 1, -1);
                inc(tails, x, 1);
            } else if (get(count, x + 1) > 0 && get(count, x + 2) > 0) {
                inc(count, x, -1);
                inc(count, x + 1, -1);
                inc(count, x + 2, -1);
                inc(tails, x + 2, 1);
            } else {
                return false;
            }
        }
        return true;
    };
    ```

## Complexity

- **Time**: O(n) — 两遍 O(n) 扫, 哈希操作 O(1) 摊销.
- **Space**: O(n) — 两个哈希表.

## 相关题目

- [0055. Jump Game](../0055-jump-game/README.md) — 同框架: 单点决策 + 维护"目前最远"
- [0056. Merge Intervals](../0056-merge-intervals/README.md) — 同框架: 单区间决策 + 维护"上一段右端"
- [0455. Assign Cookies](../0455-assign-cookies/README.md) — 同框架: 单点配对 + 两数组贪心
- 0846\. Hand of Straights (待补) — 几乎同款"连续序列分组"
- 1296\. Divide Array in Sets of K Consecutive Numbers (待补) — 0846 重复题
- 1029\. Reordered Power of 2 (待补) — 另一类"重排判定"
- [0763. Partition Labels](../0763-partition-labels/README.md) — 同款"前置预处理 + 单点贪心" 套路
