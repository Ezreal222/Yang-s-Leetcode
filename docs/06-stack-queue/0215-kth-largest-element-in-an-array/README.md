# 0215. Kth Largest Element in an Array / 数组中的第 K 个最大元素

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Heap, Quickselect, Partition, Divide & Conquer · 堆, 快速选择, 分治
    - **Link**: [LeetCode](https://leetcode.com/problems/kth-largest-element-in-an-array/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **k-th largest in array** → **v1 min-heap of size k** (O(n log k), 通用); **v2 quickselect + random pivot** (avg O(n), best worst case). Target index = `n - k` (0-indexed after sort).
>
> **中文**: **数组第 k 大** → **v1 大小 k 的最小堆** (O(n log k), 通用); **v2 快速选择 + 随机 pivot** (期望 O(n)). 目标下标 = 排序后 `n - k`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 未排序数组 `nums`, 返回**第 k 大**的元素. 不能真的排序 (要 O(n) / O(n log k)).

**中文**: 无需完全排序, 求第 k 大.

## Key Insights

1. **🔑 三种方法对比 / Three approaches**

    | 方法 | Time | Space | 稳定? | 适用 |
    |---|---|---|---|---|
    | 排序 + 取 nums[n-k] | O(n log n) | O(1) | 稳 | 最简 (面试暖场) |
    | **v1 min-heap of size k** | **O(n log k)** | O(k) | 稳 | k ≪ n / 流式 |
    | **v2 quickselect** | **O(n) 期望** | O(1) | **不稳** (最坏 O(n²)) | 面试秀 |

    > **三种都会 = 灵活**. 面试问 "还能优化吗?" 就从 sort → heap → quickselect 三级进化.

2. **🔑 v1 min-heap of size k (跟 [0347](../0347-top-k-frequent-elements/README.md) 同款) / Same as 0347's Variant B**

    ```cpp
    priority_queue<int, vector<int>, greater<int>> pq;  // min-heap
    for (int x : nums) {
        pq.push(x);
        if (pq.size() > k) pq.pop();                    // 超 k 就踢最小
    }
    return pq.top();                                    // 堆顶 = 第 k 大
    ```

    - **为啥 min-heap 而非 max-heap?** 维护"目前最大的 k 个", **踢的是最小** — 用 min-heap 顶查 O(1), 踢 O(log k).
    - **push 后立即 pop** — 保持 size ≤ k.
    - 结束堆顶就是**第 k 大** (堆里其他 k-1 个都比它大).

    > **Top-K 模板**: for each x, push + if size > k → pop. 通用套路 (跟 [0347](../0347-top-k-frequent-elements/README.md) / 0703 / 0973 同源).

3. **🔑 v2 quickselect: 分治的"半个 quicksort" / Quickselect: half of quicksort**

    quicksort 每次 partition 后**两半都递归**; **quickselect 只递归包含目标的一半** — 少一半工作 → 平均 O(n).

    ```
    target = n - k                     # 第 k 大 = 排序后第 (n-k) 位
    while l <= r:
        p = partition(l, r)
        if p == target: return nums[p]
        elif p < target: l = p + 1     # target 在右半
        else: r = p - 1                # target 在左半
    ```

    - **不排序**, 只找**第 target 位**上的元素.
    - **随机化 pivot** 防止最坏 O(n²) (对已排序输入).

4. **🔑 partition (Lomuto scheme) 逐行 / Partition breakdown**

    ```cpp
    int partition(vector<int>& nums, int l, int r) {
        int p = l + rand() % (r - l + 1);
        swap(nums[p], nums[r]);          // 随机 pivot 放最右
        int pivot = nums[r], i = l;
        for (int j = l; j < r; j++) {
            if (nums[j] < pivot) swap(nums[i++], nums[j]);  // < 的放左边
        }
        swap(nums[i], nums[r]);          // pivot 归位到 i
        return i;                        // 返 pivot 最终位置
    }
    ```

    - **`i` 是"下一个 < pivot 元素" 的目标位置**.
    - **j 扫描**: 遇到 < pivot 的元素 → swap 到 nums[i], i++.
    - **结束状态**: `[l..i-1] < pivot`, `[i..r-1] >= pivot`.
    - **最后 swap pivot 到 i** — pivot 就位, 左边全 <, 右边全 >=.

    > **Lomuto 分区**是最好写的一种. 还有 **Hoare** (更快但更绕). 面试 Lomuto 够用.

5. **🔑 为啥 `target = n - k` 是"第 k 大"? / Why target = n - k**

    排序**升序** 后: 最小在 0, 最大在 n-1. **第 k 大** = 从大到小第 k 个 = 升序第 (n-k) 位 (0-indexed).

    ```
    排序:  [1, 2, 3, 4, 5], n = 5
    第 1 大 = 5 = nums[4] = nums[n-1]
    第 2 大 = 4 = nums[3] = nums[n-2]
    第 k 大 = nums[n-k]
    ```

    quickselect 只需**保证 target 位** 上的元素是"排序后本该在那里的" — 其他位置乱序无所谓.

6. **🔑 随机化 pivot 的作用 / Random pivot**

    - **无随机化**: 已排序或近排序输入时 pivot 总是最小/最大 → 每次 partition 只减 1 → **最坏 O(n²)**.
    - **随机化**: 期望每次约在中间 → **均摊 O(n)**.

    ```cpp
    int p = l + rand() % (r - l + 1);
    swap(nums[p], nums[r]);
    ```

    > **面试 quickselect 必须**随机化. 少写会被抓最坏 case.

7. **🔑 复杂度对比 / Complexity summary**

    | | Time (avg) | Time (worst) | Space |
    |---|---|---|---|
    | v1 heap | O(n log k) | O(n log k) | O(k) |
    | **v2 quickselect** | **O(n)** | O(n²) 无随机 / **O(n) 随机** | O(log n) 递归栈 (迭代版 O(1)) |

    - **v1 稳定**, 流式数据也能扛.
    - **v2 更快** 但 worst case 差 (随机化让不发生).

## Solution

=== "C++"

    **v1: min-heap of size k (推荐首选)**

    ```cpp
    class Solution {
    public:
        int findKthLargest(vector<int>& nums, int k) {
            priority_queue<int, vector<int>, greater<int>> pq;      // min-heap
            for (int x : nums) {
                pq.push(x);
                if ((int)pq.size() > k) pq.pop();                    // 超 k 就踢最小
            }
            return pq.top();                                         // 堆顶 = 第 k 大
        }
    };
    ```

    **v2: quickselect (期望 O(n), 面试秀)**

    ```cpp
    class Solution {
    public:
        int findKthLargest(vector<int>& nums, int k) {
            int target = nums.size() - k;
            int l = 0, r = nums.size() - 1;
            while (l <= r) {
                int p = partition(nums, l, r);
                if (p == target) return nums[p];
                else if (p < target) l = p + 1;
                else r = p - 1;
            }
            return -1;
        }
    private:
        int partition(vector<int>& nums, int l, int r) {
            int p = l + rand() % (r - l + 1);
            swap(nums[p], nums[r]);                                  // 随机 pivot 放尾
            int pivot = nums[r], i = l;
            for (int j = l; j < r; j++) {
                if (nums[j] < pivot) swap(nums[i++], nums[j]);
            }
            swap(nums[i], nums[r]);                                  // pivot 归位
            return i;
        }
    };
    ```

=== "Python"
    ```python
    import heapq
    import random

    class Solution:
        # v1: heap — Python heapq 只有 min-heap, 天然适合 Top-K largest
        def findKthLargest(self, nums: list[int], k: int) -> int:
            # heapq.nlargest(k, nums)[-1] 一行, 内部就是本题的 min-heap of size k
            return heapq.nlargest(k, nums)[-1]

        # v2: quickselect 手写
        def findKthLargest_qs(self, nums: list[int], k: int) -> int:
            target = len(nums) - k
            def partition(l, r):
                p = random.randint(l, r)
                nums[p], nums[r] = nums[r], nums[p]
                pivot, i = nums[r], l
                for j in range(l, r):
                    if nums[j] < pivot:
                        nums[i], nums[j] = nums[j], nums[i]
                        i += 1
                nums[i], nums[r] = nums[r], nums[i]
                return i

            l, r = 0, len(nums) - 1
            while l <= r:
                p = partition(l, r)
                if p == target: return nums[p]
                elif p < target: l = p + 1
                else: r = p - 1
            return -1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[]} nums
     * @param {number} k
     * @return {number}
     */
    var findKthLargest = function(nums, k) {
        // v1: JS 无原生堆. 排序 O(n log n) 简单 (对小数据够用)
        // 手写堆或用 priority queue lib. 这里最简单:
        return nums.sort((a, b) => b - a)[k - 1];

        // v2 quickselect 手写 (面试更优):
        // ... (跟 C++ 结构一致, JS 递归/迭代都行)
    };
    ```

## Complexity

| Version | Time | Space |
|---|---|---|
| **v1 heap** | O(n log k) | O(k) |
| **v2 quickselect** | **O(n) 期望** | O(1) 迭代 / O(log n) 递归 |

## 相关题目

- [0347. Top K Frequent Elements](../0347-top-k-frequent-elements/README.md) — 频次 Top-K, 同款 min-heap 模板
- [0239. Sliding Window Maximum](../0239-sliding-window-maximum/README.md) — 单调队列
- [0155. Min Stack](../0155-min-stack/README.md) — 栈设计
- [0703. Kth Largest Element in a Stream](../0703-kth-largest-element-in-a-stream/README.md) — **流式版**, min-heap of k 直接应用
- [0973. K Closest Points to Origin](../0973-k-closest-points-to-origin/README.md) — 同款 heap of k, key = 距离²
- 0692\. Top K Frequent Words (待补) — 频次 Top-K + 字典序
- 0378\. Kth Smallest Element in a Sorted Matrix (待补) — 二分答案 or 堆
- 0295\. Find Median from Data Stream (待补) — 双堆维护中位数
