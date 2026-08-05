# 0025. Reverse Nodes in k-Group / K 个一组翻转链表

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Linked List, Reversal, Dummy Head · 链表, 反转, 虚拟头节点
    - **Link**: [LeetCode](https://leetcode.com/problems/reverse-nodes-in-k-group/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Reverse every k consecutive nodes** → dummy + `prev`; per group: walk `end` k steps (if `end` becomes null → not enough, return), then reverse **half-open `[start, end)`**, stitch: `prev->next = newHead`, `start->next = end`, `prev = start`.
>
> **中文**: **每 k 个一组反转** → dummy + prev; 每组: 走 end k 步 (中途 null 就返回), 反转**半开 `[start, end)`**, 缝合: `prev→next = newHead`, `start→next = end`, `prev = start`.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单链表, 每 **k 个一组反转**. **不足 k 个** 的尾部**保持原序**. 只能改指针, 不能改 val.

**中文**: k 个一组反转链表, 尾部凑不齐一组就不动.

## Key Insights

1. **🔑 灵魂设计: 半开区间 `[start, end)` / The killer design: half-open interval**

    Yang 的关键选择: helper `reverse(start, end)` 反转 **`[start, end)`** — **不包含 end**. 让 `end` 一变量担双职:

    - **反转的停止条件**: `while (curr != end)` — 到 end 停.
    - **下一段的起点**: 反转后 `start->next = end` 直接接上.

    > **"一个变量两重身份"** 是 API 设计里的漂亮 trick. 若用闭区间 `[start, last]`, 就得多带一个 `end = last->next` — 冗余.

2. **🔑 预扫 k 步验证够不够 / Look-ahead k to verify**

    走 `end` 前 k 步是**验证 + 定位** 一箭双雕:

    ```cpp
    ListNode* end = prev->next;
    for (int i = 0; i < k; i++) {
        if (!end) return dummy.next;      // 途中 null → 剩余不足 k → 停止不反转
        end = end->next;
    }
    ```

    - 走完 k 步: `end` 指向第 k+1 个 (=下一组的第一个), 或 null (若刚好用完).
    - 走**不完** k 步: `end` 提前变 null → 直接返, 剩余保持原序.

    > 面试常问"若最后一组不足 k 也反转" 变形 (0025 变体) → 只需**去掉预检**, 直接 reverse 到 null.

3. **🔑 反转 helper 是 [0206](../0206-reverse-linked-list/README.md) 的"半开版" / Reverse helper: 0206 with a stop condition**

    ```cpp
    ListNode* reverse(ListNode* start, ListNode* end) {
        ListNode* prev = nullptr;
        ListNode* curr = start;
        while (curr != end) {              // ← 唯一改动: 停在 end 而不是 null
            ListNode* next = curr->next;
            curr->next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
    ```

    循环结束: `curr == end`, `prev` = 反转段的**新头** (原本的 `end 的前一个`, 也是原来这段的第 k 个).

4. **🔑 缝合三行 / The stitch in three lines**

    反转完 `[start, end)` 后要**三处** 重新接指针:

    ```
    ... → prev → [start → ... → last] → end → ...        (反转前)
    ... → prev   [last → ... → start]   end → ...        (反转后, 段内已反, 但段与外界断开)
    ... → prev → last → ... → start → end → ...          (缝合完成)
    ```

    ```cpp
    prev->next = newHead;                  // 1. 前驱 → 新头 (last)
    start->next = end;                     // 2. 原 start (现新尾) → 下段头
    prev = start;                          // 3. 推进: 新尾成下组前驱
    ```

    > **顺序不能乱**. `newHead` 就是 helper 返的 `prev`, 是反转段的头; `start` 是原头 (=新尾).

5. **🔑 跟 [0024](../0024-swap-nodes-in-pairs/README.md) (k=2) 是同一模板 / Same template as 0024 (k=2)**

    | | 0024 (k=2) | **0025 (任意 k)** |
    |---|---|---|
    | 每组指针数 | prev / a / b (硬编码 2 个) | prev / start / end (通用) |
    | 每组反转 | 三步交换 | 调用 helper |
    | 推进 | `prev = a` | `prev = start` |
    | 尾部处理 | 循环条件自然过滤 (奇数长度 leftover) | **预扫 k 步验证** |

    > 二级到三级跳: **硬编码 k 个指针 → 一个 helper 处理任意 k**. 这就是"抽象" 在链表题里的体现.

6. **🔑 复杂度 O(n) / Linear**

    - **时间**: 每节点最多被访问 **两次** — 一次由外循环走 end, 一次由 helper 反转. 总 O(n).
    - **空间**: O(1) — dummy + prev + end + start + helper 里 3 指针, 常数.

    > 反转的 helper 内部走 k 步, 外循环走 `n / k` 次, 相乘还是 O(n). k 不改变阶.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        ListNode* reverseKGroup(ListNode* head, int k) {
            ListNode dummy(0);
            dummy.next = head;
            ListNode* prev = &dummy;

            while (true) {
                // 预扫 k 步: end 走到第 k+1 个 (下一组头), 或返回若不够
                ListNode* end = prev->next;
                for (int i = 0; i < k; i++) {
                    if (!end) return dummy.next;             // 不足 k, 剩下不反转
                    end = end->next;
                }
                ListNode* start = prev->next;                // 这组的头 (反转后是尾)
                ListNode* newHead = reverse(start, end);     // 反转 [start, end)

                // 缝合三步
                prev->next = newHead;
                start->next = end;
                prev = start;                                // 推进到"新尾" (= 下组前驱)
            }
        }

    private:
        // 反转 [start, end) 半开区间, 返回新头 (原 [start..end) 的最后一个)
        ListNode* reverse(ListNode* start, ListNode* end) {
            ListNode* prev = nullptr;
            ListNode* curr = start;
            while (curr != end) {                            // ← 半开: 到 end 停 (不含)
                ListNode* next = curr->next;
                curr->next = prev;
                prev = curr;
                curr = next;
            }
            return prev;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        def reverseKGroup(self, head, k: int):
            dummy = ListNode(0, head)
            prev = dummy
            while True:
                end = prev.next
                # for _ in range(k) 相当于 C++ for(int i=0; i<k; i++)
                for _ in range(k):
                    if end is None: return dummy.next
                    end = end.next
                start = prev.next
                new_head = self._reverse(start, end)
                prev.next = new_head
                start.next = end
                prev = start

        def _reverse(self, start, end):
            # 跟 C++ 完全同源. Python 用元组解包一行搞定四步:
            # 右边先算完再赋左边 — 天然避开"覆盖-丢失"
            prev, curr = None, start
            while curr is not end:                            # `is not` 比 `!=` 更贴 C++ 指针比较语义
                curr.next, prev, curr = prev, curr, curr.next
            return prev
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {ListNode} head
     * @param {number} k
     * @return {ListNode}
     */
    var reverseKGroup = function(head, k) {
        // JS 对象字面量当 dummy
        const dummy = { val: 0, next: head };
        let prev = dummy;

        // 内嵌 reverse — JS 没 private method 语法, closure 就是 helper
        const reverse = (start, end) => {
            let prv = null, curr = start;
            while (curr !== end) {
                const next = curr.next;
                curr.next = prv;
                prv = curr;
                curr = next;
            }
            return prv;
        };

        while (true) {
            let end = prev.next;
            for (let i = 0; i < k; i++) {
                if (!end) return dummy.next;
                end = end.next;
            }
            const start = prev.next;
            const newHead = reverse(start, end);
            prev.next = newHead;
            start.next = end;
            prev = start;
        }
    };
    ```

## Complexity

- **Time**: O(n) — 每节点被访问最多 2 次 (预扫 + 反转).
- **Space**: O(1) — dummy + 常数游标.

## 相关题目

- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题 (整链)
- [0024. Swap Nodes in Pairs](../0024-swap-nodes-in-pairs/README.md) — k=2 特例
- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题
- 0092\. Reverse Linked List II (待补) — 反转 `[left, right]` 区间, 半开区间思想同源
- 0234\. Palindrome Linked List (待补) — 快慢找中点 + 后半反转 + 比较
- [0143. Reorder List](../0143-reorder-list/README.md) — 快慢找中点 + 后半反转 + 双端 merge
- 0061\. Rotate List (待补) — 部分旋转, 指针 + 长度
