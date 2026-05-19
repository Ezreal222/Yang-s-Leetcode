# 0129. Sum Root to Leaf Numbers / 求根节点到叶节点数字之和

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Tree, DFS, Recursion · 二叉树, 深度优先搜索, 递归
    - **Link**: [LeetCode](https://leetcode.com/problems/sum-root-to-leaf-numbers/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## Problem

**EN**: Every root→leaf path spells out a number (each node's val is one digit, root = most significant). Return the sum of all root→leaf numbers.

**中文**: 每条 root→leaf 路径上的节点值拼出一个十进制数 (根是最高位). 求所有 root→leaf 数字之和.

## 思路

### Core idea

一次 DFS, 把"到当前节点为止拼出的整数" `curSum` 作为参数传下去. 每进入一个节点先 `curSum = curSum * 10 + cur->val`, 走到叶子就把它加进全局 `sum`. 标准的 **"路径状态作为参数 + 答案做全局累加"** 模板.

### Key Insights

1. **`curSum * 10 + val` = 字符串→整数 的标准构造法 / Build the number digit-by-digit**

    走一层就把已有数字左移一位 (`×10`), 再把当前位 `+val`. 比 string 拼接版省一大截: 字符串版每次拼接 O(len), 叶子还要 `stoi` 再扫一遍; int 版每步 O(1).

2. **DFS 参数 vs 全局状态: 各管一摊 / Path state vs answer state**

    - `curSum` 当参数传下去 — 路径状态, 每个递归调用版本独立, 子调用改它不会回流到父.
    - `sum` 是类成员 / 全局 — 跨调用累加最终答案, 必须共享.

    同 [0257](../0257-binary-tree-paths/README.md) / [0112](../0112-path-sum/README.md) 的"路径状态参数 + 答案全局"模式. 一旦定下来"什么是路径状态、什么是最终答案", 代码骨架就固定了.

3. **隐式回溯 / Implicit backtracking via pass-by-value**

    `int curSum` 是按值传, 父函数自己的 `curSum` 永远不会被子调用动. 走完左子树回到右子树, `curSum` 自动是"还没走左子树时的值" — 不需要手动 push/pop. (string 版同理.) 这是参数传递天然送你的 free backtracking.

4. **Refactor: 把"累加当前位"提到入口 / Hoist the digit-add to the entry**

    v1 (string 版) 在叶子分支算一次、在两个递归调用前各算一次 — **写了 3 遍**. v2 把它挪到 `if (!cur) return` 之后第一行, 每个节点访问时算一次, 自动同时处理叶子和内部节点两种情况. 是"每个调用入口固定做一件事"的通用 refactor.

### 一句话总结

**一次 DFS, 把"到这里为止拼出来的整数"作为参数传下去, 到叶子就加进 sum. 每层只做一行 `curSum = curSum*10 + val` 就够了.**

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int sum = 0;
        void dfs(TreeNode* cur, int curSum) {
            if (!cur) return;
            curSum = curSum * 10 + cur->val;          // 拼上当前位
            if (!cur->left && !cur->right) {          // 叶子: 收一条路径的数字
                sum += curSum;
                return;
            }
            dfs(cur->left,  curSum);                  // int 按值传 → 隐式回溯
            dfs(cur->right, curSum);
        }
        int sumNumbers(TreeNode* root) {
            dfs(root, 0);
            return sum;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def sumNumbers(self, root: 'TreeNode | None') -> int:
            self.sum = 0
            def dfs(node, cur):
                if not node:
                    return
                cur = cur * 10 + node.val          # int 是不可变, 重新绑定不影响父调用 → 隐式回溯
                if not node.left and not node.right:
                    self.sum += cur
                    return
                dfs(node.left,  cur)
                dfs(node.right, cur)
            dfs(root, 0)
            return self.sum
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {TreeNode} root
     * @return {number}
     */
    var sumNumbers = function(root) {
        let sum = 0;                               // 闭包变量, 跨 dfs 调用共享 (= C++ 类成员)
        const dfs = (node, cur) => {
            if (!node) return;
            cur = cur * 10 + node.val;             // number 按值传, 子调用改不到父
            if (!node.left && !node.right) {
                sum += cur;
                return;
            }
            dfs(node.left,  cur);
            dfs(node.right, cur);
        };
        dfs(root, 0);
        return sum;
    };
    ```

### v1 (string 版, 慢但思路清晰)

留作对比 — 字符串拼接走一遍, 叶子 `stoi` 转回数字. 算法对, 但每步多扫一遍字符, 整体常数大不少.

```cpp
void dfs(TreeNode* cur, string curSum) {
    if (!cur) return;
    if (!cur->left && !cur->right) {
        curSum += to_string(cur->val);
        sum += stoi(curSum);
        return;
    }
    dfs(cur->left,  curSum + to_string(cur->val));
    dfs(cur->right, curSum + to_string(cur->val));
}
```

## Complexity

- **Time**: O(n) — 每个节点访问一次.
- **Space**: O(h) recursion. 题目约束保证答案落在 int 范围内, 不需要 long long.

## 易错点

- **`sum` 别声明成局部 int**: 同 0257/0538 一样的坑. 递归之间需要共享, 必须类成员 / 闭包 / 实例属性.
- **叶子判定不能漏**: `!cur->left && !cur->right` 才算叶子. 写成 `if (!cur)` 就 base case 而已, 没收数字 → 答案永远 0.
- **`curSum * 10 + val` 不要写成 `curSum + val`**: 经典笔误, 把"拼数字"写成"加和", 这题就废了.
- **改写后的位置很关键**: `curSum = curSum * 10 + cur->val` 必须在 `if (!cur) return` 之后. 写在前会对 nullptr 解引用; 写在 dfs 调用里 (像 v1 那样) 是 string 版的旧写法 — 改 int 版后要挪到入口.
- **题目变体: 路径里出现负数 / 进制不是 10**: 把 `*10` 改成 `*base`, 加和逻辑不变. 思维通用.

## 相关题目

- [0257. Binary Tree Paths](../0257-binary-tree-paths/README.md) — 同款"路径状态作参数", 收集成字符串 list (本题的字符串版思路就是从它来的)
- [0112. Path Sum](../0112-path-sum/README.md) — 同款 root→leaf 累加, 只是答案是 bool 不是 sum
- [0404. Sum of Left Leaves](../0404-sum-of-left-leaves/README.md) — 同款"叶子聚合", 但只挑左叶子
- [0113. Path Sum II](../0113-path-sum-ii/README.md) — 0112 的进阶, 收集所有满足和的完整路径, 用显式回溯
- 0988. Smallest String Starting From Leaf (待补) — 反向版: 叶子→根拼字符串, 取字典序最小
