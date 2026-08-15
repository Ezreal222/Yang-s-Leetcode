# 0236. Lowest Common Ancestor of a Binary Tree / 二叉树的最近公共祖先

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion, LCA · 二叉树, 深度优先搜索, 递归, 最近公共祖先
    - **Link**: [LeetCode](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☑ ☐

## Problem

**EN**: Find the **lowest (deepest) common ancestor** of two given nodes `p` and `q` in a **general** binary tree (no BST property, no parent pointers). Both `p` and `q` are guaranteed to exist.

**中文**: 在**普通**二叉树里找两个节点 `p` 和 `q` 的**最近公共祖先** (没有 BST 性质, 也没有父指针). 保证 `p`、`q` 都存在.

## 思路

### Core idea

**后序递归 + 返回值处理**: 自底向上一路返回"我或我子树里看到了哪些目标". 当一个节点同时收到"左子树报告找到一个" + "右子树报告找到另一个", **这个节点就是 LCA** —— 因为它是第一个让 p 和 q 分别落在两个子树里的祖先.

如果 p、q 都在同一边, 那一边返回的就是答案 (更深处那个早被认作 LCA), 直接传上去.

### Key Insights

1. **走整棵树 + 处理返回值 / Visit-all + merge-return**

    属于 [0112 Key Insights](../0112-path-sum/README.md#key-insights) 表里**第二种**递归模式: 搜索整棵树, **必须处理返回值**, 用返回值在父节点处汇总左右子树的"发现". 跟 0112 (找一条路径就短路) 形成对比 —— 这里**不能短路**, 必须左右都跑.

2. **Base case 同时承担两个角色 / Base case is dual-purpose**

    `if (!root || root == p || root == q) return root;`

    - `!root` → 这条路走到尽头, 没看见 p/q.
    - `root == p` or `root == q` → 看见了**一个**目标, 把这个目标自己作为"信号"传上去.

    **重点**: 即使一个节点是 `p` 的祖先 (不是 p 本身), 它也只把"我子树里有 p"通过返回值反馈; 真正的 LCA 由父节点根据左右子树的返回综合判定.

3. **三种返回组合 / Three return patterns**

    | 左返回 | 右返回 | 当前节点该返回什么 | 含义 |
    |---|---|---|---|
    | non-null | non-null | **`root` (LCA found!)** | p 和 q 分居左右子树, root 就是 LCA |
    | non-null | null | `left` | 答案在左 (可能是 p, 或更深处的 LCA) |
    | null | non-null | `right` | 答案在右 |
    | null | null | `null` | 这棵子树跟 p/q 无关 |

    一旦 LCA 被确定 (case 1), 这个值一路沿调用栈往上传, 不会被覆盖 —— 因为更上层只看左右**子树**的返回, LCA 已经在某个子树里了, 它在上层只会作为唯一的非空返回向上传.

### 一句话总结

**后序递归. Base case 撞到 `null / p / q` 就向上传; 当前节点根据左右两侧返回值汇总: 两侧都非空 → 我是 LCA; 一侧非空 → 把那侧传上去. 关键是不能短路, 必须左右都跑完.**

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to find the **lowest common ancestor** of two nodes `p` and `q` in a binary tree. 'Lowest' meaning the **deepest** node that has both `p` and `q` in its subtree. And a node is considered its own descendant — so if `p` is `q`'s ancestor, then `p` itself is the LCA. Let me confirm a couple of things:"

- "Is this a **BST** or a **general** binary tree?" *(general — that's the harder version, no ordering property.)*
- "Are `p` and `q` both **guaranteed to exist** in the tree?" *(yes — simplifies the code.)*
- "Do nodes have **parent pointers**?" *(no — pure top-down.)*
- "Are node values **unique**?" *(yes — so pointer equality and value equality are the same, but I'll use pointer comparison to be safe.)*

### 2. Brainstorm approaches (1 min)

> "Let me think through a few options.
>
> **Approach 1 — naive**: for each node, check if both `p` and `q` are in its subtree, take the deepest. That's O(n²). Too slow.
>
> **Approach 2 — path comparison**: find the root-to-`p` path, find the root-to-`q` path, then walk them until they diverge. That's O(n) time, O(h) space, works fine but requires two full traversals and explicit path storage.
>
> **Approach 3 — recursive post-order**, which is the cleanest. The insight is: at every node, I ask *'what did my left subtree find? what did my right subtree find?'* If left found one target and right found the other, **I'm the LCA** — I'm the first node where `p` and `q` split. Otherwise I propagate up whichever side found something."

> "I'll go with approach 3. Single traversal, no extra data structures."

### 3. Sketch the algorithm before coding (1 min)

> "The recursive function:
>
> - **Base case**: if the current node is `null`, return `null`. If it's `p` or `q`, return itself — this acts as a **signal** to the parent: 'I saw one target here.'
> - **Recurse both sides** — I can't short-circuit; I need to know what both sides found.
> - **Combine**:
>   - If **both** sides returned non-null → `p` and `q` are split across my two subtrees → **I am the LCA**, return me.
>   - If **only one** side is non-null → return that one (it's either a target itself, or an already-found LCA deeper down).
>   - If both null → return null.
>
> Key subtlety: once the LCA is found deep in the tree, it just bubbles up unchanged — every ancestor above it sees `(non-null, null)` and forwards it."

### 4. Code while narrating (2 min)

> "Let me write it out."

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    // Base case: null, or hit one of the targets
    if (!root || root == p || root == q) return root;

    // Post-order: fully explore both sides before deciding
    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);

    // Combine
    if (left && right) return root;    // split → I'm the LCA
    return left ? left : right;         // forward the non-null side
}
```

> "That's it — surprisingly compact. The magic is in what the return value *means* at each level: 'something my subtree found that's relevant to the answer.'"

### 5. Trace an example (1 min)

> "Let me sanity-check with a small tree, `[3, 5, 1, 6, 2, 0, 8]`, and find the LCA of **5 and 1**:
>
> - At node 3: recurse left, recurse right.
> - Left recurses to node 5 → base case hits (`root == p`) → returns 5.
> - Right recurses to node 1 → base case hits → returns 1.
> - Both non-null at node 3 → return 3. ✓
>
> Now a trickier case: LCA of **5 and 4**, where 4 is a descendant of 5.
>
> - At node 3: right subtree has neither, returns null.
> - Left recurses down; node 5 hits base case and returns itself — we **don't keep searching for 4 inside 5's subtree**.
> - Node 3 sees `(left=5, right=null)` → returns 5.
> - Correct — 5 is the LCA of 5 and 4 because 5 is its own descendant."

> "That second case is the subtle one — the base case treating `root == p` as a stopping point is what makes 'ancestor is also allowed to be the LCA' work naturally."

### 6. Complexity (20s)

> "**Time O(n)** — every node visited exactly once. **Space O(h)** for the recursion stack — O(log n) for balanced, O(n) worst case for a skewed tree."

### 7. Edge cases + follow-ups (1 min)

> "Two things worth flagging:
>
> 1. **This assumes both `p` and `q` are in the tree.** If that weren't guaranteed, this code would return a false positive when only one exists — it'd return the one it found, thinking it was the LCA. To handle that, I'd add a second pass to verify, or change the recursion to return a `(node, found_p, found_q)` tuple. That's the follow-up problem 1644.
>
> 2. **For a BST, there's a much simpler O(h) solution** — we can walk one path using the ordering: if both `p` and `q` are less than root, go left; if both greater, go right; otherwise root is the LCA. That's 0235. Worth mentioning if the interviewer wants to see that I know the specialization."

> "Any follow-ups you'd like me to code?"

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
            // Base: null 或撞到目标本身
            if (!root || root == p || root == q) return root;

            // 后序: 先两边都跑完
            TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
            TreeNode* right = lowestCommonAncestor(root->right, p, q);

            // 汇总
            if (left && right) return root;   // p、q 分居两侧 → 当前就是 LCA
            return left ? left : right;        // 否则把非空那侧传上去
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
            if not root or root is p or root is q:
                return root
            left  = self.lowestCommonAncestor(root.left,  p, q)
            right = self.lowestCommonAncestor(root.right, p, q)
            if left and right:
                return root
            # `left or right`: Python 短路 or 直接返回第一个 truthy 值,
            # 两个都 None 时返回 None. 等价 C++ `left ? left : right`.
            return left or right
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @param {TreeNode} p
     * @param {TreeNode} q
     * @return {TreeNode}
     */
    var lowestCommonAncestor = function(root, p, q) {
        if (!root || root === p || root === q) return root;
        const left  = lowestCommonAncestor(root.left,  p, q);
        const right = lowestCommonAncestor(root.right, p, q);
        if (left && right) return root;
        // `left || right`: JS 短路 ||, 跟 Python 的 `or` 同款.
        // 第一个 truthy 就返回它; 全部 falsy 返回最后一个 (这里就是 null).
        return left || right;
    };
    ```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion.

## 易错点

- **不能短路 (跟 0112 区别)**: 这里**必须**先递归左**再**递归右, 然后才能判断当前是不是 LCA. 写成 `if (!left) return right;` 之类提前 return 会跳过右子树的搜索, 漏掉"另一目标在右"的情况. 0112 找一条路径找到就 return 是另一种递归模式 (见 0112 的 Key Insights 表).
- **Base case 要把 p/q "自己"也算上**: 即使 `root == p`, 你仍然返回 `root` (= p). 上层就根据"左/右子树是否非空"来判断当前 p 是不是同时也是 q 的祖先. 这种"把自己当作发现的信号"的写法是后序递归的常用招式.
- **保证 p、q 都存在**: 题目保证两者都在树中, 所以不需要额外检查"找不到"的情况. 否则要小心: 单边非空时返回那一边, 可能只找到了一个目标但另一个根本不在树里 (返回值会让调用者以为找到了 LCA).
- **`==` vs equality of values**: 比较的是**节点指针/引用**, 不是值. 因为题目里值唯一, `node.val == p.val` 也对; 但工程代码统一用 `node is p` (Python) / `node === p` (JS) / `node == p` (C++ pointer) 更稳, 避免值重复时翻车.
- **BST 版本 0235 简单多了**: BST 有大小性质, 不需要走两边 —— 只需要根据 `p.val`、`q.val` 跟 `root.val` 的关系决定走哪边, O(h) 但只走一条路径. 跟这里的"必须两边都跑"形成鲜明对比.

## 相关题目

- [0235. Lowest Common Ancestor of a Binary Search Tree / 二叉搜索树的最近公共祖先](../0235-lowest-common-ancestor-of-a-binary-search-tree/README.md) — BST 版, 用大小性质单向走, 不需要后序合并
- [0112. Path Sum](../0112-path-sum/README.md) — Carl 框架第三种 (找一条路径短路); 0236 是第二种 (走整棵 + 合并返回)
- [0098. Validate Binary Search Tree](../0098-validate-binary-search-tree/README.md) — 也是"后序 + 处理返回值"模式
- [0110. Balanced Binary Tree](../0110-balanced-binary-tree/README.md) — 同款后序 + 返回值短路 (用 -1 哨兵)
- 0865. Smallest Subtree with all the Deepest Nodes (待补) — LCA 的变体: 最深叶子的 LCA
- 1644. LCA II (待补) — p、q 不一定存在的版本, 要额外计数
