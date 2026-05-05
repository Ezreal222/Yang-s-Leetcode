# 0094. Binary Tree Inorder Traversal / 二叉树的中序遍历

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Stack · 二叉树, 深度优先搜索, 栈
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-inorder-traversal/)
    - **Status**: ✅ Solved
    - **First solved**: 2026-05-04
    - **Reviewed**: ☐ ☐ ☐

## Problem / 题意

**EN**: Return the **inorder** traversal (left → root → right) of a binary tree.

**中文**: 返回二叉树的**中序**遍历 (左 → 根 → 右).

## Approach / 思路

### 核心思想 / Core idea

[0144](../0144-binary-tree-preorder-traversal/README.md) 同款, 把 "访问 root" 那行**挪到中间** —— 先递归左, 再 push 当前, 再递归右.

### 关键洞察 / Key insights

**迭代版独立一档**: 不像 pre/post 可以从 preorder 直接改, inorder 的迭代写法**自成一派** —— 必须先一路压入左子树到尽头, 再 pop 一个 visit, 然后转向 right 子树重复.

```
while node or stack:
    while node:                      # 一路向左
        stack.push(node)
        node = node.left
    node = stack.pop()
    visit(node)                      # 中间访问
    node = node.right
```

这是 inorder 在 BST 里产出**升序序列**的根源 —— 等到遇见 BST 的题 (0098 / 0230 / 0501) 这套迭代结构会反复出现.

### 一句话总结 / One-liner

**Inorder = visit 在中间. BST 中序 ⇒ 升序; 迭代写法和 pre/post 不同, 是"压左到底, pop, 转右"的循环.**

## Solution / 题解

### Variant A — recursion / 递归

=== "C++"
    ```cpp
    class Solution {
    public:
        void traversal(TreeNode* cur, vector<int>& vec) {
            if (cur == nullptr) return;
            traversal(cur->left, vec);
            vec.push_back(cur->val);   // root 在中间
            traversal(cur->right, vec);
        }
        vector<int> inorderTraversal(TreeNode* root) {
            vector<int> res;
            traversal(root, res);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def inorderTraversal(self, root: 'TreeNode | None') -> list[int]:
            res: list[int] = []
            def dfs(node):
                if not node:
                    return
                dfs(node.left)
                res.append(node.val)   # root 中间
                dfs(node.right)
            dfs(root)
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number[]}
     */
    var inorderTraversal = function(root) {
        const res = [];
        const dfs = (node) => {
            if (!node) return;
            dfs(node.left);
            res.push(node.val);   // root 中间
            dfs(node.right);
        };
        dfs(root);
        return res;
    };
    ```

### Variant B — iterative (press-left, pop, go-right) / 迭代

跟 pre/post 的"push children"模板不一样, inorder 迭代要用**两层循环**: 外层游标 `cur` 控制方向, 内层循环把 cur 一路向左压栈到底; 然后 pop 一个 visit, 转 `cur->right` 重复. 直到 `cur` 和栈都空.

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<int> inorderTraversal(TreeNode* root) {
            vector<int> res;
            stack<TreeNode*> stk;
            TreeNode* cur = root;
            while (cur || !stk.empty()) {
                while (cur) {
                    stk.push(cur);
                    cur = cur->left;            // left: 一路压到最左
                }
                cur = stk.top(); stk.pop();
                res.push_back(cur->val);        // mid: 弹出来的那个就是当前最左未访问
                cur = cur->right;               // right: 转向右子树, 重新进入外层 while
            }
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def inorderTraversal(self, root: 'TreeNode | None') -> list[int]:
            res: list[int] = []
            stack: list = []
            cur = root
            # 外层: cur 还能走 OR 栈里还有未访问的祖先, 就继续
            while cur or stack:
                while cur:
                    stack.append(cur)
                    cur = cur.left          # left
                cur = stack.pop()
                res.append(cur.val)         # mid
                cur = cur.right             # right
            return res
    ```

=== "JavaScript"
    ```javascript
    var inorderTraversal = function(root) {
        const res = [];
        const stack = [];
        let cur = root;
        while (cur || stack.length > 0) {
            while (cur) {
                stack.push(cur);
                cur = cur.left;             // left
            }
            cur = stack.pop();
            res.push(cur.val);              // mid
            cur = cur.right;                // right
        }
        return res;
    };
    ```

## Complexity / 复杂度

- **Time**: O(n).
- **Space**: O(h) recursion (h = height) + O(n) output.

## Pitfalls / 易错点

- C++ helper 必须 `vector<int>&` —— 同 0144.
- 写 BST 题时常常不需要先把 inorder 拿出来再处理, 直接在 inorder 的 "中间访问" 那一步实时判断更省空间.

## Related / 相关题目

- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 完整讨论
- [0145. Binary Tree Postorder Traversal](../0145-binary-tree-postorder-traversal/README.md) — 三兄弟之老幺
- 0098. Validate Binary Search Tree (待补) — inorder 升序判 BST
- 0230. Kth Smallest Element in a BST (待补) — inorder 第 k 个
- 0501. Find Mode in BST (待补) — inorder 上做众数计数
