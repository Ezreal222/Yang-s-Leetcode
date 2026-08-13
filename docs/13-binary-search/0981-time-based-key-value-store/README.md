# 0981. Time Based Key-Value Store / 基于时间的键值存储

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Binary Search, Design, Hash Map · 二分查找, 设计, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/time-based-key-value-store/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **`set(key, val, ts)` + `get(key, ts)` returning val whose timestamp ≤ ts (closest)** → **hash map `key → vector<(ts, val)>`**; timestamps increase monotonically → vector always sorted → **binary search for largest `ts_i ≤ query`**.
>
> **中文**: **`set` + `get` 返最接近但 ≤ query 时间戳的 val** → **哈希 `key → 有序 (ts, val) 列表`**; ts 单调递增 → 列表天然有序 → **二分找最大 `ts_i ≤ query`**.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 设计 TimeMap 类, 支持:

- `set(key, value, timestamp)` — 存.
- `get(key, timestamp)` — 返回**该 key 在 ≤ timestamp 时**最大 timestamp 对应的 value; 无则返 "".

保证 **`set` 的 timestamp 严格递增**.

**中文**: 时间戳键值存储, get 返最接近但 ≤ query 的 value.

## Key Insights

1. **🔑 结构: 哈希 `key → 有序列表` / Hash map to sorted list**

    - **外层**: 用 `unordered_map<string, vector<...>>` 按 key 分桶 — get / set 都 O(1) 找到对应 vector.
    - **内层**: `vector<pair<int, string>>` 存 `(ts, val)` 对. 因为 **set 的 ts 单调递增** (题目保证), **push_back 后天然有序** — 无需自排.

    > **"key 独立 + 每 key 内部有序"** 是设计题里**分桶 + 有序** 的常见组合. 无 ts 单调保证时得改用 `map<int, string>` (自平衡 BST) 或每次插入排序.

2. **🔑 灵魂: 二分找"最大 ts ≤ query" / Binary search for largest ts ≤ query**

    经典**变体二分**: 找**最大**满足条件的位置.

    ```cpp
    int l = 0, r = v.size() - 1, ans = -1;
    while (l <= r) {
        int mid = (l + r) / 2;
        if (v[mid].first <= timestamp) {         // 可行, 记候选, 试更右
            ans = mid;
            l = mid + 1;
        } else {                                  // 太大, 收右边界
            r = mid - 1;
        }
    }
    ```

    - **`ans = mid; l = mid + 1;`** 是关键: **命中就记, 继续向右扩** 找更大的可行 ts.
    - **`ans = -1` 兜底**: 所有 ts 都 > query → 无解.

    > **"找最大可行"** 二分模板: `l ≤ r + ans = mid + l = mid + 1`. 对称的 "**找最小可行**" 用 `ans = mid + r = mid - 1` (见 [0875](../0875-koko-eating-bananas/README.md)).

3. **🔑 等价 STL: `upper_bound` 找 > query 的第一个 - 1 / STL alternative**

    ```cpp
    // 用 STL 一行找到答案
    auto it = upper_bound(v.begin(), v.end(), make_pair(timestamp, string("\xff")),
                          [](auto& a, auto& b) { return a.first < b.first; });
    if (it == v.begin()) return "";
    return prev(it)->second;
    ```

    - **`upper_bound`** 返回**第一个 > timestamp** 的迭代器.
    - **`prev(it)`** 是**最后一个 ≤ timestamp**.
    - 若 it 就是 begin, 说明整个都 > query → 无解.

    > **STL 版更短**, 但**需要小心比较器** (因为 pair 默认按 (first, second) 字典序). 面试写 Yang 手撸的更**通用**.

4. **🔑 `set` O(1) — 因为 ts 单调递增 / set is O(1) thanks to monotonic ts**

    ```cpp
    void set(string key, string value, int timestamp) {
        m[key].push_back({timestamp, value});
    }
    ```

    若 ts 不保证递增, `set` 得**二分插入 O(log n)** 或用 `map<int, string>` (自平衡树 O(log n)).

    > 题目给的"单调" 保证是**极大简化**. 读题时**这类"guarantee" 要抓紧** — 决定数据结构选型.

5. **🔑 get 找不到时返 "" / Not-found ⇒ empty string**

    两种"找不到":

    - **key 不存在**: `m.find(key) == m.end()`.
    - **key 存在但所有 ts 都 > query**: 二分后 `ans == -1`.

    两种**都返 ""**. Yang 分开处理清晰.

6. **🔑 复杂度 / Complexity**

    | API | Time | 备注 |
    |---|---|---|
    | `set` | **O(1)** | 摊销 (vector push_back) |
    | `get` | **O(log n_k)** | n_k = 该 key 的 set 次数 |
    | **Space** | O(N) | N = 总 set 次数 |

7. **🔑 跟其他二分变体的对比 / vs other binary variants**

    | 题 | 找什么 | 模板 |
    |---|---|---|
    | [0704](../0704-binary-search/README.md) | 精确 target | `<=` + return mid |
    | [0074](../0074-search-a-2d-matrix/README.md) | 精确 target (2D) | flattened 二分 |
    | [0875](../0875-koko-eating-bananas/README.md) | 最小可行 | `<=` + return left |
    | **0981 (本题)** | **最大可行** | `<=` + **`ans = mid; l = mid + 1`** |
    | [0153](../0153-find-minimum-in-rotated-sorted-array/README.md) | 断点 | `<` + return nums[l] |

    > **变体二分家族**: 找精确 / 找最小可行 / 找最大可行 / 找断点 — 各有小差. 建议对照记.

## Solution

=== "C++"
    ```cpp
    class TimeMap {
        unordered_map<string, vector<pair<int, string>>> m;
    public:
        TimeMap() {}

        void set(string key, string value, int timestamp) {
            m[key].push_back({timestamp, value});                    // O(1) 摊销
        }

        string get(string key, int timestamp) {
            auto it = m.find(key);
            if (it == m.end()) return "";
            auto& v = it->second;

            // 二分找最大的 (ts <= timestamp)
            int l = 0, r = v.size() - 1, ans = -1;
            while (l <= r) {
                int mid = (l + r) / 2;
                if (v[mid].first <= timestamp) {
                    ans = mid;
                    l = mid + 1;                                     // 试更右
                } else {
                    r = mid - 1;
                }
            }
            return ans == -1 ? "" : v[ans].second;
        }
    };
    ```

=== "Python"
    ```python
    from collections import defaultdict
    from bisect import bisect_right

    class TimeMap:
        def __init__(self):
            # defaultdict(list): key 不存在时自动建 list
            self.m: dict[str, list[tuple[int, str]]] = defaultdict(list)

        def set(self, key: str, value: str, timestamp: int) -> None:
            self.m[key].append((timestamp, value))

        def get(self, key: str, timestamp: int) -> str:
            if key not in self.m: return ""
            v = self.m[key]
            # bisect_right 找 > (timestamp, +inf) 的第一个位置; 前一个就是答案
            # 用 (timestamp + 1, '') 作 sentinel 触发"大于 timestamp"
            # 更简洁: 只按 timestamp 二分, 需要维护一个纯 ts 数组
            # 这里用 bisect_right + key 参数 (Python 3.10+) 最 Pythonic
            i = bisect_right(v, (timestamp, chr(127)))       # chr(127) 让 tuple 比较视为最大
            return "" if i == 0 else v[i - 1][1]
    ```

=== "JavaScript"
    ```javascript
    var TimeMap = function() {
        // Map 保留插入顺序 + 数字/字符串 key 类型
        this.m = new Map();
    };

    /**
     * @param {string} key
     * @param {string} value
     * @param {number} timestamp
     * @return {void}
     */
    TimeMap.prototype.set = function(key, value, timestamp) {
        if (!this.m.has(key)) this.m.set(key, []);
        this.m.get(key).push([timestamp, value]);
    };

    /**
     * @param {string} key
     * @param {number} timestamp
     * @return {string}
     */
    TimeMap.prototype.get = function(key, timestamp) {
        if (!this.m.has(key)) return "";
        const v = this.m.get(key);
        let l = 0, r = v.length - 1, ans = -1;
        while (l <= r) {
            const mid = Math.floor((l + r) / 2);
            if (v[mid][0] <= timestamp) {
                ans = mid;
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
        return ans === -1 ? "" : v[ans][1];
    };
    ```

## Complexity

- **`set`**: O(1) 摊销.
- **`get`**: O(log n_k) — n_k = 该 key 的 set 次数.
- **Space**: O(N) — 总 set 数.

## 相关题目

- [0704. Binary Search](../0704-binary-search/README.md) — 标准二分母题
- [0074. Search a 2D Matrix](../0074-search-a-2d-matrix/README.md) — 一次二分 + 坐标映射
- [0875. Koko Eating Bananas](../0875-koko-eating-bananas/README.md) — BSA 最小可行
- [0153. Find Minimum in Rotated Sorted Array](../0153-find-minimum-in-rotated-sorted-array/README.md) — 找断点
- [0033. Search in Rotated Sorted Array](../0033-search-in-rotated-sorted-array/README.md) — 旋转数组找 target
- 0035\. Search Insert Position (待补) — 找最小满足 arr[i] >= target
- [0034. Find First and Last Position of Element in Sorted Array](../0034-find-first-and-last-position-of-element-in-sorted-array/README.md) — 找第一个/最后一个 target (lower_bound trick)
- 0146\. LRU Cache (待补) — 设计题, 双向链表 + 哈希
- 1146\. Snapshot Array (待补) — 版本化数组, 同思路二分
- 0715\. Range Module (待补) — 区间管理设计
