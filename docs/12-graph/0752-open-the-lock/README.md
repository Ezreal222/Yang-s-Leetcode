# 0752. Open the Lock / 打开转盘锁

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: BFS, State-space Search, String, Hash Set · 广度优先, 状态空间搜索, 字符串, 哈希集合
    - **Link**: [LeetCode](https://leetcode.com/problems/open-the-lock/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Min turns = shortest path in a state graph** of 10⁴ lock codes. Each code has 8 neighbors (4 wheels × ±1, wrapping 9↔0). Deadends are **blocked nodes**. BFS level by level from `"0000"`; first time you reach `target` is the answer.
>
> **中文**: **最少旋转次数 = 状态图最短路**, 共 10⁴ 个密码. 每个密码 8 个邻居 (4 个拨轮 × ±1, 9↔0 循环). deadends 是**不能进的节点**. 从 `"0000"` 按层 BFS, 首次碰到 target 即答案.
>
> *Template / 模版*: **State-space BFS + blocked set** — [0773](../0773-sliding-puzzle/README.md) 的同款, 节点换成 4 位密码.

## Problem

**EN**: A lock has 4 circular wheels (digits 0–9). One move turns a single wheel by one slot either way. Start at `"0000"`; reaching any code in `deadends` locks you out. Return the fewest moves to reach `target`, or -1.

**中文**: 4 位转盘锁, 每步转一个拨轮一格 (可正可反, 循环). 起点 `"0000"`, 碰到 deadend 即卡死. 求到 target 最少步数, 不可达返 -1.

## Key Insights

1. **🔑 灵魂: 10⁴ 个密码 = 10⁴ 个节点, 每点 8 条边 / 10⁴ nodes × 8 edges**

    - 状态空间 = `"0000"` ~ `"9999"`, 共 **10,000** 个.
    - 每个密码的邻居: 4 个位置 × {+1, −1} = **8 个**.
    - 边权都 = 1 → **BFS 首次到达 = 最少步数**.

    总工作量 ≈ 10⁴ × 8 × 4 (字符串长度) ≈ 3×10⁵, 秒过.

    > **"最少操作 + 状态可枚举" → 状态空间 BFS**. 跟 [0773](../0773-sliding-puzzle/README.md) 一个模子, 只是邻居生成规则不同.

2. **🔑 deadends = 不可进入的节点 / Deadends are blocked nodes**

    - 把 deadends 放进 `unordered_set`, 生成邻居时**跳过**.
    - **起点本身在 deadends 里** → 直接 -1 (Yang 开头已判). 这是最容易漏的边界.
    - 实现技巧: 也可以把 deadends 直接**预塞进 seen** — 效果一样, 少一个 set 查询.

    > **易错点 top 1**: 忘判 `"0000"` 本身是 deadend. 若不判, BFS 会从一个"锁死"的状态出发, 答案错.

3. **🔑 循环进位: `(c - '0' + d + 10) % 10` / Wrap-around digit**

    ```cpp
    cur[j] = '0' + ((c - '0' + d + 10) % 10);
    ```

    - `d = +1`: 9 → 0.
    - `d = -1`: 0 → 9. **`+10` 防止负数取模** — C++ 里 `-1 % 10 = -1`, 不是 9.
    - Python 的 `%` 对负数返非负 (`-1 % 10 = 9`), 所以 Python 不需要 `+10`, 但写上也无害.

    > **易错点 top 2**: C++/JS 负数取模是负的. 循环下标一律写 `(x + d + M) % M`.

4. **🔑 原地修改 + 还原 — 省拷贝 / Mutate in place, then restore**

    Yang 的写法不为每个邻居拷贝一个新 string:

    ```cpp
    char c = cur[j];
    for (int d : {1, -1}) {
        cur[j] = ...;           // 改一位, cur 现在就是邻居
        ...                     // 判断 / 入队 (入队时 q.push 会拷贝)
    }
    cur[j] = c;                 // 这一位还原, 下一轮改别的位
    ```

    - 跟回溯的"做选择 → 撤销选择"同款思路.
    - `for (int d : {1, -1})` — 用 initializer_list 遍历两个方向, 比写两遍代码干净.

    > 对比 [0773](../0773-sliding-puzzle/README.md) 每个邻居都 `string next = cur` 拷一份 — 那题每步只换一对, 两种写法都行. 本题 8 个邻居, 原地改更省.

5. **🔑 入队时判 target + 标记 seen / Check target at generation time**

    ```cpp
    if (!dead.count(cur) && seen.insert(cur).second) {
        if (cur == target) return turns + 1;
        q.push(cur);
    }
    ```

    - `seen.insert(cur).second` — 没见过就插入并返 true, 一次 hash 完成 (见 [0773](../0773-sliding-puzzle/README.md) Insight 5).
    - 生成时判 target → 返 `turns + 1`, 少扩一层.
    - 开头 `target == "0000"` 返 0, 补上 0 步的边界.

6. **🔑 进阶: 双向 BFS / Bidirectional BFS**

    从 `"0000"` 和 `target` 两头同时扩, 每次扩**较小**的那一侧, 两个 frontier 相交即返. 分支因子 8, 深度 d → 搜索量从 8^d 降到约 2·8^(d/2).

    - 本题 10⁴ 状态, 单向已够快.
    - 面试追问"能更快吗" → 提双向 BFS, 讲清"每次扩小的那头" 这个细节即可.

7. **🔑 复杂度 / Complexity**

    - **Time**: O(N · A · L), N = 10⁴ 个状态, A = 8 个邻居, L = 4 (字符串操作/哈希). 实际 ≈ 3×10⁵.
    - **Space**: O(N · L) — seen + queue + deadends.

8. **🔑 状态空间 BFS 的通用检查清单 / Checklist for state-space BFS**

    | 步骤 | 本题 | 0773 |
    |---|---|---|
    | 状态编码 | 4 位 string | 6 位 string |
    | 邻居生成 | 4 位 × ±1 循环 | 0 与 adj[zero] 交换 |
    | 禁止状态 | deadends | 无 |
    | 起点边界 | start ∈ dead → -1; start == target → 0 | start == goal → 0 |
    | 判重 | `seen.insert(x).second` | 同 |

    > 背这张表, 同类题 (0127 / 0433 / 0854) 都是填空.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int openLock(vector<string>& deadends, string target) {
            unordered_set<string> dead(deadends.begin(), deadends.end());
            string start = "0000";
            if (dead.count(start)) return -1;                      // 起点就锁死
            if (target == start) return 0;

            unordered_set<string> seen{start};
            queue<string> q;
            q.push(start);
            int turns = 0;

            while (!q.empty()) {
                int sz = q.size();
                for (int i = 0; i < sz; i++) {
                    string cur = q.front(); q.pop();
                    for (int j = 0; j < 4; j++) {
                        char c = cur[j];
                        for (int d : {1, -1}) {
                            cur[j] = '0' + ((c - '0' + d + 10) % 10);  // +10 防负数取模
                            if (!dead.count(cur) && seen.insert(cur).second) {
                                if (cur == target) return turns + 1;
                                q.push(cur);
                            }
                        }
                        cur[j] = c;                                 // 还原这一位
                    }
                }
                ++turns;
            }
            return -1;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def openLock(self, deadends: list[str], target: str) -> int:
            # 把 deadends 直接当 seen 的初始内容: 死锁状态 = "已访问过, 别进"
            # 省掉一个单独的 dead 集合, 少一次查询
            seen = set(deadends)
            if '0000' in seen:
                return -1
            if target == '0000':
                return 0

            seen.add('0000')
            q = deque(['0000'])
            turns = 0

            while q:
                turns += 1
                # range(len(q)) 在循环开始时就定格了层大小, 中途 append 不影响
                for _ in range(len(q)):
                    cur = q.popleft()
                    for j in range(4):
                        digit = int(cur[j])
                        for d in (1, -1):
                            # Python 的 % 对负数返非负: (0 - 1) % 10 == 9, 不用 +10
                            # 切片拼接生成新串: str 不可变, 没法原地改
                            nxt = cur[:j] + str((digit + d) % 10) + cur[j + 1:]
                            if nxt == target:
                                return turns
                            if nxt not in seen:
                                seen.add(nxt)
                                q.append(nxt)
            return -1
    ```

=== "JavaScript"
    ```javascript
    var openLock = function(deadends, target) {
        // new Set(array) 一次建集合; deadends 预塞进 seen, 同 Python 版思路
        const seen = new Set(deadends);
        if (seen.has('0000')) return -1;
        if (target === '0000') return 0;

        seen.add('0000');
        const q = ['0000'];
        let head = 0, turns = 0;                   // 游标代替 shift(), O(1) 出队

        while (head < q.length) {
            turns++;
            const levelEnd = q.length;
            for (; head < levelEnd; head++) {
                const cur = q[head];
                for (let j = 0; j < 4; j++) {
                    const digit = cur.charCodeAt(j) - 48;   // '0' 的 char code 是 48
                    for (const d of [1, -1]) {
                        // JS 的 % 对负数返负 (-1 % 10 === -1), 必须 +10
                        const nd = (digit + d + 10) % 10;
                        // slice 拼接生成新串 (JS 字符串不可变)
                        const nxt = cur.slice(0, j) + nd + cur.slice(j + 1);
                        if (nxt === target) return turns;
                        if (!seen.has(nxt)) {
                            seen.add(nxt);
                            q.push(nxt);
                        }
                    }
                }
            }
        }
        return -1;
    };
    ```

## Complexity

- **Time**: O(N · A · L) — N = 10⁴ 状态, A = 8 邻居, L = 4.
- **Space**: O(N · L).

## 相关题目

- [0773. Sliding Puzzle](../0773-sliding-puzzle/README.md) — 同款状态空间 BFS, 局面 = 棋盘
- [0127. Word Ladder](../0127-word-ladder/README.md) — 状态空间 BFS, 节点 = 单词, 邻居 = 改一个字母
- [0542. 01 Matrix](../0542-01-matrix/README.md) — 网格 BFS, 对比"格子节点" vs "状态节点"
- 0433\. Minimum Genetic Mutation (待补) — 0127 小号版, 8 位基因串
- 0854\. K-Similar Strings (待补) — 字符串交换状态 BFS
- 1197\. Minimum Knight Moves (待补) — 无限棋盘 BFS + 对称剪枝
