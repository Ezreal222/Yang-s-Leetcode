# 06 · Stack & Queue — Summary / 总结

**EN**: Fast review for the 9 problems in this category. Each block: the *one* idea that unlocks the problem, plus a takeaway you can reuse on different problems with the same shape.

**中文**: 这一章 9 题的速复习. 每题两块: 一个能让你"啊就是这个"的核心 idea, 加上能搬到别的题上的套路.

---

## 0020. Valid Parentheses / 有效的括号

**Key idea / 核心**: 看到左括号, push 它**对应的右括号**进栈. 之后碰到右括号直接和 `top()` 比 —— 不用分支 (`(↔)` / `{↔}` / `[↔]`), 不用映射表.

**Takeaway / 套路**: **Stack as canceler** —— "push expectation, pop on match". 任何"配对消除"问题都是这个形状: 栈里维护"还没被满足的期待", 来一个能匹配的就消, 否则就压.

---

## 0071. Simplify Path / 简化路径

**Key idea / 核心**: 把路径按 `/` 切段, 普通目录 push, `..` 时 pop (注意空栈别 pop), `.` / 空串跳过. 栈底到顶 join 起来加个前导 `/` 就是答案.

**Takeaway / 套路**: **Stack as undoer** —— 凡是题目里出现"返回上一级 / 撤销 / 嵌套折叠"语义都是它 (路径、解码 0394、退格 0844). 配套技能: 字符串处理通用三步 `split → 逐段处理 → join`. C++ 没有原生 split, 用 `istringstream + getline`; Python/JS 直接 `.split('/')`.

---

## 1047. Remove All Adjacent Duplicates In String / 删除字符串中的所有相邻重复项

**Key idea / 核心**: 跟 0020 一模一样的形状, 把"匹配"换成"相等". 扫一遍, 跟栈顶相同就 pop, 否则 push. 栈里剩下的就是答案 (用 `string` 当栈直接省掉 `reverse`).

**Takeaway / 套路**: 题目说"反复消除相邻 X" → 栈扫一遍 O(n). 不要被"反复"骗得用循环重扫 —— 栈天然处理连锁消除 (一对消掉, 新邻居自动暴露给栈顶判断).

---

## 0150. Evaluate Reverse Polish Notation / 逆波兰表达式求值

**Key idea / 核心**: 数字 push, 运算符 pop 两个再算. 操作数顺序: **先 pop = 右操作数 (num1)**, 后 pop = 左 (num2), 计算是 `num2 op num1`. RPN 的全部价值: 不用解析优先级, 不用括号.

**Takeaway / 套路**: **Stack as evaluator** —— "push values, fold on operator". 一切后缀 / 树展开 / 编译器中间表示都是这个套路. 别忘了 Python 截断要 `int(a/b)` 不是 `//`.

---

## 0155. Min Stack / 最小栈

**Key idea / 核心**: O(1) `getMin` 靠"在 push 时把当时的最小值缓存进节点"——空间换时间, 利用栈"pop 自动恢复历史快照"的特性. 优化版用辅助栈, 只在最小值真的变小时 push (`<=` 而不是 `<`), 这就接上单调栈思想了.

**Takeaway / 套路**: **Cache aggregate at write-time** —— 看到"O(1) 查询某种聚合值 (min/max/sum/count)", 先想能不能在 insert 时顺手算好. 同款骨架: 前缀和、稀疏表、Trie 计数、并查集 size.

---

## 0225. Implement Stack using Queues / 用队列实现栈

**Key idea / 核心**: 一个队列就够 —— 入队后把前 `n-1` 个元素轮转到队尾, 最新入队的就跑到队头, 直接弹就是栈顶.

**Takeaway / 套路**: **Rotation trick** —— 当你要从 FIFO 模拟 LIFO (或反过来), 想"轮转"而不是"再来一个容器". 类似的还有循环数组、队列模拟双端队列.

---

## 0232. Implement Queue using Stacks / 用栈实现队列

**Key idea / 核心**: 两个栈 `stIn` / `stOut`, **懒搬运**: 只有 `stOut` 空了才把 `stIn` 整个倒过来. 这样每个元素最多搬两次 → 均摊 O(1).

**Takeaway / 套路**: **Amortized / lazy transfer** —— 不到不得已不动数据, 把开销摊平. 这个 mindset 在 dynamic array (vector 扩容)、splay tree、persistent data structure 里都见得到.

---

## 0239. Sliding Window Maximum / 滑动窗口最大值

**Key idea / 核心**: 队列里**只装可能成为最大值的候选**. 维护单调递减 deque: 新元素来了, 把队尾比它小的全踢掉 (反正以后它们也轮不上); 队头出窗口的话再 pop_front. 每个元素入队一次出队一次 → O(n).

**Takeaway / 套路**: **Monotonic deque for sliding extrema** —— 滑动窗口 + 极值 (max / min) → 单调队列, O(n). 推广: 比较条件可以是任何弱序, 比如 ≤ / ≥ / 长度. Python 存下标比存值更顺 (`dq[0] <= i - k` 一行判窗口).

---

## 0347. Top K Frequent Elements / 前 K 个高频元素

**Key idea / 核心**: 数完频次后, **min-heap of size k** —— 维护一个大小为 k 的最小堆, 新来的和堆顶比, 大就替换. 堆里最后剩的就是 Top-K. 频次有上界 (=n) 时换成 **bucket sort** 直接 O(n).

**Takeaway / 套路**: **Bounded heap for Top-K** —— 想要 top-largest k 个就用 min-heap of size k (堆顶是当前"最差", 一查 O(1) 一踢 O(log k)). 对称地, top-smallest 用 max-heap. C++ comparator 反着写 (`Cmp(a,b)==true` → a 沉), Python 直接 `heapq.nlargest` / `Counter.most_common`.

---

## Cross-cutting patterns / 跨题套路

回头看, 这一章的招式就 6 种:

| 套路 | 长什么样 | 出现于 |
|---|---|---|
| **Stack as canceler / undoer** | push 期待 / 字符, 匹配或"撤销指令"出现就消 | 0020, 0071, 1047 |
| **Stack as evaluator** | push 操作数, 运算符 fold 栈顶若干个 | 0150 |
| **Stack/Queue adapter** | 容器互相模拟, 关键是懒搬运 / 轮转 | 0225, 0232 |
| **Cache aggregate at write-time** | push 时把"当下的聚合值"存好, 查询 O(1) | 0155 |
| **Monotonic deque** | 维护"还可能赢"的候选, 滑动极值 O(n) | 0239 |
| **Bounded heap (size k)** | min-heap-of-k 取 top-largest, 反过来取 top-smallest | 0347 |

**遇到新题怎么先猜方向**:

- 看到"括号 / 配对 / 相邻消除 / 撤销 / 上一级 / 嵌套" → Stack as canceler / undoer.
- 看到"后缀表达式 / 表达式求值" → Stack as evaluator.
- 看到"用 X 实现 Y, 而 X 和 Y 操作方向相反" → 找 lazy / 轮转.
- 看到"O(1) 查询聚合值 (min/max/sum/...)" → 在 push 时把答案算好缓存.
- 看到"滑动窗口 + 最大/最小" → Monotonic deque.
- 看到"前 K 个 / 第 K 大" → Bounded heap. 频次/范围有界 → bucket sort 更快.

**容易踩的几个坑统一记一下**:

- C++ `stack::pop()` 不返回值 → 永远 `top()` 拿值再 `pop()`.
- C++ `priority_queue` 的 comparator 反着读 (`Cmp(a,b)==true` → a 优先级低 → a 沉).
- Python `//` 是 floor, `int(a/b)` 才是向 0 截断 —— RPN 这种题踩过别再踩.
- 单调队列里, `pop` 是值匹配 / 下标判窗口, 不是无脑弹 (老元素可能早被踢掉了).
- Top-K 用 max-heap 弹 k 次会变 O(n log n), min-heap of k 才是 O(n log k).
