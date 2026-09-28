# 0773. Sliding Puzzle / 滑动谜题

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: BFS, State-space Search, String, Hash Set · 广度优先, 状态空间搜索, 字符串, 哈希集合
    - **Link**: [LeetCode](https://leetcode.com/problems/sliding-puzzle/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Minimum moves on a puzzle = shortest path in the state graph** → BFS where each **node is a whole board**. Flatten the 2×3 board into a 6-char string, precompute which indices each position can swap with, and BFS level by level until you hit `"123450"`.
>
> **中文**: **最少步数 = 状态图上的最短路** → BFS, **每个节点是一整个棋盘**. 2×3 拍平成 6 字符串, 预先写好每个位置能和谁交换 (adj 表), 按层 BFS 直到 `"123450"`.
>
> *Template / 模版*: **State-space BFS** — 节点不是格子而是"局面". 同款: [0127 Word Ladder](../0127-word-ladder/README.md).

## Problem

**EN**: A 2×3 board holds tiles 1–5 and one empty slot (0). A move swaps 0 with a 4-directionally adjacent tile. Return the fewest moves to reach `[[1,2,3],[4,5,0]]`, or -1 if impossible.

**中文**: 2×3 棋盘, 0 可与上下左右相邻格交换. 求到目标局面的最少步数, 无解返 -1.

## Key Insights

1. **🔑 灵魂: 节点 = 整个局面, 不是格子 / A node is a whole board**

    网格 BFS (如 [0542](../0542-01-matrix/README.md)) 的节点是**格子**. 本题要找"最少步数", 每一步改变的是**整个棋盘**, 所以图的节点是**局面**, 边是**一次合法交换**.

    - 局面总数 ≤ 6! = **720**, 其中只有一半可达 (360). 状态空间极小, BFS 秒过.
    - 边权都 = 1 → **BFS 首次到达 = 最少步数**.

    > **"求最少操作次数 + 状态有限" → 状态空间 BFS**. 识别信号: 操作可逆、每步代价相同、状态能编码成 key.

2. **🔑 局面编码成 string / Encode the board as a string**

    ```cpp
    for (auto& row : board) for (int x : row) start += ('0' + x);
    ```

    - `vector<vector<int>>` 不能直接当 `unordered_set` 的 key (无默认 hash). **string 可以**.
    - 2×3 按行拍平: 位置 `(r, c)` → 下标 `r * 3 + c`.
    - 目标 `"123450"` 也成了普通字符串比较.

    > **状态编码是状态空间搜索的第一步**. 常用: string、整数 (位压缩 / 进制)、tuple.

3. **🔑 预计算 adj 表 — 拍平后的邻居 / Precomputed neighbor table**

    拍平后 2D 邻接变成 1D 下标关系. 2×3 太小, 直接手写:

    ```
    下标布局:   0 1 2
               3 4 5

    adj[0] = {1, 3}      adj[1] = {0, 2, 4}   adj[2] = {1, 5}
    adj[3] = {0, 4}      adj[4] = {1, 3, 5}   adj[5] = {2, 4}
    ```

    - 避免在 1D 下标上做 `±1 / ±3` 再判越界 — 下标 2 的 `+1` 是 3, 会**错跨行**.
    - 通用 m×n 也可以现场算: `idx → (idx / n, idx % n)`, 四方向判界再转回.

    > **易错点 top 1**: 拍平后直接用 `±1` 当左右邻居, 行末 `+1` 会跳到下一行开头.

4. **🔑 按层 BFS 计步 / Level-order BFS counts moves**

    ```cpp
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) { ... }    // 处理完整一层
        moves++;
    }
    ```

    - 每层 = 同一步数的所有局面.
    - Yang 在**生成 next 时就判 goal** 返 `moves + 1` — 比出队时判少扩一层, 小优化.
    - 开头 `if (start == goal) return 0` 处理 0 步的边界, 否则生成时判会漏掉.

5. **🔑 `seen.insert(next).second` — 判重 + 插入一步 / Insert-and-check idiom**

    `unordered_set::insert` 返 `pair<iterator, bool>`, `.second` = **是否真插入了** (之前不存在).

    ```cpp
    if (seen.insert(next).second) q.push(next);
    ```

    → 一次 hash 完成"没见过就标记并入队". 比 `if (!seen.count(x)) { seen.insert(x); ... }` 少一次查找.

    > **入队时标记, 不是出队时** — 否则同一局面可能被多次入队, 队列膨胀.

6. **🔑 无解判定: 队空仍没到 / Unreachable → queue drains**

    8-puzzle 类问题有**奇偶性不变量**: 一半局面永远到不了目标. BFS 不需要知道这个数学 — 队列耗尽还没碰到 goal 就返 -1.

    - 面试加分: 可以提"逆序对奇偶性可以 O(1) 预判无解", 但本题 BFS 本身已经很快, 不必.

7. **🔑 复杂度 / Complexity**

    - **状态数** S ≤ 6! = 720, 每个状态最多 3 个邻居, 每次生成/哈希 O(6).
    - **Time**: O(S · 6) ≈ O(1) for fixed 2×3; 一般化 m×n 为 O((mn)! · mn).
    - **Space**: O(S) — seen + queue.

8. **🔑 进阶: 双向 BFS / A\* / Bidirectional BFS or A\***

    - **双向 BFS**: 从 start 和 goal 同时扩, 相遇即停. 搜索量从 b^d 降到 2·b^(d/2). 大棋盘 (3×3 八数码、4×4 十五数码) 时有用.
    - **A\***: 启发函数用曼哈顿距离和. 15-puzzle 必用.

    本题 720 个状态, 普通 BFS 足够. 追问时提即可.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int slidingPuzzle(vector<vector<int>>& board) {
            // 拍平后每个下标能和谁交换 (2x3 手写, 避免跨行)
            const vector<vector<int>> adj = {
                {1, 3}, {0, 2, 4}, {1, 5}, {0, 4}, {1, 3, 5}, {2, 4}
            };
            string start;
            for (auto& row : board)
                for (int x : row) start += ('0' + x);
            const string goal = "123450";
            if (start == goal) return 0;

            queue<string> q;
            unordered_set<string> seen;
            q.push(start);
            seen.insert(start);

            int moves = 0;
            while (!q.empty()) {
                int sz = q.size();
                for (int i = 0; i < sz; i++) {                  // 一层 = 同一步数
                    string cur = q.front(); q.pop();
                    int zero = cur.find('0');
                    for (int nxt : adj[zero]) {
                        string next = cur;
                        swap(next[zero], next[nxt]);
                        if (next == goal) return moves + 1;     // 生成时即判
                        if (seen.insert(next).second) q.push(next);
                    }
                }
                moves++;
            }
            return -1;                                          // 不可达
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def slidingPuzzle(self, board: list[list[int]]) -> int:
            adj = [[1, 3], [0, 2, 4], [1, 5], [0, 4], [1, 3, 5], [2, 4]]

            # ''.join(生成器) 拍平: 比循环 += 更 Pythonic
            # str(x) for row in board for x in row — 嵌套推导式, 外层在前
            start = ''.join(str(x) for row in board for x in row)
            goal = '123450'
            if start == goal:
                return 0

            # deque 存 (局面, 步数) 二元组 — 不按层也能拿到步数
            # 相当于 C++ 按层 BFS 的 moves 计数, 写法更扁平
            q = deque([(start, 0)])
            seen = {start}                      # set 字面量, 相当于 unordered_set

            while q:
                cur, moves = q.popleft()
                zero = cur.index('0')
                for nxt in adj[zero]:
                    # Python str 不可变, 不能 swap 下标 → 转 list 改完再 join
                    s = list(cur)
                    s[zero], s[nxt] = s[nxt], s[zero]   # 元组解包交换
                    nxt_state = ''.join(s)
                    if nxt_state == goal:
                        return moves + 1
                    if nxt_state not in seen:
                        seen.add(nxt_state)
                        q.append((nxt_state, moves + 1))
            return -1
    ```

=== "JavaScript"
    ```javascript
    var slidingPuzzle = function(board) {
        const adj = [[1, 3], [0, 2, 4], [1, 5], [0, 4], [1, 3, 5], [2, 4]];

        // flat() 把二维数组摊平一层, join('') 拼成字符串
        // 相当于 C++ 的双层 for + start += ('0' + x)
        const start = board.flat().join('');
        const goal = '123450';
        if (start === goal) return 0;

        // 用下标游标 head 代替 shift(): shift 是 O(n), 游标是 O(1)
        const q = [start];
        const seen = new Set([start]);
        let head = 0, moves = 0;

        while (head < q.length) {
            const levelEnd = q.length;              // 当前层的右边界
            for (; head < levelEnd; head++) {
                const cur = q[head];
                const zero = cur.indexOf('0');
                for (const nxt of adj[zero]) {
                    // JS 字符串也不可变 → 展开成数组, 解构交换, 再 join
                    const arr = [...cur];
                    [arr[zero], arr[nxt]] = [arr[nxt], arr[zero]];
                    const next = arr.join('');
                    if (next === goal) return moves + 1;
                    if (!seen.has(next)) {
                        seen.add(next);
                        q.push(next);
                    }
                }
            }
            moves++;
        }
        return -1;
    };
    ```

## Complexity

- **Time**: O(S · L), S = 可达局面数 (≤ 720), L = 6 (字符串长度). 固定尺寸下是常数.
- **Space**: O(S · L).

## 相关题目

- [0127. Word Ladder](../0127-word-ladder/README.md) — 状态空间 BFS 母题, 节点 = 单词
- [0542. 01 Matrix](../0542-01-matrix/README.md) — 网格 BFS (节点 = 格子), 对比本题
- [0994. Rotting Oranges](../0994-rotting-oranges/README.md) — 多源网格 BFS
- 0752\. Open the Lock (待补) — 状态空间 BFS, 4 位转盘 + deadends, 同款最直接
- 0433\. Minimum Genetic Mutation (待补) — 0127 的小号版本
- 0854\. K-Similar Strings (待补) — 字符串交换状态 BFS
- 1091\. Shortest Path in Binary Matrix (待补) — 8 方向网格 BFS
