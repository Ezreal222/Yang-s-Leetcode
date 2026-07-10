# 0133. Clone Graph / 克隆图

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Graph, DFS, BFS, Hash Map · 图, 深度优先搜索, 广度优先搜索, 哈希表
    - **Link**: [LeetCode](https://leetcode.com/problems/clone-graph/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Deep-copy an undirected connected graph** → **DFS + hash map `original → clone`**: on visit, if seen, return the mapped clone; else create clone, **store mapping first**, then recursively clone neighbors.
>
> **中文**: **图的深拷贝** → **DFS + 哈希 `原节点 → 克隆节点`**: 见过就返映射; 没见过**先造 clone 存映射**, 再递归克隆邻居.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 给无向连通图的**一个节点** (每个 Node 有 `val` 和 `neighbors`). 返回**该图的深拷贝** (每节点都新建, 边关系保留).

**中文**: 深拷贝无向连通图.

## Key Insights

1. **🔑 核心挑战: **图里有环** → 无脑 DFS 会**无限递归** / Key challenge: cycles cause infinite recursion**

    树的深拷贝直接递归 — 因为没环. **图有环**: A → B → A → B → ..., 无终止.

    **解法**: 用 `visited` 记住"已克隆过的节点", 见到就**直接返回已建的 clone**, 不再递归下去.

    > **图 DFS 三件套**: 递归函数 + visited + base case (visited 命中就返). 树 DFS 没 visited, 图 DFS 必有.

2. **🔑 hash map `original → clone` 一石二鸟 / Hash map does two jobs**

    | 用途 | 语义 |
    |---|---|
    | **判是否 visited** | `visited.count(node)` |
    | **拿到已建的 clone** | `visited[node]` |

    一个 map 同时充当"visited 集" 和"节点映射表". 比"visited set + 单独 clone map" 更省.

    > 之前 §12 里 [0797](../0797-all-paths-from-source-to-target/README.md) 等只用 visited set (不需要"映射"). 本题需要**映射到新节点**, 升级成 map.

3. **🔑 关键顺序: **先建 clone + 存映射**, 再递归邻居 / Order matters: create → store → recurse**

    Yang 的核心 3 行:

    ```cpp
    Node* clone = new Node(node->val);      // 1. 建 clone (空 neighbors)
    visited[node] = clone;                  // 2. 立刻存映射 (关键!)
    for (Node* nb : node->neighbors) {
        clone->neighbors.push_back(dfs(nb, visited));    // 3. 递归克隆邻居
    }
    ```

    **顺序反了 (先递归再存映射) → 环里节点递归回来时 visited 里没自己 → 死递归**.

    例: A ↔ B (双向边).

    - 正确: dfs(A) → 建 A', 存 visited[A]=A' → 递归 dfs(B) → 建 B', 存 visited[B]=B' → 递归 dfs(A) → **命中 visited, 返 A'** → B' 的 neighbors = [A'] → 返回 → A' 的 neighbors = [B'] → 完成.
    - 错误 (递归后存映射): dfs(A) → 递归 dfs(B) → 递归 dfs(A) → visited 里没 A → 又建一个 A → 又递归 → 死循环.

    > **"先造壳, 再填内容"** 是打破环的通用技巧. **空 neighbors 的 clone 已足够作为"已 visited" 的证据**.

4. **🔑 邻居的 clone 用递归返回值填 / Fill neighbors via recursive return values**

    `clone->neighbors.push_back(dfs(nb, visited))` — 每个邻居的 clone 是递归结果. **不用手动查表** 因为递归本身就会返映射.

    > **递归"信任子问题"**: 不管 nb 是"新克隆" 还是"已建过", `dfs(nb)` 返回它的 clone. 干净.

5. **🔑 备选: BFS 版本 / Alternative: BFS**

    ```
    queue.push(node); visited[node] = new Node(node.val)
    while queue:
        cur = queue.pop()
        for nb in cur.neighbors:
            if nb not in visited:
                visited[nb] = new Node(nb.val)
                queue.push(nb)
            visited[cur].neighbors.push(visited[nb])
    ```

    - **BFS 好处**: 避免 stack overflow (超深图), 也没递归函数调用开销.
    - **DFS 好处**: 代码短 4 行, 递归自然.

    > 面试**能写 DFS 就够**, BFS 是"深图" 变体的加分.

6. **🔑 跟 [0138 Copy List with Random Pointer]** 是同款问题 / Same trick as 0138**

    - 0138: **链表** 每节点有 `next + random`.
    - **0133**: **图** 每节点有 `neighbors`.

    **本质一样**: 每个"原节点 → 新节点" 映射, 打破环.

    > **"图 / 复杂链表深拷贝"** 一族问题, 记住 `map<原, 新> + 先造壳后填` 模板.

7. **复杂度 O(V + E) 时间, O(V) 空间 / Linear**

    - 每节点访问 1 次 → O(V).
    - 每边**两遍** (无向图, 一次从每端), 但只算 O(E) 阶.
    - 空间: hash map 存 V 对 + 递归栈 O(V).

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        Node* cloneGraph(Node* node) {
            if (!node) return nullptr;
            unordered_map<Node*, Node*> visited;
            return dfs(node, visited);
        }
    private:
        Node* dfs(Node* node, unordered_map<Node*, Node*>& visited) {
            if (visited.count(node)) return visited[node];                   // 已建 → 直接返

            Node* clone = new Node(node->val);                               // 1. 建壳
            visited[node] = clone;                                           // 2. 立刻存映射 (关键)
            for (Node* nb : node->neighbors) {
                clone->neighbors.push_back(dfs(nb, visited));                // 3. 递归填 neighbors
            }
            return clone;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def cloneGraph(self, node: 'Node') -> 'Node':
            if not node: return None
            # dict 直接当 visited + 映射双用
            visited: dict['Node', 'Node'] = {}
            def dfs(n):
                if n in visited: return visited[n]      # 已建 → 直接返
                clone = Node(n.val)                     # 1. 建壳
                visited[n] = clone                      # 2. 立刻存映射
                # 用推导式一步填 neighbors, 比 for-append 简洁
                clone.neighbors = [dfs(nb) for nb in n.neighbors]
                return clone
            return dfs(node)
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {_Node} node
     * @return {_Node}
     */
    var cloneGraph = function(node) {
        if (!node) return null;
        // Map 支持任意 key 类型 (对象引用是可 key 的). 数字也自动 key
        const visited = new Map();
        const dfs = (n) => {
            if (visited.has(n)) return visited.get(n);
            const clone = new _Node(n.val);
            visited.set(n, clone);                       // 顺序: 先存映射 再递归
            // Array.map 一步生成 neighbors 数组
            clone.neighbors = n.neighbors.map(nb => dfs(nb));
            return clone;
        };
        return dfs(node);
    };
    ```

## Complexity

- **Time**: O(V + E) — 每节点访问 1 次, 每边处理常数次.
- **Space**: O(V) — hash map + 递归栈.

## 相关题目

- [0797. All Paths From Source to Target](../0797-all-paths-from-source-to-target/README.md) — DAG DFS 母题
- [0200. Number of Islands](../0200-number-of-islands/README.md) — DFS 洪水填充
- [0127. Word Ladder](../0127-word-ladder/README.md) — BFS 最短路径
- 0138\. Copy List with Random Pointer (待补) — 复杂链表深拷贝, 同款 map + 先造壳
- 0399\. Evaluate Division (待补) — 加权 DFS/UF
- 0323\. Number of Connected Components in an Undirected Graph (待补) — DFS / UF
- 1971\. Find if Path Exists in Graph — 见 §12 index
- 0785\. Is Graph Bipartite (待补) — 图 DFS + 染色
