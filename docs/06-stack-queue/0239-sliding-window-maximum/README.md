# 0239. Sliding Window Maximum / 滑动窗口最大值

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Monotonic Queue, Sliding Window, Deque · 单调队列, 滑动窗口, 双端队列
    - **Link**: [LeetCode](https://leetcode.com/problems/sliding-window-maximum/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given an integer array and window size `k`, slide the window from left to right and return the max of each window.

**中文**: 给定数组和窗口大小 `k`, 窗口从左滑到右, 返回每个窗口的最大值数组.

## 思路

### Why not brute / 暴力解为什么不行

**EN**: 每个窗口扫一遍 → O(n·k). k 大就炸. 而堆 (priority queue) 删旧元素难, 退化成 O(n log n) 还要懒删除. 真正 O(n) 的解是**单调队列**.

**中文**: 暴力 O(n·k), 堆 O(n log n) 且删除老元素麻烦. 单调队列才是 O(n) 解.

### Core idea / 核心思路

**EN**: Don't store every window element — only keep elements that **could still become the max**. If a new element `x` enters and beats some older smaller element `y` already in the queue, `y` will never be the max again (it has a younger and bigger neighbor that will outlast it in any future window). So **drop `y`**.

Maintain a deque that's **monotonically non-increasing from front to back**:
- Front = current window max.
- Back = the most recent element (or what's left after kicking smaller predecessors).

**中文**: 队列不需要装窗口里所有元素, 只装"可能还会成为最大值"的. 新进来一个 `x`, 把队尾比它小的全踢掉 (反正它们比 `x` 旧又比 `x` 小, 永远轮不到当最大值). 保持队列从队头到队尾**单调递减**, 队头永远是当前窗口最大值.

### Two ops / 两个操作

| op | when | what |
|---|---|---|
| `push(x)` | 新元素进窗口 | 从队尾把 `< x` 的元素全 `pop_back`, 然后 `push_back(x)` |
| `pop(v)` | 老元素离开窗口 | 如果队头 `== v`, `pop_front`; 否则不动 (老元素早被 push 阶段踢掉了) |

每个元素**最多入队一次、出队一次** → 总操作 O(n), 均摊 O(1).

Key invariant / 关键不变量: 任何时刻, deque 从 front 到 back **严格递减** (相等也可以保留, 看实现), 所以 `front()` 就是窗口最大值.

### Trace / 完整状态: `nums = [1,3,-1,-3,5,3,6,7], k = 3`

| step | i | nums[i] | window | mono deque (front → back) | output |
|------|---|---------|---|---|--------|
| init | 0 | 1  | —          | `[1]`                  | —     |
| init | 1 | 3  | —          | `[3]` (1 popped, < 3)  | —     |
| init | 2 | -1 | `[1,3,-1]` | `[3,-1]`               | **3** |
| slide | 3 | -3 | `[3,-1,-3]` | `[3,-1,-3]` (front≠1, no front-pop) | **3** |
| slide | 4 | 5  | `[-1,-3,5]` | `[5]` (front=3 popped because old 3 leaves; then -1,-3 < 5 popped) | **5** |
| slide | 5 | 3  | `[-3,5,3]` | `[5,3]` (front≠-1, no front-pop)| **5** |
| slide | 6 | 6  | `[5,3,6]`  | `[6]` (front≠-3; then 3, 5 < 6 popped) | **6** |
| slide | 7 | 7  | `[3,6,7]`  | `[7]` (front=6, but front≠5 so no front-pop¹; then 6 < 7 popped) | **7** |

¹ At i=7 the leaving element is `nums[4]=5` but the front of the deque is already `6`, so the value-based `pop(5)` is a no-op — `5` was kicked out long ago when `6` arrived.

Final: `[3, 3, 5, 5, 6, 7]` ✓

## Solution

=== "Python"
    Idiomatic Python: store **indices** in the deque (not values). Then "is the front out of window?" becomes a one-line index check, no need for the C++ value-equality dance.

    ```python
    from collections import deque

    class Solution:
        def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
            dq: deque[int] = deque()   # indices; nums[dq[0]] is current window max
            res: list[int] = []
            for i, x in enumerate(nums):
                # 1) keep deque monotonically decreasing — kick smaller tails
                while dq and nums[dq[-1]] < x:
                    dq.pop()
                dq.append(i)
                # 2) front out of window?
                if dq[0] <= i - k:
                    dq.popleft()
                # 3) once first window is full, start emitting
                if i >= k - 1:
                    res.append(nums[dq[0]])
            return res
    ```

=== "C++"
    把单调队列封装成 `MyQueue` (Carl 风格), `push`/`pop` 接口和题面对得上. 队列里存**值**, 所以 `pop(v)` 要查值是否等于队头.

    ```cpp
    class Solution {
    private:
        class MyQueue {
        public:
            deque<int> que;
            // 出口元素 == 待移除值才弹, 否则它早被 push 阶段踢掉了
            void pop(int value) {
                if (!que.empty() && value == que.front()) {
                    que.pop_front();
                }
            }
            // 维持单调递减: 队尾比 value 小的都踢掉, 再把 value 放进队尾
            void push(int value) {
                while (!que.empty() && value > que.back()) {
                    que.pop_back();
                }
                que.push_back(value);
            }
            int front() { return que.front(); }
        };
    public:
        vector<int> maxSlidingWindow(vector<int>& nums, int k) {
            MyQueue que;
            vector<int> res;
            for (int i = 0; i < k; i++) que.push(nums[i]);
            res.push_back(que.front());
            for (int i = k; i < (int)nums.size(); i++) {
                que.pop(nums[i - k]);   // 移除窗口左边离开的元素
                que.push(nums[i]);       // 加入窗口右边新进来的元素
                res.push_back(que.front());
            }
            return res;
        }
    };
    ```

=== "Java"
    ```java
    // TBD
    ```

## Complexity

- **Time**: O(n) — each element enters and leaves the deque at most once. 均摊 O(1) per step.
- **Space**: O(k) — deque holds ≤ k elements at any time. Output O(n - k + 1) extra.

## 易错点

- **Python 存下标更顺**: 存值就要像 C++ 一样在 `pop` 时比较值, 一旦窗口里有重复值容易绕. 存下标后 "队头出窗口?" 就是 `dq[0] <= i - k`, 一行搞定.
- **`while` not `if` in push**: 入队时要把队尾**所有**比新值小的都踢掉, 不是只踢一个. 写成 `if` 会留下不该留的.
- **Pop 是值匹配, 不是无脑弹**: 老元素离开窗口时, 它**可能早就被踢掉了**(被某个更大的后来者). 所以只在队头 == 离开值时才 `pop_front`.
- **维护单调"递减"还是"非增"?** 严格递减 (kick equals too) 也对, 但保留相等更省一点踢的开销, 结果一样 (反正最大值在队头).
- **一开始的填窗**: Python 版本用 `i >= k-1` 起步发结果, 不需要专门跑一遍前 k 个; C++ 版本分两段写 (Carl 风格) 也行, 看个人.
- **不要用堆**: O(n log k), 删除老元素只能"懒删除"(top 是失效的就丢, 直到合法). 能写但慢且乱. 单调队列才是教科书答案.

## 相关题目

- [0150. Evaluate Reverse Polish Notation / 逆波兰表达式求值](../0150-evaluate-reverse-polish-notation/README.md) — 栈/队列处理序列
- 0496. Next Greater Element I (待补) — 单调**栈**入门
- 0503. Next Greater Element II (待补) — 循环数组上的单调栈
- 0862. Shortest Subarray with Sum at Least K (待补) — 单调队列 + 前缀和
