# 0450. Delete Node in a BST / 删除二叉搜索树中的节点

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, BST, DFS, Recursion · 二叉树, 二叉搜索树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/delete-node-in-a-bst/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Delete the node with value `key` from a BST and return the new root (any valid BST shape OK, no rebalancing needed).

**中文**: 在 BST 里删除值为 `key` 的节点, 返回新的根 (任何合法 BST 形状都行, 不要求平衡).

## 思路

### Core idea

BST 操作三件套 ([0700](../0700-search-in-a-binary-search-tree/README.md) 搜 / [0701](../0701-insert-into-a-binary-search-tree/README.md) 插 / 本题删) —— **单向走找到 key**, 然后处理"如何把它从 BST 里拿掉, 还保持 BST 性质". 复用 [0701](../0701-insert-into-a-binary-search-tree/README.md) 同款 `child = recurse(child, key)` 模式自然把树缝合回去.

### Key Insights

**三种删除情形 / Three deletion cases**:

| 情形 | 处理 |
|---|---|
| **Case 1**: 没有左孩子 | 用右孩子顶替自己 (右可能也是 null, 那就返回 null, 等于把这条线砍掉) |
| **Case 2**: 没有右孩子 | 用左孩子顶替自己 |
| **Case 3**: 左右都有 | 找**中序后继** (右子树的最左节点), 把它的值搬过来覆盖当前, 然后**递归删掉后继**那个真实节点 |

**Yang 的微观察**: case 1 隐含了"左右都空"—— 当 left 是 null 时 case 1 触发, return root.right; 如果 right 也是 null, 返回的就是 null, 整条线断掉. 所以不用单独写"左右都空 → null"的 case.

为啥用**中序后继**? 它是右子树里**最小**的值, 替换上来后左子树所有值仍 ≤ 新根 ≤ 右子树所有剩余值, BST 性质继续成立. 用**中序前驱**(左子树最大值)也对, 对称.

### 一句话总结

**找 key, 三种情形: 缺一孩子直接顶, 双孩子找右子树最左节点 (中序后继), 值替换再删后继. 全程靠 `root->left/right = recurse(...)` 把树缝合回去.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* deleteNode(TreeNode* root, int key) {
            if (!root) return nullptr;
            if (key < root->val) {
                root->left  = deleteNode(root->left,  key);   // 走左找
            } else if (key > root->val) {
                root->right = deleteNode(root->right, key);   // 走右找
            } else {
                // 找到了, 三种情形
                if (!root->left)  return root->right;         // case 1 (兼顾"左右都空")
                if (!root->right) return root->left;          // case 2
                // case 3: 找中序后继 (右子树最左)
                TreeNode* cur = root->right;
                while (cur->left) cur = cur->left;
                root->val = cur->val;                          // 值搬上来
                root->right = deleteNode(root->right, cur->val); // 递归删后继
            }
            return root;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def deleteNode(self, root: 'TreeNode | None', key: int) -> 'TreeNode | None':
            if not root:
                return None
            if key < root.val:
                root.left  = self.deleteNode(root.left,  key)
            elif key > root.val:
                root.right = self.deleteNode(root.right, key)
            else:
                if not root.left:  return root.right
                if not root.right: return root.left
                # case 3: in-order successor = right subtree's leftmost
                cur = root.right
                while cur.left:
                    cur = cur.left
                root.val = cur.val
                root.right = self.deleteNode(root.right, cur.val)
            return root
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number} key
     * @return {TreeNode}
     */
    var deleteNode = function(root, key) {
        if (!root) return null;
        if (key < root.val) {
            root.left  = deleteNode(root.left,  key);
        } else if (key > root.val) {
            root.right = deleteNode(root.right, key);
        } else {
            if (!root.left)  return root.right;
            if (!root.right) return root.left;
            let cur = root.right;
            while (cur.left) cur = cur.left;
            root.val = cur.val;
            root.right = deleteNode(root.right, cur.val);
        }
        return root;
    };
    ```

## Complexity

- **Time**: O(h) — 一次单向搜 (O(h)) + case 3 的"找后继 + 递归再删一次" (再 O(h)). 总 O(h).
- **Space**: O(h) recursion.

## 易错点

- **Case 1 隐含覆盖"左右都空"**: 你的观察对的. 一定要把"无左 → return right" 这个 case 写在前面 —— 当 left 是 null 时, return root->right 不管是不是 null 都对. 如果你单独又写 `if (!root->left && !root->right) return nullptr;` 是多余的, 但不影响正确性.
- **Case 3 别忘了第二次递归**: 找到后继 `cur` 后, 不能直接 "断开 cur" —— `cur` 可能本身还有右孩子 (左肯定没有, 它是 leftmost). 正确做法: **把 cur 的值搬到 root, 然后在 `root->right` 子树里递归删 `cur.val`**. 那次递归会从右子树根走到 cur, 触发 case 1 (cur 没左孩子), 用 cur 的右孩子顶替 cur, 树自然缝合.
- **不能直接交换 root 和 cur 然后 free**: 这种"拷贝值"的做法只改了值不挪指针, 简单但要求节点值是"值类型"可拷贝. 如果节点带的是大对象 (Java/C++ 节点持有不可拷贝资源), 真实工程要"指针重排"删法 —— 复杂得多, 但 LeetCode 这种 int val 的版本拷贝值最干净.
- **中序前驱也行**: 左子树的最右节点 (右最大) 也是合法选择, 与中序后继对称. 选哪个看个人风格.
- **`child = recurse(child, key)` 模式**: 跟 0701 同款. 每层调用返回"修改后的子树根", 上层赋值回 `root->left/right`. 这是树修改类问题的通用骨架.

## 相关题目

- [0701. Insert into a BST](../0701-insert-into-a-binary-search-tree/README.md) — 同款 child-assignment 递归, 但找空位插入 (比 delete 简单)
- [0700. Search in a Binary Search Tree](../0700-search-in-a-binary-search-tree/README.md) — BST 单向走入门
- [0235. Lowest Common Ancestor of a BST](../0235-lowest-common-ancestor-of-a-binary-search-tree/README.md) — 同款单向走
- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — BST 性质入门
- [0669. Trim a Binary Search Tree / 修剪二叉搜索树](../0669-trim-a-binary-search-tree/README.md) — 同款 child-assignment 递归, 但目标是删一整段超界节点
