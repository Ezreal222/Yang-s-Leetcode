# 1110. Delete Nodes And Return Forest / 删点成林

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Hash Set, Recursion · 二叉树, 深度优先搜索, 哈希表, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/delete-nodes-and-return-forest/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: Given a binary tree and a list `to_delete` of values, delete every node with a value in `to_delete`. Return the **forest** of remaining subtree roots (any order).

**中文**: 给一棵二叉树和待删值列表 `to_delete`, 删掉所有命中的节点, 返回剩余子树的根**列表 (森林)**.

## 思路

### Core idea

[0669](../0669-trim-a-binary-search-tree/README.md) / [0814](../0814-binary-tree-pruning/README.md) 树删除通用模板的**进阶版**: 这次删除时不只是返回 null, 还要**把被孤立出来的子树根收集到结果里**. 自然是后序: 先递归两边, 再处理当前.

**孤立判定**: 一个节点变成新的森林根, 当且仅当它**自己没被删** AND **它的父亲被删了 (或它本来就是树根)**. 把"父亲是否被删"通过递归参数传下来.

### Key Insights

1. **`isRoot` 不是"原树的根", 是"现在没父亲"/ `isRoot` means "no parent right now"**

    Yang 的关键发现. 在 dfs 里 `isRoot=true` 不是"我是输入的 root", 而是"调用我的那一层不会作为我的父亲挂着我了". 两种情况:

    - 入口调用: 原 root 没父亲, `dfs(root, true)`.
    - 递归调用: 当前节点的父亲被删 → 传 `dfs(child, parent_deleted)`. 父亲被删意味着子节点失去父亲, 升级成森林根.

    所以递归调用永远写 `dfs(cur->left, deleted)` —— 把"我是否被删" 当成"你是否失去父亲" 直接向下传.

2. **`vector<int>` 查包含 → 升级成 `unordered_set<int>` / Batch lookup pattern**

    Yang 的另一个关键观察. `find(vec.begin(), vec.end(), x)` 是 O(n), 对每个节点查一遍 → 总 O(n·k). 改成 `unordered_set<int>::count` 后 O(1) 查找, 总 O(n + k).

    **C++ 肌肉记忆**: 看到 `vector<int>` + "判某值在不在里面" → 第一反应建个 `unordered_set`. Python/JS 一样, 用 `set` / `Set` 而不是 list/array linear search.

3. **后序收集 + 删除模板的扩展 / Post-order with side collection**

    跟 0669/0814 一样的 `root->child = dfs(child, ...)` 骨架, 加一个"sub-effect": 收集森林根到全局 `forest` 里. 这种"递归主要返回值 + 顺手收集副结果" 是树题里常用扩展, 同款见 [0257 paths](../0257-binary-tree-paths/README.md), 0113 path sum II.

4. **微观察: `if (!cur->left && !cur->right) return nullptr` 是冗余的 / Dead code**

    Yang 在 v1 写了它, v2 删掉. 原因: 函数末尾本来就有"如果该删 return null, 否则 return cur". 这个特判只是把"叶子被删"提前一行返回, 没有正确性增益.

### 一句话总结

**后序 DFS, 删除模板扩展: `dfs(cur, isRoot)` 里, 没被删 + 没父亲 → 加入森林; 然后用 `dfs(child, cur_is_deleted)` 把"父亲被删"传给下层 (子节点会变成新森林根候选); 最后被删的话返回 null 砍断挂载. 用 `unordered_set` 查 to_delete, 别用 vector + find.**

## Solution

### Variant A — v1 提交版 (没用 isRoot 抽象, 手动判分支)

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* traversal(TreeNode* cur, vector<int>& to_delete, vector<TreeNode*>& res) {
            if (!cur) return nullptr;
            cur->left  = traversal(cur->left,  to_delete, res);
            cur->right = traversal(cur->right, to_delete, res);
            // 当前命中? (vector + find, O(k) 每次)
            if (find(to_delete.begin(), to_delete.end(), cur->val) != to_delete.end()) {
                if (!cur->left && !cur->right) return nullptr;     // 冗余但不错
                if (cur->left)  res.push_back(cur->left);
                if (cur->right) res.push_back(cur->right);
                return nullptr;
            }
            return cur;
        }
        vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
            vector<TreeNode*> res;
            // 入口特判: root 自己没被删 → 它是森林第一个根
            if (find(to_delete.begin(), to_delete.end(), root->val) == to_delete.end())
                res.push_back(root);
            traversal(root, to_delete, res);
            return res;
        }
    };
    ```

### Variant B — 改进版 (`unordered_set` + `isRoot` 抽象)

=== "C++"
    ```cpp
    class Solution {
    public:
        vector<TreeNode*> forest;
        unordered_set<int> toDel;

        TreeNode* dfs(TreeNode* cur, bool isRoot) {
            if (!cur) return nullptr;
            bool deleted = toDel.count(cur->val);                 // O(1) hash 查
            // 当前没被删 且 没父亲 (原根 或 父亲被删) → 加入森林
            if (!deleted && isRoot) forest.push_back(cur);
            cur->left  = dfs(cur->left,  deleted);                // 把"我是否被删"传给孩子 = "你失去父亲了吗"
            cur->right = dfs(cur->right, deleted);
            return deleted ? nullptr : cur;
        }
        vector<TreeNode*> delNodes(TreeNode* root, vector<int>& to_delete) {
            toDel = unordered_set<int>(to_delete.begin(), to_delete.end());
            dfs(root, true);                                       // 入口: root 没父亲
            return forest;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def delNodes(self, root: 'TreeNode', to_delete: list[int]) -> list['TreeNode']:
            # set(iterable): 把 list 转成哈希集合, O(1) 查 `x in s`.
            # 等价 C++ 的 unordered_set<int>(begin, end).
            to_del = set(to_delete)
            forest: list = []

            def dfs(cur, is_root):
                if not cur:
                    return None
                deleted = cur.val in to_del
                if not deleted and is_root:
                    forest.append(cur)
                cur.left  = dfs(cur.left,  deleted)
                cur.right = dfs(cur.right, deleted)
                return None if deleted else cur

            dfs(root, True)
            return forest
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {number[]} to_delete
     * @return {TreeNode[]}
     */
    var delNodes = function(root, to_delete) {
        // new Set(iterable): JS 哈希集合, .has(x) 是 O(1). 比 array.includes O(n) 快多了.
        //   C++ 等价: unordered_set<int>; Python 等价: set(...).
        const toDel = new Set(to_delete);
        const forest = [];

        const dfs = (cur, isRoot) => {
            if (!cur) return null;
            const deleted = toDel.has(cur.val);
            if (!deleted && isRoot) forest.push(cur);
            cur.left  = dfs(cur.left,  deleted);
            cur.right = dfs(cur.right, deleted);
            return deleted ? null : cur;
        };
        dfs(root, true);
        return forest;
    };
    ```

## Complexity

| | Time | Space |
|---|---|---|
| Variant A (vector + find) | **O(n·k)** | O(k) for to_delete + O(h) recursion |
| Variant B (unordered_set) | **O(n + k)** | O(k) set + O(h) recursion |

n = 节点数, k = to_delete 长度. 大数据 A 容易 TLE.

## 易错点

- **入口 root 的特殊处理 (v1 风格)**: v1 在 `delNodes` 入口手动判"root 是否在 to_delete 里, 不在就 push 进 res"; v2 用 `dfs(root, true)` 让逻辑统一在 dfs 内. 后者更干净, 不容易漏特例.
- **传 `isRoot=deleted` 给孩子是关键**: 别写成传 `false` 或 `isRoot`. 含义是"我作为父亲是否还存在": 我被删 → 你失去父亲 → 你 isRoot. 这一行是整个算法的语义压缩点.
- **vector + find 在工业代码里也写不出来**: `to_delete` 长几百个时 O(n·k) 就崩了. **看到 "vector<int> 判是否包含"** 立刻条件反射改 `unordered_set`. C++ 这是 idiom, Python `set`, JS `Set` 同理.
- **后序顺序重要**: 必须先 dfs 两个孩子 (它们已经在自己的子树里收集了森林根) 再处理自己. 写成前序会让"删自己"先发生, 孩子来不及把它当父亲处理.
- **节点引用 vs 节点值**: 题目保证值唯一, 所以用值查 set 没问题. 如果题目允许重复值, 必须按节点指针查 (用 `unordered_set<TreeNode*>`).
- **被删节点的左右孩子可能也被删**: 这种情况靠递归自然处理 —— 父亲被删时, 它的孩子在递归中如果也被删了, 那一支 return null, 不会被错误收集.

## 相关题目

- [0669. Trim a Binary Search Tree](../0669-trim-a-binary-search-tree/README.md) — 树删除模板, 前序判删
- [0814. Binary Tree Pruning](../0814-binary-tree-pruning/README.md) — 树删除模板, 后序判删
- [0450. Delete Node in a BST](../0450-delete-node-in-a-bst/README.md) — 删单点重组子树
- [0257. Binary Tree Paths](../0257-binary-tree-paths/README.md) — 同款"递归 + 全局收集副结果"
- 0113. Path Sum II (待补) — 同款"DFS + 收集合格路径"
- 0863. All Nodes Distance K in Binary Tree (待补) — 树 + BFS, 同样要把树看作图扩展
