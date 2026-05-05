# 0102. Binary Tree Level Order Traversal / 二叉树的层序遍历

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, Queue · 二叉树, 广度优先搜索, 队列
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-level-order-traversal/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Return values level-by-level (each level its own array).

**中文**: 按层返回每层的节点值, 每层一个数组.

## 思路

### BFS-by-level 模板

这是后面 9 道 BFS 类树题 (107 / 199 / 637 / 429 / 515 / 116 / 117 / 104 / 111) 的母模板. 关键是**外层 while + 内层 `for(size)`** —— 进入循环前先把当前队列大小**冻住**, 这一轮就只处理"那么多个", 它们就是一整层.

```text
queue.push(root) if root
while queue not empty:
    size = queue.size()         # 冻住本层节点数
    for i in 0..size-1:
        node = queue.pop()
        process(node)            # ← 各题的差异在这里
        push node.left / node.right (or all children)
    // 一层处理完, 收尾本层结果
```

### 一句话总结

**BFS, 队列做底, 外层 while + 内层 `for(size)` 锁住当前层 —— 模板抄出来, 后面 9 道全是改 `process` 那一行.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> levelOrder(TreeNode* root) {
            vector<vector<int>> res;
            queue<TreeNode*> que;
            if (root) que.push(root);
            while (!que.empty()) {
                int size = que.size();   // 冻住本层节点数
                vector<int> vec;
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    vec.push_back(cur->val);
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                res.push_back(vec);
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def levelOrder(self, root: 'TreeNode | None') -> list[list[int]]:
            # collections.deque: 双端队列, popleft / append 都是 O(1).
            # 用普通 list 模拟队列要 list.pop(0) 是 O(n), 大数据会爆.
            #   C++ 等价: queue<TreeNode*>
            res: list[list[int]] = []
            if not root:
                return res
            q: deque = deque([root])
            while q:
                level = []
                for _ in range(len(q)):    # 冻住本层节点数
                    cur = q.popleft()
                    level.append(cur.val)
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                res.append(level)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number[][]}
     */
    var levelOrder = function(root) {
        const res = [];
        if (!root) return res;
        // 数组模拟队列. arr.shift() 是 O(n), 这里题目规模 (≤2000 节点) 完全够用.
        // 严格 O(n) 总复杂度需要手动维护一个读指针 (head idx), 不然 shift 累加是 O(n²).
        const queue = [root];
        while (queue.length > 0) {
            const size = queue.length;   // 冻住本层节点数
            const level = [];
            for (let i = 0; i < size; i++) {
                const cur = queue.shift();
                level.push(cur.val);
                if (cur.left)  queue.push(cur.left);
                if (cur.right) queue.push(cur.right);
            }
            res.push(level);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点入队、出队各一次.
- **Space**: O(n) — 队列最坏存满最宽一层 (近满二叉树时 ~n/2).

## 相关题目

- [0107. Binary Tree Level Order Traversal II / 二叉树的层序遍历 II](../0107-binary-tree-level-order-traversal-ii/README.md) — 倒序版
- [0199. Binary Tree Right Side View / 二叉树的右视图](../0199-binary-tree-right-side-view/README.md) — 每层最后一个
- [0637. Average of Levels in Binary Tree / 二叉树的层平均值](../0637-average-of-levels-in-binary-tree/README.md) — 每层均值
- [0429. N-ary Tree Level Order Traversal / N 叉树的层序遍历](../0429-n-ary-tree-level-order-traversal/README.md) — N 叉树版
- [0515. Find Largest Value in Each Tree Row / 在每个树行中找最大值](../0515-find-largest-value-in-each-tree-row/README.md) — 每层最大
- [0116. Populating Next Right Pointers / 填充每个节点的下一个右侧节点指针](../0116-populating-next-right-pointers-in-each-node/README.md)
- [0117. Populating Next Right Pointers II / 填充每个节点的下一个右侧节点指针 II](../0117-populating-next-right-pointers-in-each-node-ii/README.md)
- [0104. Maximum Depth of Binary Tree / 二叉树的最大深度](../0104-maximum-depth-of-binary-tree/README.md)
- [0111. Minimum Depth of Binary Tree / 二叉树的最小深度](../0111-minimum-depth-of-binary-tree/README.md)
