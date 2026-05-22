# 0253. Meeting Rooms II / 会议室 II

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Greedy, Heap, Sort, Sweep Line, Interval · 贪心, 堆, 排序, 扫描线, 区间
    - **Link**: [LeetCode](https://leetcode.com/problems/meeting-rooms-ii/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given meeting intervals `[start, end)`, return the minimum number of rooms needed so all meetings can run simultaneously.

**中文**: 给一组会议 `[start, end)`, 求并行举办所有会议所需的最少会议室数.

## Key Insights

1. **核心: "同一时刻最大并发数" = 最少房间数 / Max concurrent meetings = min rooms**

    问题本质是**求任意时刻最多有多少会议在进行**. 两种自然角度:

    - **堆 (按时间事件追踪每个房间)**: 给每个房间一个"最早结束时间", 用 min-heap 维护.
    - **扫描线 / 差分 (只看时间轴上 +1 / -1 事件)**: 起点 = +1 房间占用, 终点 = -1 释放. 时间扫一遍取 max.

    两种都是 O(n log n), 选哪个看口味.

2. **堆解法: 按 start 排 + min-heap of end times / Min-heap approach**

    ```
    sort by start asc;
    heap = min-heap of end times;
    for each meeting [s, e]:
        if heap.top() <= s:   // 最早结束的房间已空 → 复用
            heap.pop();
        heap.push(e);
    return heap.size();
    ```

    每来一个会议: 若**最早结束的房间已空** (其 end ≤ 当前 start) → 复用 (pop 旧 end, push 新 end); 否则**开新房间** (直接 push, heap 长大 1). **heap.size() 即答案**.

    Yang 的写法用了独立 `rooms` 计数器 + `<` 严格判定. 等价但稍隐式: `<` 意味着"恰好接上 (start == end)" 算复用 — 对应 `[s, e)` 半开区间语义.

    > 一句话: **每个房间 = 一个 end 时间, heap 帮你 O(log n) 拿"最快空出来的那个"**.

3. **扫描线解法: starts/ends 分两个数组 + 双指针 / Sweep-line approach**

    ```
    sort(starts); sort(ends);
    i = j = rooms = maxRooms = 0;
    while (i < n):
        if starts[i] < ends[j]:           // 一个会议开始 (在最早结束之前)
            rooms++; maxRooms = max(...);
            i++;
        else:                              // 一个会议结束
            rooms--;
            j++;
    return maxRooms;
    ```

    把 n 个起点和 n 个终点分别 sort, **不再保留谁是谁** — 因为只关心"时间轴上 +1 / -1 事件". 这本质是**事件 sort + 差分**.

    `starts[i] < ends[j]` 用 `<` (不是 `<=`): start == end 算"先 end 再 start" (`[s, e)` 半开区间, 端点接上可以复用). 改成 `<=` 就是闭区间语义, 端点重叠算冲突.

4. **两种解法等价 / Why they're equivalent**

    - 堆的"复用 / 开新房" = 扫描线的"end 事件 + start 事件 (差额不动)" vs "start 事件 (+1)".
    - 堆的 `heap.size()` 终值 = 扫描线的 `maxRooms`.

    两者本质都是**事件驱动**的并发计数, 只是数据结构不同.

5. **跟其它区间题的位置 / Where it sits in the interval family**

    | 题 | 求什么 | 经典做法 |
    |---|---|---|
    | [0056 合并](../0056-merge-intervals/README.md) | 合并重叠 | sort by left + 维护右端 |
    | [0435 不重叠](../0435-non-overlapping-intervals/README.md) | 删最少使不重叠 | sort by right + 计数 |
    | [0452 气球](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) | 最少分组数 | sort by right + 维护右端 |
    | **0253 (本题)** | **最大并发数** | **heap 或扫描线** |

    > 一句话: **合并/分组**按"区间排" + 单指针; **并发计数**按"时间事件排" + 堆/差分.

## Solution

=== "C++"
    === "v1 推荐: heap"
        ```cpp
        class Solution {
        public:
            int minMeetingRooms(vector<vector<int>>& intervals) {
                sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
                    return a[0] < b[0];                            // start 升序
                });
                priority_queue<int, vector<int>, greater<int>> ends; // min-heap of end times
                int rooms = 0;
                for (auto& interval : intervals) {
                    if (ends.empty() || interval[0] < ends.top()) { // 最早结束的房间还没空 → 新开
                        rooms++;
                    } else {
                        ends.pop();                                // 复用: 弹掉那个房间的旧 end
                    }
                    ends.push(interval[1]);                        // 当前会议占用某个房间, 更新该房间 end
                }
                return rooms;                                      // 等价 ends.size()
            }
        };
        ```

    === "v2: 扫描线 / 差分"
        ```cpp
        class Solution {
        public:
            int minMeetingRooms(vector<vector<int>>& intervals) {
                int n = intervals.size();
                vector<int> starts(n), ends(n);
                for (int i = 0; i < n; i++) {
                    starts[i] = intervals[i][0];
                    ends[i]   = intervals[i][1];
                }
                sort(starts.begin(), starts.end());                // 起点终点分别 sort, 不再保留谁是谁
                sort(ends.begin(), ends.end());
                int rooms = 0, maxRooms = 0, i = 0, j = 0;
                while (i < n) {
                    if (starts[i] < ends[j]) {                     // 下一个事件是"开始" → +1
                        rooms++;
                        maxRooms = max(maxRooms, rooms);
                        i++;
                    } else {                                       // 下一个事件是"结束" → -1
                        rooms--;
                        j++;
                    }
                }
                return maxRooms;
            }
        };
        ```

=== "Python"
    ```python
    import heapq

    class Solution:
        def minMeetingRooms(self, intervals: list[list[int]]) -> int:
            # heapq 是 min-heap, 直接 push end 时间; 等价 C++ 的 priority_queue<int, vector<int>, greater<int>>
            intervals.sort(key=lambda x: x[0])
            ends = []
            for s, e in intervals:
                if ends and ends[0] <= s:                          # heapq[0] = min, 等价 C++ top()
                    heapq.heappop(ends)                            # 复用最早空的房间
                heapq.heappush(ends, e)
            return len(ends)                                       # 房间数 = heap 最终大小
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number[][]} intervals
     * @return {number}
     */
    var minMeetingRooms = function(intervals) {
        // JS 没有内建 heap, 这里用扫描线 (差分) 解法, 实现更直白
        const n = intervals.length;
        const starts = intervals.map(x => x[0]).sort((a, b) => a - b);  // map + sort with compareFn
        const ends   = intervals.map(x => x[1]).sort((a, b) => a - b);  // compareFn 必须给, 否则字典序错
        let rooms = 0, maxRooms = 0, i = 0, j = 0;
        while (i < n) {
            if (starts[i] < ends[j]) {
                rooms++;
                maxRooms = Math.max(maxRooms, rooms);
                i++;
            } else {
                rooms--;
                j++;
            }
        }
        return maxRooms;
    };
    ```

## Complexity

- **Time**: O(n log n) — sort 主导; heap 操作 O(log n) × n.
- **Space**: O(n) — heap / starts+ends 数组.

## 相关题目

- [0056. Merge Intervals](../0056-merge-intervals/README.md) — 合并重叠区间
- [0435. Non-overlapping Intervals](../0435-non-overlapping-intervals/README.md) — 删最少使不重叠
- [0452. Minimum Number of Arrows to Burst Balloons](../0452-minimum-number-of-arrows-to-burst-balloons/README.md) — 最少箭 / 最少分组
- 0252\. Meeting Rooms (待补) — 本题简化版, 只问能不能开完
- 1851\. Minimum Interval to Include Each Query (待补) — 同款 heap + 区间
- 0630\. Course Schedule III (待补) — heap + 区间贪心
