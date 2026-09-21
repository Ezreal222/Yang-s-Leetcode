# 0442. Find All Duplicates in an Array / 数组中重复的数据

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Array, Hash Table, In-place Marking · 数组, 哈希表, 原地标记
    - **Link**: [LeetCode](https://leetcode.com/problems/find-all-duplicates-in-an-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Values are in `[1, n]`, array length is n** — use the **array itself as a hash table**. Visit each `x`: flip the sign at index `|x| - 1`. If that slot was **already negative**, `|x|` has appeared before → push to result. **O(n) time, O(1) extra space**.
>
> **中文**: **值域 `[1, n]`, 数组长 n** — 把**数组本身当哈希**. 遍历时 `nums[|x|-1]` 取反标记; 若**已负**说明 `|x|` 之前出现过 → 加入结果. **O(n) 时间, O(1) 额外空间**.
>
> *Template / 模版*: **Index-as-key marking** — 值域紧配数组长时的空间黑魔法.

## Problem

**EN**: 数组 `nums` 长度 n, 每个值 ∈ [1, n], 每个值最多出现**两次**. 返回所有出现两次的值. **要求 O(1) 额外空间 & O(n) 时间**.

**中文**: 找所有出现两次的数, O(n) + O(1) 额外空间.

## Key Insights

1. **🔑 灵魂: 值域 `[1, n]` 恰配数组长度 → 值-下标一一对应 / Value fits into index**

    - 值域 `[1, n]` 减 1 后 → `[0, n-1]`, 恰是数组合法下标.
    - **`value x` 对应 `index x - 1`**. 数组本身就是一张**长度 n 的哈希表**, 每个"槽"对应一个值.

    → 这是所有 "index-as-key" 招式的前提. 一旦值域超出 `[0, n]`, 就用不了.

    > **面试若见"值 ∈ [1, n], 长 n, O(1) 空间"** 这三特征齐全 → **必用负号标记 / 位标记 / swap-to-position**.

2. **🔑 灵魂: 用符号位当"访问标记" / Sign bit as visited flag**

    每个 int 有 32 位, 值域 `[1, n]` (**都是正数, 无 0**) 只用低 log₂n 位. **符号位是白送的一位**, 用来记"这个槽被访问过".

    - 第一次见 `x`: `nums[|x|-1]` 还是正 → **flip 成负**.
    - 第二次见 `x`: `nums[|x|-1]` 已是负 → `x` 是重复值, 记录.

    → 用**已有位**代替额外空间. 前提: 原值都 > 0, 负号不冲突.

    > **"符号位当 flag" 是数组类的经典空间省法**. 也见 0448 Find All Numbers Disappeared in an Array (待补), [0287 Duplicate](../../02-linked-list/0287-find-the-duplicate-number/README.md) 的另一系解法.

3. **🔑 灵魂: 遍历时始终读 `abs(x)` — 因为 x 自己可能被翻负 / Always read absolute value**

    ```cpp
    for (int x : nums) {
        int idx = abs(x) - 1;                       // ⚠️ abs 必需
        if (nums[idx] < 0) res.push_back(abs(x));
        else nums[idx] = -nums[idx];
    }
    ```

    **易错**: 若之前某轮把 `nums[k]` 翻负了, 后面遍历到 `nums[k]` 时读到的**是负数**. 不加 `abs` 就用错误的下标了.

    → **不变量**: `|nums[i]|` 始终代表**原始值**. Sign 只作 flag, 不改语义.

    > **易错点 top 1**: 忘写 `abs`. 一漏就大概率数组越界或死循环.

4. **🔑 为啥每个值最多出现 2 次能保证正确 / Correctness under ≤2 occurrences**

    - 第一次见 `x`: 翻负 (标记).
    - 第二次见 `x`: 发现已负 → 记录.
    - **第三次见 `x`**: 又发现已负 → **再次记录** → 结果重复!

    题目保证**至多 2 次**, 所以只可能记录一次. 若题目改成"≥ 2 次", 需**额外去重** 或改**记录时再翻正** 让第三次不重复:

    ```cpp
    if (nums[idx] < 0) {
        res.push_back(abs(x));
        nums[idx] = abs(nums[idx]);              // 翻回正, 防止第三次再录
    } else {
        nums[idx] = -nums[idx];
    }
    ```

    > **易错点 top 2**: 题目变体没读完就套模板, 三次出现的场景要多加"翻正".

5. **🔑 复杂度分析 / Complexity**

    - **Time**: O(n) — 一遍扫.
    - **Space**: O(1) — 只用 res (输出不算) + 几个变量.

    → 严格 O(1) extra space, 满足题目要求.

6. **🔑 破坏原数组 vs 保留原数组 / Modifies input — restorable if needed**

    此法**修改了 nums**. 若面试官追问"能否保留原数组?":

    - 遍历结束后**再遍历一遍** 把所有负数翻回正: O(n) 一次.
    - 或**用另一种标记招**: 加 n 而非翻负 (`nums[idx] += n`, 判 `> n` 即访问过). 值域变大, 需保证不溢出.

    > **"原地修改" 是 O(1) 空间的隐形代价**. 面试主动提, 加分.

7. **🔑 替代解法对比 / Alternative approaches**

    | 方法 | Time | Extra Space | 特点 |
    |---|---|---|---|
    | **负号标记** (Yang) | O(n) | **O(1)** | 修改数组, 需值 ∈ [1, n] |
    | +n 标记 | O(n) | O(1) | 修改数组, 无负号冲突 |
    | Swap-to-position | O(n) | O(1) | 一样, 但每个值放到 `idx = val - 1` 位置 |
    | Hash set | O(n) | **O(n)** | 通用, 值域不限 |
    | 排序 + 扫相邻 | O(n log n) | O(1) | 违反 O(n) 时间 |

    > **负号标记 = 值域限定题的黄金解**. 值域没限就 hash set.

8. **🔑 相关变体 / Related variants**

    - **0448 找所有缺失的数字**: 同款负号标记. 扫完后**仍为正**的下标 = 缺失值.
    - **0041 First Missing Positive**: 值域含负 / > n, 需先 swap-to-position 归位.
    - **0287 Find the Duplicate Number**: 恰有 1 个重复但可能出现多次, Floyd 环检测.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> findDuplicates(vector<int>& nums) {
            vector<int> res;
            for (int x : nums) {
                int idx = abs(x) - 1;                             // ⚠️ 必须 abs
                if (nums[idx] < 0) res.push_back(abs(x));         // 已标记 → 重复
                else nums[idx] = -nums[idx];                      // 第一次见 → 翻负
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def findDuplicates(self, nums: list[int]) -> list[int]:
            # Python 天然支持 abs / 负数, 逻辑跟 C++ 同
            # 但注意 Python int 无溢出问题, 不像 C++ 的 -INT_MIN
            res = []
            for x in nums:
                idx = abs(x) - 1
                if nums[idx] < 0:
                    res.append(abs(x))                            # 已负 → 重复
                else:
                    nums[idx] = -nums[idx]                        # 首见 → 翻负
            return res
    ```

=== "JavaScript"
    ```javascript
    var findDuplicates = function(nums) {
        // JS 的 Math.abs 处理负 int
        // 数组元素默认可读写, 直接原地修改
        const res = [];
        for (const x of nums) {
            const idx = Math.abs(x) - 1;                          // 必须 abs
            if (nums[idx] < 0) {
                res.push(Math.abs(x));                            // 已标记
            } else {
                nums[idx] = -nums[idx];                           // 翻负
            }
        }
        return res;
    };
    ```

=== "C++ (v2: swap-to-position, 也 O(1) 空间)"
    ```cpp
    // 每个值 x 放到 nums[x-1]. 遍历时若 nums[i] 已在正确位置且 nums[i] != i+1, 说明重复
    class Solution {
    public:
        vector<int> findDuplicates(vector<int>& nums) {
            int n = nums.size();
            for (int i = 0; i < n; i++) {
                while (nums[i] != nums[nums[i] - 1])              // 归位, 至多 n 次总交换
                    swap(nums[i], nums[nums[i] - 1]);
            }
            vector<int> res;
            for (int i = 0; i < n; i++)
                if (nums[i] != i + 1) res.push_back(nums[i]);    // 未归位 = 重复
            return res;
        }
    };
    ```

## Complexity

| 版本 | Time | Space |
|---|---|---|
| **负号标记** | **O(n)** | **O(1)** |
| Swap-to-position | O(n) 摊销 | O(1) |
| Hash set | O(n) | O(n) |

## 相关题目

- [0287. Find the Duplicate Number](../../02-linked-list/0287-find-the-duplicate-number/README.md) — 保证 1 个重复, Floyd 环检测
- 0448\. Find All Numbers Disappeared in an Array (待补) — 同款负号标记, 找**缺失**
- 0041\. First Missing Positive (待补) — swap-to-position 母题
- 0268\. Missing Number (待补) — 单缺失, XOR / 求和差
- 0136\. Single Number (待补) — 异或消对
- 0645\. Set Mismatch (待补) — 找重复 + 缺失, 同款标记
- 0217\. Contains Duplicate (待补) — 判是否含重复
