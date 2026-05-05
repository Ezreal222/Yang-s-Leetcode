# 0559. Maximum Depth of N-ary Tree / N 叉树的最大深度

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, N-ary Tree, BFS, DFS, Recursion · N 叉树, 广度优先搜索, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/maximum-depth-of-n-ary-tree/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Given an N-ary tree (each node has a `children` list), return the maximum depth (number of nodes on the longest root-to-leaf path).

**中文**: N 叉树的最大深度 (从根到最远叶子的节点数).

## 思路

### 一句话总结

**[0104 最大深度](../0104-maximum-depth-of-binary-tree/README.md) 的 N 叉版: 递归取 `1 + max(每个孩子的深度)`; BFS 数完层数. 唯一差异是把 `left/right` 换成"遍历整个 `children` 列表".**

## Solution

### Variant A — recursion / 递归

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxDepth(Node* root) {
            if (!root) return 0;
            int depth = 0;
            // 没有孩子就保持 depth = 0; 有孩子就取最深那个
            for (int i = 0; i < (int)root->children.size(); i++) {
                depth = max(depth, maxDepth(root->children[i]));
            }
            return depth + 1;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def maxDepth(self, root: 'Node | None') -> int:
            if not root:
                return 0
            # max(iterable, default=0): 关键是 default —— 当 children 是空列表时,
            # max([]) 会抛 ValueError. default 给了一个"空集兜底值".
            # 这里叶子节点 children 为空, depth 应该是 0, 加上自己的 1 就是 1, 正确.
            #   C++ 等价: int depth = 0; for (...) depth = max(depth, ...); return depth + 1;
            return 1 + max(
                (self.maxDepth(c) for c in root.children),
                default=0,
            )
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {Node} root
     * @return {number}
     */
    var maxDepth = function(root) {
        if (!root) return 0;
        // arr.reduce(callback, initial): 把数组折叠成一个值. 这里用 0 作初始,
        // 累加器 d 始终保持"已经看过的孩子里最深那个". 空 children → 直接返回 0.
        // 比 Math.max(...arr.map(...)) 干净 (后者在空数组上会得到 -Infinity).
        //   C++ 等价: int d = 0; for (c : children) d = max(d, depth(c));
        return 1 + root.children.reduce(
            (d, c) => Math.max(d, maxDepth(c)),
            0
        );
    };
    ```

### Variant B — BFS counting levels

=== "C++"
    ```cpp
    class Solution {
    public:
        int maxDepth(Node* root) {
            queue<Node*> que;
            if (root) que.push(root);
            int max_depth = 0;
            while (!que.empty()) {
                max_depth++;
                int size = que.size();
                for (int i = 0; i < size; i++) {
                    Node* cur = que.front(); que.pop();
                    for (Node* child : cur->children) {
                        if (child) que.push(child);
                    }
                }
            }
            return max_depth;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def maxDepth(self, root: 'Node | None') -> int:
            if not root:
                return 0
            q: deque = deque([root])
            depth = 0
            while q:
                depth += 1
                for _ in range(len(q)):
                    cur = q.popleft()
                    q.extend(cur.children)   # 一次性 append 整个 children 列表
            return depth
    ```

=== "JavaScript"
    ```javascript
    var maxDepth = function(root) {
        if (!root) return 0;
        const queue = [root];
        let depth = 0;
        while (queue.length > 0) {
            depth++;
            const size = queue.length;
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                queue.push(...cur.children);   // spread 一次性入队所有 children
            }
        }
        return depth;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: 递归 O(h); BFS O(n).

## 相关题目

- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 二叉版, 思路一模一样
- [0429. N-ary Tree Level Order Traversal](../0429-n-ary-tree-level-order-traversal/README.md) — 同款 N 叉 BFS 模板
- [0111. Minimum Depth](../0111-minimum-depth-of-binary-tree/README.md) — 对称的最小深度 (注意 null 子树坑)
