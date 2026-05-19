# 0257. Binary Tree Paths / 二叉树的所有路径

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Tree, DFS, Backtracking, String · 二叉树, 深度优先搜索, 回溯, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/binary-tree-paths/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Return all root-to-leaf paths formatted as `"a->b->c"`.

**中文**: 返回所有从根到叶子的路径, 格式化成 `"a->b->c"`.

## 思路

### 一句话总结

**前序 DFS 把当前节点拼到 `path`, 到叶子就把 `path` 收进 `res`. `path` 用**值传递** —— 每次递归拿到的是拷贝, 函数返回后调用者的 `path` 自动"回退"到原状态, 不需要手动 pop —— 这是隐式回溯.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        // path 用值传递: 每次递归是一份拷贝, 返回后调用者那份不受影响 → 隐式回溯
        void traversal(TreeNode* cur, string path, vector<string>& res) {
            path += to_string(cur->val);
            // 到叶子, 把这条路径收下
            if (!cur->left && !cur->right) {
                res.push_back(path);
                return;
            }
            if (cur->left)  traversal(cur->left,  path + "->", res);
            if (cur->right) traversal(cur->right, path + "->", res);
        }
        vector<string> binaryTreePaths(TreeNode* root) {
            vector<string> res;
            traversal(root, "", res);
            return res;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def binaryTreePaths(self, root: 'TreeNode | None') -> list[str]:
            res: list[str] = []
            def dfs(node, path: str):
                # Python 字符串是不可变 (immutable) —— path + str 永远生成新对象,
                # 所以这里跟 C++ 的"值传递 string"一样, 天然不会污染上层调用.
                #   C++ 等价: void traversal(TreeNode* cur, string path, ...)
                path += str(node.val)
                if not node.left and not node.right:
                    res.append(path)
                    return
                if node.left:
                    dfs(node.left, path + '->')
                if node.right:
                    dfs(node.right, path + '->')
            if root:
                dfs(root, '')
            return res
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {string[]}
     */
    var binaryTreePaths = function(root) {
        const res = [];
        // 跟 Python 一样, JS 字符串也是 immutable, 拼接生成新串, 不需要手动撤销.
        const dfs = (node, path) => {
            path += node.val;
            if (!node.left && !node.right) {
                res.push(path);
                return;
            }
            if (node.left)  dfs(node.left,  path + '->');
            if (node.right) dfs(node.right, path + '->');
        };
        if (root) dfs(root, '');
        return res;
    };
    ```

## Complexity

- **Time**: O(n · h) — 每条路径长度最多 h, 最多 n/2 条路径 (满二叉树叶子数). 字符串拼接每次 O(h).
- **Space**: O(n · h) for the output strings + O(h) recursion stack.

## 易错点

- **值传递 vs 引用传递**: 这题最优雅的写法是 `string path` (值传递), 利用拷贝实现自动回退. 如果非要 `vector<int>& path` (引用传递), 就要在递归后**手动 `path.pop_back()`** 才能回退 —— 这是经典的"显式回溯"模式, 后面回溯系列 (子集/排列/组合) 会反复用到.
- **递归条件检查在调用前**: `if (cur->left) traversal(...)` —— 别先递归再判 nullptr, 否则空树进 traversal 会崩 (`cur->val` 解引用空指针). 入口 `binaryTreePaths` 也要在 `traversal` 前判 root.
- **叶子判定**: `!cur->left && !cur->right` —— 必须**两边都空**才是叶子. 单边为空 (e.g. 只有右孩子) 还得继续递归右边, 不能当叶子.
- **`->` 分隔符的位置**: 把 `"->"` 放在**递归调用时拼**, 不要放在函数开头拼. 否则根节点前会多一个 `->`.
- **字符串拼接性能**: 大量 `path += s` 在 C++/Python/JS 里都是 O(n²) 总开销 (每次拷贝). 这题 n 小不影响, 大数据要换 `vector<string>` + 最后 `join` 或者 `StringBuilder`.

## 相关题目

- [0144. Binary Tree Preorder Traversal](../0144-binary-tree-preorder-traversal/README.md) — 同款前序骨架
- [0104. Maximum Depth of Binary Tree](../0104-maximum-depth-of-binary-tree/README.md) — 同款 root-to-leaf 递归
- [0112. Path Sum / 路径总和](../0112-path-sum/README.md) — 把"收集字符串"换成"判和是否等于目标"
- [0113. Path Sum II](../0113-path-sum-ii/README.md) — 收集所有"和等于目标"的路径, 这里就要用显式回溯了
- [0129. Sum Root to Leaf Numbers](../0129-sum-root-to-leaf-numbers/README.md) — 把路径当数字累加, 字符串 → int 加速
- 回溯系列 (子集 0078 / 排列 0046 / 组合 0077) 待补 —— 显式回溯的经典训练场
