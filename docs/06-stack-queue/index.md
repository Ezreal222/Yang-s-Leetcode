# 06 · Stack & Queue / 栈与队列

**EN**: LIFO/FIFO patterns, parens matching, expression parsing, monotonic queue, heap-based Top-K.

**中文**: 栈/队列模式 — 括号匹配、表达式解析、单调队列、堆求 Top-K.

> 📚 **[Category Summary / 总结](./summary.md)** — key idea + takeaway for each problem, plus the recurring patterns across all of them. Read this before review.

## 题目分类 / Problems by Category

> 12 题分 6 类. 同类题共享**"用什么结构 + 怎么用"** 的思维.

### 括号匹配 / Parentheses Matching

**特征**: 栈的经典应用 — 遇**左括号**入栈, 遇**右括号**弹栈匹配. 变体在"发现不合法怎么处理" 或"计分规则".

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0020 | [Valid Parentheses / 有效的括号](./0020-valid-parentheses/README.md) (母题) | Easy | ✅ | ☑ ☐ ☐ |
| 1249 | [Minimum Remove to Make Valid Parentheses / 移除无效的括号](./1249-minimum-remove-to-make-valid-parentheses/README.md) (删非法) | Medium | ✅ | ☐ ☐ ☐ |
| 0856 | [Score of Parentheses / 括号的分数](./0856-score-of-parentheses/README.md) (计分变体) | Medium | ✅ | ☐ ☐ ☐ |

### 表达式与路径解析 / Expression & Path Parsing

**特征**: 用栈处理 tokens — 数字/操作符 或 目录段 — 遇 "反向操作" 弹栈.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0150 | [Evaluate Reverse Polish Notation / 逆波兰表达式求值](./0150-evaluate-reverse-polish-notation/README.md) (RPN 计算) | Medium | ✅ | ☑ ☐ ☐ |
| 0071 | [Simplify Path / 简化路径](./0071-simplify-path/README.md) (`..` 弹栈) | Medium | ✅ | ☐ ☐ ☐ |

### 相邻消除 / Adjacent Elimination

**特征**: 栈顶跟当前元素**匹配即弹**, 否则入栈. 相邻同色/同字符自动消掉.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 1047 | [Remove All Adjacent Duplicates In String / 删除字符串中的所有相邻重复项](./1047-remove-all-adjacent-duplicates-in-string/README.md) | Easy | ✅ | ☑ ☐ ☐ |

### 栈/队列设计 / Stack/Queue Design

**特征**: **用另一种结构实现** (栈实现队列 / 队列实现栈), 或**增强栈** (支持 O(1) 取 min).

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0155 | [Min Stack / 最小栈](./0155-min-stack/README.md) (O(1) getMin) | Medium | ✅ | ☐ ☐ ☐ |
| 0225 | [Implement Stack using Queues / 用队列实现栈](./0225-implement-stack-using-queues/README.md) | Easy | ✅ | ☑ ☐ ☐ |
| 0232 | [Implement Queue using Stacks / 用栈实现队列](./0232-implement-queue-using-stacks/README.md) (双栈, 摊销 O(1)) | Easy | ✅ | ☑ ☐ ☐ |

### 单调队列 / Monotonic Queue (滑窗极值)

**特征**: **deque** 维护单调性, 求**定长窗口极值** O(n) 摊销. 跟 §11 单调栈方向不同但思想同源.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0239 | [Sliding Window Maximum / 滑动窗口最大值](./0239-sliding-window-maximum/README.md) (母题) | Hard | ✅ | ☑ ☐ ☐ |

### 优先队列 (堆) / Priority Queue (Heap) — Top-K

**特征**: **大小 k 的 min/max heap** 求 Top-K. 通用模板: `push → if size > k → pop`.

| #    | Title | Difficulty | Status | Reviewed |
|------|-------|------------|--------|----------|
| 0347 | [Top K Frequent Elements / 前 K 个高频元素](./0347-top-k-frequent-elements/README.md) (频次 Top-K) | Medium | ✅ | ☑ ☐ ☐ |
| 0215 | [Kth Largest Element in an Array / 数组中的第 K 个最大元素](./0215-kth-largest-element-in-an-array/README.md) (min-heap + quickselect) | Medium | ✅ | ☐ ☐ ☐ |
| 1046 | [Last Stone Weight / 最后一块石头的重量](./1046-last-stone-weight/README.md) (max-heap 模拟) | Easy | ✅ | ☐ ☐ ☐ |
| 0973 | [K Closest Points to Origin / 最接近原点的 K 个点](./0973-k-closest-points-to-origin/README.md) (max-heap of k) | Medium | ✅ | ☐ ☐ ☐ |
| 0703 | [Kth Largest Element in a Stream / 数据流中的第 K 大元素](./0703-kth-largest-element-in-a-stream/README.md) (流式 min-heap) | Easy | ✅ | ☐ ☐ ☐ |
