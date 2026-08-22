# 0202. Happy Number / 快乐数

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Set, Floyd's Cycle Detection, Math · 哈希集合, Floyd 判环, 数学
    - **Link**: [LeetCode](https://leetcode.com/problems/happy-number/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Iterate digit-square-sum: reaches 1 = happy, else cycles**. Detect cycle with **hash set** (O(n) extra) — or elegantly with **Floyd tortoise/hare** on the sequence (O(1) extra, same trick as [0142](../../02-linked-list/0142-linked-list-cycle-ii/README.md)).
>
> **中文**: **反复算各位平方和: 到 1 = 快乐, 否则进环**. **hash set** 判环 (O(n) 空间), 或用 **Floyd 龟兔** 直接在数字序列上跑, O(1) 空间.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 判断 `n` 是否为"快乐数": 反复用**各位数字的平方和** 替换 n, **最终变成 1** 就是快乐; 否则会**陷入循环** 返 false.

**中文**: 每步替换成"各位平方和", 到 1 是快乐, 陷环是不快乐.

## Key Insights

1. **🔑 灵魂洞察: 序列**要么到 1 要么进环** / The sequence either reaches 1 or cycles**

    数学证明:

    - **三位数以上** 的 squareSum 严格小于原数 (例: 4位数最大 9999 → sum ≤ 4·81 = 324, 远小于 4 位下界 1000).
    - → 序列必然**下降进入 [1, 999]** 区间.
    - **有限值域 + 确定性下一步** → 无穷序列必**周期**.
    - **周期含 1**  ⇒ happy (因为 squareSum(1) = 1 自环). **不含 1** ⇒ 陷入其他环.

    > 事实: **所有 unhappy 数**最终都会进入这个 **8 元环**: `4 → 16 → 37 → 58 → 89 → 145 → 42 → 20 → 4`. 记住是趣味题.

2. **🔑 把数字序列当"隐式链表" / Treat the number sequence as an implicit linked list**

    定义 `next(n) = squareSum(n)`. 于是"从 n 出发不断迭代" 就是**沿着一个虚拟链表走**. 判环问题变成 [0142 Linked List Cycle II](../../02-linked-list/0142-linked-list-cycle-ii/README.md) — **同款问题**, 只是节点不是 struct 而是 int.

    > **抽象等价**: Floyd 判环不管数据类型, 只要有"next 函数" 就能用. 这是算法**通用性** 的漂亮体现.

3. **🔑 两种判环方法对比 / Two cycle-detection methods**

    | 方法 | Time | Space | 备注 |
    |---|---|---|---|
    | **hash set** (v1 / v3) | O(k) | **O(k)** | 直观易写 |
    | **Floyd 龟兔** (v2) | O(k) | **O(1)** | 秀操作, 数学等价 [0142](../../02-linked-list/0142-linked-list-cycle-ii/README.md) |

    k = 序列进环前的长度 + 环长. 都是常数级 (值域有限).

    > **面试问 O(1) 空间** → 上 Floyd. 若追问"证明为啥能行" → 拉出 0142 的数学.

4. **🔑 v1 (hash set): 每步查 & 存 / Hash set: check & insert**

    ```
    while n != 1:
        若 n 已见过 → 环, 返 false
        记录 n; 推进 n = squareSum(n)
    返 true
    ```

    `while (n != 1 && !seen.count(n))` 是**双终止条件**: 中奖到 1 就 happy, 否则重复见就 unhappy. 循环结束后**看 n 是不是 1** 决定结果.

5. **🔑 v2 (Floyd): slow 1 步, fast 2 步 / Floyd: slow steps 1, fast steps 2**

    ```cpp
    do {
        slow = squareSum(slow);
        fast = squareSum(squareSum(fast));    // 2 步一动
    } while (slow != fast);
    return slow == 1;                          // 相遇后看在不在 "1 自环"
    ```

    - **`do-while` 而不是 `while`**: 起始 slow == fast == n → 直接判会立刻退出. 需要**先动一次**再比较. `do-while` 天然做这件事.
    - **相遇处若是 1** → 序列进入了"1 自环" → happy.
    - 相遇处**不是 1** → 进了别的环 → unhappy.

    > **`do-while` 是 Floyd 的常见 pattern**. 起点相同的两指针必须"先走再比".

6. **🔑 squareSum helper: 十进制拆位 / squareSum: digit split**

    ```cpp
    while (n > 0) {
        int d = n % 10;      // 取末位
        sum += d * d;        // 累加平方
        n /= 10;             // 去末位
    }
    ```

    - `n % 10` 取末位, `n /= 10` 去末位 — **十进制拆解** 的标准招式.
    - `sum` 复用作累加器 (v3 里 Yang 用 `curr` 避免跟外层 n 混).

    > 记住"**mod-10 + div-10 逐位处理**" 一次, 到处用: 数位反转, 数位求和, digit DP, Base-10 palindrome...

7. **🔑 v3 vs v1: 内联 helper 的取舍 / v3 vs v1: inline helper**

    v3 把 squareSum 写在主循环里, 少一层函数调用, 变量 `curr` 隔开外层 n. **可读性略差**, 性能微小. 面试时**推荐 v1 抽 helper** — 单一职责.

8. **复杂度 / Complexity**

    - **Time**: O(k) — k = 序列长度, 有界 (值域有限).
    - **Space**: **hash set O(k)** / **Floyd O(1)**.

## Solution

=== "C++"

    **v1: hash set (推荐首选)**

    ```cpp
    class Solution {
    public:
        bool isHappy(int n) {
            unordered_set<int> seen;
            while (n != 1 && !seen.count(n)) {
                seen.insert(n);
                n = squareSum(n);
            }
            return n == 1;
        }
    private:
        int squareSum(int n) {
            int sum = 0;
            while (n > 0) {
                int d = n % 10;
                sum += d * d;
                n /= 10;
            }
            return sum;
        }
    };
    ```

    **v2: Floyd 龟兔 (O(1) 空间, 秀操作)**

    ```cpp
    class Solution {
    public:
        bool isHappy(int n) {
            int slow = n, fast = n;
            do {
                slow = squareSum(slow);
                fast = squareSum(squareSum(fast));       // fast 每轮走 2 步
            } while (slow != fast);
            return slow == 1;                            // 相遇处是不是 "1 自环"?
        }
    private:
        int squareSum(int n) {
            int sum = 0;
            while (n > 0) { int d = n % 10; sum += d * d; n /= 10; }
            return sum;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v1: hash set (推荐)
        def isHappy(self, n: int) -> bool:
            seen = set()
            while n != 1 and n not in seen:
                seen.add(n)
                # sum + genexp + str(n) 是 Pythonic 拆位方案
                # str(n) 得到 "123", 每个字符转 int 再平方. 比 while-mod-10 短
                n = sum(int(c) ** 2 for c in str(n))
            return n == 1

        # v2: Floyd
        def isHappy_floyd(self, n: int) -> bool:
            def sq(x):
                return sum(int(c) ** 2 for c in str(x))
            slow, fast = n, n
            # Python 没 do-while, 用 while True + break 或"先走一次再进循环"
            slow = sq(slow); fast = sq(sq(fast))
            while slow != fast:
                slow = sq(slow)
                fast = sq(sq(fast))
            return slow == 1
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {number} n
     * @return {boolean}
     */
    var isHappy = function(n) {
        // v1: hash set. JS 的 Set 跟 C++ unordered_set / Python set 同 API
        const seen = new Set();
        const sq = (x) => {
            let s = 0;
            while (x > 0) {
                const d = x % 10;
                s += d * d;
                x = Math.floor(x / 10);     // 注意 JS 的 / 是浮点除, 必须 Math.floor
            }
            return s;
        };
        while (n !== 1 && !seen.has(n)) {
            seen.add(n);
            n = sq(n);
        }
        return n === 1;
    };
    ```

## Complexity

- **Time**: O(k) — k = 序列长度, 常数级 (值域有限).
- **Space**: O(k) hash set / **O(1) Floyd**.

## 相关题目

- [0142. Linked List Cycle II](../../02-linked-list/0142-linked-list-cycle-ii/README.md) — Floyd 龟兔母题
- [0141. Linked List Cycle](../../02-linked-list/0141-linked-list-cycle/README.md) — 只判环
- [0287. Find the Duplicate Number](../../02-linked-list/0287-find-the-duplicate-number/README.md) — **Floyd 判环用在数组上**, 神应用
- [0349. Intersection of Two Arrays](../0349-intersection-of-two-arrays/README.md) — hash set 用法
- [0242. Valid Anagram](../0242-valid-anagram/README.md) — 计数数组基础
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set + "只从头扩"
- 1071\. Greatest Common Divisor of Strings (待补) — 数学 + 判"进不进环" 思路
