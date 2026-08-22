# 0287. Find the Duplicate Number / 寻找重复数

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Two Pointers, Floyd's Cycle Detection, Abstraction · 数组, 双指针, Floyd 判环, 抽象
    - **Link**: [LeetCode](https://leetcode.com/problems/find-the-duplicate-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Find the duplicate in `nums[1..n]` of length `n+1`, O(1) space, don't modify** → **treat `nums` as an implicit linked list** where `i → nums[i]` (values are 'next pointers'). Duplicate = node with **two incoming edges** = **cycle entry**. Solve with **Floyd's tortoise & hare** exactly like [0142](../0142-linked-list-cycle-ii/README.md).
>
> **中文**: **长 n+1 数组含 [1,n] 值, 找唯一重复; O(1) 空间不改数组** → **把数组当隐式链表** (值作 next 指针). 重复数 = 多入度节点 = **环入口**. **Floyd 龟兔** 直接搬 [0142](../0142-linked-list-cycle-ii/README.md).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 数组 `nums` 长 `n+1`, 每个元素 ∈ `[1, n]`. **有且仅有 1 个数字重复** (可能多次). 找出该重复数.

**约束**: **不能改数组**, **O(1) 额外空间**, **O(n) 时间**.

**中文**: n+1 长数组值 ∈ [1,n], 唯一重复找出. 空间 O(1), 不改.

## Key Insights

1. **🔑 灵魂抽象: 数组 = 隐式链表 / Array as implicit linked list**

    把 nums 看成链表:

    - **节点 = 下标** `0, 1, 2, ..., n`.
    - **`i` 的 next = `nums[i]`** (因为值 ∈ [1, n], 也是合法下标).
    - **起点 = 下标 0** (`nums[0]` ∈ [1, n], 一直能走).

    → **值 = 下一个节点的下标**. 数组"活" 成了链表.

    ```
    nums = [1, 3, 4, 2, 2]
    从 0 走: 0 → nums[0]=1 → nums[1]=3 → nums[3]=2 → nums[2]=4 → nums[4]=2 → ...
    路径:     0 → 1 → 3 → 2 → 4 → 2 → 4 → 2 → ... (环: 2 → 4 → 2)
    ```

    > **"看不见的链表"** — 数据结构的抽象能力. 一旦看出这层结构, [0142](../0142-linked-list-cycle-ii/README.md) 的所有工具**原样搬**.

2. **🔑 重复数 = 有环, 且它是环入口 / Duplicate = cycle entry**

    **为啥有环?**

    - n+1 个下标, 每个 next 是 [1, n] 中的值 (共 n 个可能).
    - **鸽巢原理**: n+1 → n 的映射必有两个 index 映到同一值 → 同一 next → **多入度** → **环**.

    **为啥重复数是环入口?**

    - 重复的**值** `d` 被多个下标指向 (例上, d=2 被 index 3 和 index 4 指) → 有**多条路径汇到 d**.
    - **环入口** 的定义就是"进入循环的第一个节点" = 被多路径进入的地方.
    - → **重复数 = 环入口**.

    → 问题变成 [0142 Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) **找环入口**.

3. **🔑 完全套 [0142](../0142-linked-list-cycle-ii/README.md) 的 Floyd 龟兔 / Full Floyd**

    **Phase 1 — 找相遇点**:

    ```cpp
    int slow = nums[0], fast = nums[nums[0]];    // 一步和两步
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[nums[fast]];
    }
    ```

    Yang 的写法**跳过循环变量初始化的边界** — 先手动走一步 (slow) 两步 (fast), 然后用 `while (slow != fast)` (do-while 语义).

    **Phase 2 — 找环入口**:

    ```cpp
    slow = 0;                                     // 从起点重新出发
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];                        // 现在也 1 步
    }
    return slow;                                  // 相遇即入口 = 重复数
    ```

    数学论证跟 [0142](../0142-linked-list-cycle-ii/README.md) 一字不差: **`a = c + (n-1) × 环长`** → 从起点和相遇点同时每步 1, 在入口相遇.

    > **算法通用性的美感** — Floyd 不管数据类型, 只要有"next 函数" 就能用. **数组 (本题) / 链表 (0142) / 数字迭代 ([0202 Happy Number](../../03-hash-table/0202-happy-number/README.md))** 三种载体一个 Floyd.

4. **🔑 为啥其他方法都不完美 / Why not other methods**

    | 方法 | Time | Space | 改数组? | 备注 |
    |---|---|---|---|---|
    | 排序 + 相邻查重 | O(n log n) | O(1) | **改!** ❌ | 违反约束 |
    | Hash set | O(n) | **O(n)** ❌ | 不改 | 违反空间约束 |
    | 二分答案 | O(n log n) | O(1) | 不改 | 时间不最优 |
    | 就地标记 (nums[|nums[i]|] = -1) | O(n) | O(1) | **改!** ❌ | 违反约束 |
    | **Floyd** (Yang) | **O(n)** | **O(1)** | **不改** ✅ | **唯一完美** |

    → **Floyd 是本题唯一满足所有约束**的解. 也是题目"要求 O(n) + O(1) + 不改" 的**暗示**.

    > **"约束条件 (O(n)/O(1)/不改) 组合起来 = 强烈暗示某个特定算法"** — 面试识别这个暗号很关键.

5. **🔑 Yang 的初始化技巧: 手动走 1/2 步替代 do-while / Init trick**

    ```cpp
    int slow = nums[0], fast = nums[nums[0]];    // 先各走 1 / 2 步
    while (slow != fast) { ... }
    ```

    等价 do-while:

    ```cpp
    int slow = 0, fast = 0;
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    ```

    - **do-while 版**更 canonical (跟 [0142](../0142-linked-list-cycle-ii/README.md) 一致).
    - **Yang 版更简洁** — "先动一步, 再 while 比" 避免 `slow == fast == 0` 时假触发退出.

    > **do-while 或"手动预走一步"** 都是 Floyd 起点相同时的通用招. 记住语义就行.

6. **🔑 为啥用 nums[0] 作起点? / Why start at nums[0]**

    - **不能起 0** (若起 0, `slow = 0` 是**环外**节点, 后续要走 nums[0] 才进图).
    - Yang 的 `slow = nums[0]` 相当于"跳过起点 0 直接进入 nums 定义的图".
    - 也可以 `slow = 0, fast = 0` 起手, 然后 do-while.

    > **起点 0 (即 index 0) 保证不在环里** — 因为 nums[i] ∈ [1, n], **没人指向 0**. 所以从 0 出发能保证 Phase 2 的"起点 vs 相遇点" 关系跟 [0142](../0142-linked-list-cycle-ii/README.md) 一致.

7. **🔑 跟 Floyd 家族的完整关系 / Floyd family**

    | 题 | 载体 | 结构 |
    |---|---|---|
    | [0141 Linked List Cycle](../0141-linked-list-cycle/README.md) | 链表 | Phase 1 (判环) |
    | [0142 Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) | 链表 | Phase 1 + 2 (找入口) |
    | [0202 Happy Number](../../03-hash-table/0202-happy-number/README.md) | 数字迭代 (squareSum) | Phase 1 (判环, 1 是自环) |
    | **0287 (本题)** | **数组 (值作 next)** | **Phase 1 + 2 (入口 = 重复数)** |

    > **Floyd 一族 4 题, 3 种载体**. 学一得四. **本题是 Floyd 的"神应用" 因为它把数组问题化到抽象链表问题**.

8. **🔑 复杂度 O(n) 时间, O(1) 空间 / Linear, constant**

    - Time: Phase 1 ≤ 2n, Phase 2 ≤ n.
    - Space: 2 int 指针.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int findDuplicate(vector<int>& nums) {
            // Phase 1: 找相遇点
            int slow = nums[0], fast = nums[nums[0]];       // 手动走 1/2 步
            while (slow != fast) {
                slow = nums[slow];
                fast = nums[nums[fast]];
            }
            // Phase 2: 找环入口 = 重复数
            slow = 0;
            while (slow != fast) {
                slow = nums[slow];
                fast = nums[fast];
            }
            return slow;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findDuplicate(self, nums: list[int]) -> int:
            # do-while 版 — 起点相同用无限循环 + break 表达
            slow = fast = 0
            while True:
                slow = nums[slow]
                fast = nums[nums[fast]]
                if slow == fast:
                    break
            # Phase 2
            slow = 0
            while slow != fast:
                slow = nums[slow]
                fast = nums[fast]
            return slow
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @return {number}
     */
    var findDuplicate = function(nums) {
        // Phase 1
        let slow = nums[0], fast = nums[nums[0]];
        while (slow !== fast) {
            slow = nums[slow];
            fast = nums[nums[fast]];
        }
        // Phase 2
        slow = 0;
        while (slow !== fast) {
            slow = nums[slow];
            fast = nums[fast];
        }
        return slow;
    };
    ```

## Complexity

- **Time**: O(n) — Phase 1 ≤ 2n, Phase 2 ≤ n.
- **Space**: O(1) — 两 int 指针.

## 相关题目

- [0142. Linked List Cycle II](../0142-linked-list-cycle-ii/README.md) — **Floyd 母题**, 本题的抽象来源
- [0141. Linked List Cycle](../0141-linked-list-cycle/README.md) — Phase 1 only, 判环
- [0202. Happy Number](../../03-hash-table/0202-happy-number/README.md) — Floyd 用在数字序列
- [0019. Remove Nth Node From End of List](../0019-remove-nth-node-from-end-of-list/README.md) — 快慢差 n
- [0002. Add Two Numbers](../0002-add-two-numbers/README.md) — 链表加法
- [0021. Merge Two Sorted Lists](../0021-merge-two-sorted-lists/README.md) — 双指针合并
- 0041\. First Missing Positive (待补) — 数组值作下标, 原地标记
- 0448\. Find All Numbers Disappeared in an Array (待补) — 同款"值作下标" 抽象
- 0645\. Set Mismatch (待补) — 找重复 + 缺失
- 0442\. Find All Duplicates in an Array (待补) — 允许改数组 (可用就地标记)
