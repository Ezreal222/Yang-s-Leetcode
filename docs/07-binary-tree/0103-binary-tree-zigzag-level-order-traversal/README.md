# 0103. Binary Tree Zigzag Level Order Traversal / 二叉树的锯齿形层序遍历

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BFS, Queue · 二叉树, 广度优先搜索, 队列
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Return level-order traversal but alternate direction per level — row 0 left→right, row 1 right→left, row 2 left→right, …

**中文**: 层序遍历, 但相邻两层方向相反 — 第 0 层从左到右, 第 1 层从右到左, 交替.

## 思路

### Core idea

**[0102](../0102-binary-tree-level-order-traversal/README.md) 模板原封不动 + 奇数行 reverse**. BFS 永远按"左孩子先入队 → 右孩子后入队", 收完一行后看 `res.size()` 的奇偶决定要不要反转这行 vector. **不动 BFS 方向, 动行容器**.

### Key Insights

1. **奇偶用 `res.size() % 2` 判 / Parity via row index about to push**

    在"加这一行"之前, `res.size()` 就是这一行的 index. 0 → 不反转 (row 0 左→右), 1 → 反转 (row 1 右→左), 依此类推. 简单直接, 不需要单独维护 level 计数器.

2. **反转 vs 双向插入: 总工相同, 哪个更短 / Reverse vs deque, same total work**

    - **post-hoc reverse** (Yang 选这个): `std::reverse(vec.begin(), vec.end())`, O(width) per row.
    - **双端 deque 边走边插**: 偶数行 push_back, 奇数行 push_front, O(1) per push.

    两者总操作数都是 O(n). Reverse 版**代码更短**, 而且能直接复用 0102 的模板, 推荐. 别为了"更优雅"上 deque, 这题没有意义.

3. **千万别动 BFS 入队顺序 / Don't reverse the queue side**

    一个常见误解: "奇数行右先左后, 那 BFS 就反着入队呗". **错的** — BFS 入队顺序决定的是**下下层**怎么生成, 不是当前层方向. 把入队反了, 下下层会跟祖先关系错位.

    正确思路: BFS 始终按"标准方向"建图, 输出层时**只动输出行的容器顺序**.

4. **判奇偶的位置: 行收完之后 / Check parity after the row is gathered**

    Yang 把 reverse 放在"for 循环出来 + push_back 之前". 此时 `vec` 是完整一行, `res.size()` 还没含它. 这是最干净的位置 — 在 push 前判断、按需反转、然后 push.

### 一句话总结

**0102 BFS 不动, 收完一行根据 `res.size() % 2` 选择反不反转, 再 push.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
            vector<vector<int>> res;
            queue<TreeNode*> que;
            if (root) que.push(root);

            while (!que.empty()) {
                int size = que.size();
                vector<int> vec;
                for (int i = 0; i < size; i++) {
                    TreeNode* cur = que.front(); que.pop();
                    vec.push_back(cur->val);
                    if (cur->left)  que.push(cur->left);
                    if (cur->right) que.push(cur->right);
                }
                if (res.size() % 2 == 1) reverse(vec.begin(), vec.end());
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
        def zigzagLevelOrder(self, root: 'TreeNode | None') -> list[list[int]]:
            res = []
            if not root:
                return res
            q = deque([root])                          # collections.deque: BFS 标配, popleft/append 都是 O(1)
            while q:
                row = []
                for _ in range(len(q)):                # 固定本层 size, 当前 q 后面新加的孩子算下一层
                    cur = q.popleft()
                    row.append(cur.val)
                    if cur.left:  q.append(cur.left)
                    if cur.right: q.append(cur.right)
                if len(res) % 2:
                    row.reverse()                      # in-place 反转, list method, O(width)
                res.append(row)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number[][]}
     */
    var zigzagLevelOrder = function(root) {
        const res = [];
        if (!root) return res;
        const q = [root];                              // 数组当 queue: shift = O(n) 但本题 n 小, 没问题
        while (q.length) {
            const size = q.length;
            const row = [];
            for (let i = 0; i < size; i++) {
                const cur = q.shift();
                row.push(cur.val);
                if (cur.left)  q.push(cur.left);
                if (cur.right) q.push(cur.right);
            }
            if (res.length % 2) row.reverse();         // Array.prototype.reverse: in-place, O(width)
            res.push(row);
        }
        return res;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点入队/出队一次. Reverse 每行 O(width), 总和 O(n).
- **Space**: O(width) 队列峰值 + O(n) 输出.

## 易错点

- **不要反 BFS 入队方向**: 反入队后, 后续层会按反方向生成孩子, 整棵树的"哪些节点是兄弟"会错乱. 必须 BFS 标准方向 + 只反输出行.
- **`res.size()` 还是 `res.size()+1` 判奇偶**: 这一行还没 push 进 res, 所以 `res.size()` 就是它的 index. 别加 1.
- **空树**: `if (root) que.push(root)` 把空树挡在外面, while 不进入, 自然返回空 res.
- **双端 deque 解法是过度优化**: 总工一样, 代码长且容易写反方向. 面试除非问"能不能不用 reverse", 否则不必上.
- **题目要的是 `vector<vector<int>>`**: 别返回平铺的 vector. 每层一个内层 vector.

## 相关题目

- [0102. Binary Tree Level Order Traversal](../0102-binary-tree-level-order-traversal/README.md) — 本题骨架, 去掉 reverse 就是它
- [0107. Binary Tree Level Order Traversal II](../0107-binary-tree-level-order-traversal-ii/README.md) — 同款 BFS, 最后整体 reverse 整个 res
- [0199. Binary Tree Right Side View](../0199-binary-tree-right-side-view/README.md) — 每行只留最后一个
- [0515. Find Largest Value in Each Tree Row](../0515-find-largest-value-in-each-tree-row/README.md) — 每行取 max
- [0637. Average of Levels in Binary Tree](../0637-average-of-levels-in-binary-tree/README.md) — 每行算平均
- 1161. Maximum Level Sum (待补) — 每行求和取最大行号
