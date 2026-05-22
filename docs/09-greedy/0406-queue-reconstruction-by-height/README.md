# 0406. Queue Reconstruction by Height / 根据身高重建队列

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Array, Sort · 贪心, 数组, 排序
    - **Link**: [LeetCode](https://leetcode.com/problems/queue-reconstruction-by-height/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Each person in `people[i] = [h_i, k_i]` represents height `h` and the number of people in front of them with height ≥ `h`. Reconstruct the queue (return the array in the correct order).

**中文**: 每个人 `people[i] = [h, k]`, h 是身高, k 是前面身高 ≥ h 的人数. 重建队列, 返回正确顺序.

## Key Insights

1. **排序: 身高降序 + 同身高 k 升序 / Sort by `(-h, k)`**

    Yang 的双关键: **身高降序优先**, **同身高 k 升序**.

    - 身高降序: 让矮的人插入时, **已经排好的高个子的 k 值不受影响** — 因为新插入的矮人对高个子来说"够不上 ≥", 不算数. 这是整个贪心的灵魂.
    - 同身高 k 升序: 同身高之间互相算数 (≥ 包含等). k 小的应该排前面 (因为它面前的同身高人就少).

2. **插入到下标 k 位置 / Insert at index k**

    sort 完后, 按顺序处理. 处理每个人时, `res` 里**全是 ≥ 他身高的人** (因为先放高个的). 题目说"我前面有 k 个 ≥ 我", 所以直接 `res.insert(begin + k, person)` 就把它放到了正确位置.

    > 关键的不变量: 每次插入后, **已插入的人之间的 k 关系全部满足**. 新人插入不破坏老人, 老人不影响新人的位置选择.

3. **复杂度 O(n²) / Naive insert dominates**

    sort 是 O(n log n), 但 vector `insert` 是 O(n) per call (要 shift 后面的元素), 总 O(n²). 大数据时可以用平衡 BST / 树状数组按"位置 k" 二分插入降到 O(n log n), 但题目数据小没必要.

    Python `list.insert(k, x)` 同样 O(n). 真正的 O(n log n) 实现需要 SortedList (sortedcontainers).

4. **跟 [0455 / 1005] 那种"sort + 阶段处理" 同根 / Sort-first greedy family**

    贪心系列里另一支主流: **先用 sort 把"什么先做" 的优先级定下来**, 然后按顺序施加局部最优操作. 同款思路: [0455 Cookies](../0455-assign-cookies/README.md) (双 sort + 配对), [1005 K Negations](../1005-maximize-sum-of-array-after-k-negations/README.md) (|x| sort + 两阶段), 这题是排序顺序 + 按 k 插入.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
            // 身高降序; 同身高 k 升序
            sort(people.begin(), people.end(), [](const vector<int>& a, const vector<int>& b) {
                if (a[0] != b[0]) return a[0] > b[0];
                return a[1] < b[1];
            });
            vector<vector<int>> res;
            for (auto& person : people) {
                res.insert(res.begin() + person[1], person);             // 插到下标 k
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def reconstructQueue(self, people: list[list[int]]) -> list[list[int]]:
            # key=(-h, k): 身高降序, 同身高 k 升序. 等价 C++ 的双关键比较 lambda
            people.sort(key=lambda p: (-p[0], p[1]))
            res = []
            for p in people:
                res.insert(p[1], p)                                       # list.insert(i, x) 同 C++ vector::insert
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} people
     * @return {number[][]}
     */
    var reconstructQueue = function(people) {
        // 排序双关键: 高度降序优先, 同高度 k 升序
        people.sort((a, b) => a[0] !== b[0] ? b[0] - a[0] : a[1] - b[1]);
        const res = [];
        for (const p of people) {
            // Array.prototype.splice(start, deleteCount, ...items): 在 start 位置插入 items, 删 0 个
            // 等价 C++ vector::insert(begin + k, x)
            res.splice(p[1], 0, p);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n²) — sort O(n log n), insert n 次每次 O(n).
- **Space**: O(n) 输出.

## 相关题目

- [0455. Assign Cookies](../0455-assign-cookies/README.md) — sort + 贪心配对
- [1005. Maximize Sum Of Array After K Negations](../1005-maximize-sum-of-array-after-k-negations/README.md) — sort + 阶段化贪心
- [0135. Candy](../0135-candy/README.md) — 同款"按某种排序后处理" 思路 (但不是排序; 是双向扫)
- 0853\. Car Fleet (待补) — sort + 单调栈, 同根 sort-first 贪心
- 0179\. Largest Number (待补) — 自定义比较器排序 (字符串拼接序)
- 1665\. Minimum Initial Energy to Finish Tasks (待补) — sort 自定义比较 + 累加贪心
