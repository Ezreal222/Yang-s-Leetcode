# 0138. Copy List with Random Pointer / 复制带随机指针的链表

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Hash Map, DFS, Interleaving · 链表, 哈希表, 深度优先, 穿插技巧
    - **Link**: [LeetCode](https://leetcode.com/problems/copy-list-with-random-pointer/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☑ ☐ ☐

## TL;DR / 一句话

> **EN**: **Deep-copy a linked list where each node has `next` + `random`** → 3 options: **(v1) DFS + map** (same as [0133](../../12-graph/0133-clone-graph/README.md)), **(v2) two-pass map** (build all clones, then wire), **(v3) interleave trick** (weave clones into original, wire, then split — **O(1) extra space**).
>
> **中文**: **带 random 指针的链表深拷贝** → 三种: **v1 DFS + map** (跟克隆图同款), **v2 两遍扫 + map** (先建全部克隆再连指针), **v3 穿插原地** (克隆插原链, 连 random, 再拆分 — **O(1) 额外空间**).
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 单链表, 每节点除 `next` 外还有一个 `random` 指针 (可能指向链表**任意节点或 null**). 深拷贝返回新链头.

**中文**: 每节点带 random 指针的链表深拷贝.

## Key Insights

1. **🔑 核心难点: random 指针可能"回头指" / The hard part: random can point backward**

    朴素**一遍扫 + 建 clone + 连 next**是没问题的 (单向 next). 但 `random` 可能指向**尚未创建的后续节点** — 或已创建的**任何**节点. 无脑一遍扫会遇 null 引用.

    **两种破法**:

    - **map 记忆化**: `原 → 克隆` 映射. 无论 random 指向哪, 查表就知道对应 clone.
    - **原地穿插**: 让 clone 就在**原节点旁边**, `原.random.next` 就是"random 目标的克隆" — 不需要 map.

2. **🔑 v1 (DFS + map): 跟 [0133 Clone Graph](../../12-graph/0133-clone-graph/README.md) 同款 / Same as Clone Graph**

    把链表**当图**处理: 每节点两条出边 (`next` + `random`). 用**图 DFS + visited map**打破环:

    ```cpp
    if (visited[node]) return visited[node];      // 已建 → 返
    clone = new Node(val)
    visited[node] = clone                         // 先存映射 (关键!)
    clone->next   = dfs(node->next)               // 递归
    clone->random = dfs(node->random)
    ```

    > **"链表 + random = 复杂图"**, 图深拷贝的 map 模板直接搬. 顺序: **建 → 存映射 → 递归** (跟 [0133](../../12-graph/0133-clone-graph/README.md) 灵魂一致).

3. **🔑 v2 (两遍扫 + map): 更工业, 无递归 / Two-pass: industrial-grade, no recursion**

    ```
    Pass 1: 沿 next 走一遍, 每个原节点造一个 clone, 存 map[原] = clone
    Pass 2: 再走一遍, 用 map 连 next 和 random
    ```

    Yang 的这行**特别巧**:

    ```cpp
    mp[p]->next = mp[p->next];      // mp[null] 默认返 nullptr (unordered_map operator[])
    mp[p]->random = mp[p->random];
    ```

    - `unordered_map<Node*, Node*>::operator[]` 缺 key 时**默认构造** value = `nullptr` (指针默认零值). 所以 `mp[null] == nullptr` — **不用 if 判 null**!
    - **副作用**: 会插入一个 `null → nullptr` 条目. 无害.

    > **"缺 key 默认零" 的隐式契约** 让代码短一半. 但**懂它是"副作用式" 建 key** — 有时不想插得用 `.count()` 判.

4. **🔑 v3 (穿插原地): 灵魂 trick, **O(1) 额外空间** / The genius trick: O(1) extra space**

    分三步:

    **Step 1**: 每个原节点旁边插一个 clone.

    ```
    Before: A → B → C
    After:  A → A' → B → B' → C → C'
    ```

    实现: `for cur = head; cur; cur = cur->next->next` 每一格建 clone 插在后面.

    **Step 2**: 用**"原节点的 random 后面就是它的 clone"** 事实连 random.

    ```
    A.random == B  →  A'.random == B' == A.random.next
    ```

    实现: `cur->next->random = cur->random ? cur->random->next : nullptr`.

    **Step 3**: 拆两条链, 恢复原链, 拿出 clone 链.

    ```
    Before split: A → A' → B → B' → C → C'
    After split:  A → B → C          (原, 复原)
                  A' → B' → C'       (新)
    ```

    实现: 交替修 next 指针.

    > **O(1) 额外空间**! 唯一"空间" 是最终 clone 链本身 (必然存在, 不算 extra). **面试问"能不能 O(1)"** 就掏这个.

5. **🔑 三版对比 / Comparison**

    | | v1 DFS | v2 两遍扫 | **v3 穿插** |
    |---|---|---|---|
    | Time | O(n) | O(n) | O(n) |
    | **Extra Space** | O(n) map + O(n) 栈 | O(n) map | **O(1)** |
    | 代码量 | 短 | 中 | 长 (3 步) |
    | 递归 | 是 | 否 | 否 |
    | **最易想到** | ✅ | ✅ | ❌ |
    | **面试最秀** | ❌ | 中 | ✅✅ |

    > **首选 v2** (工业友好, 无 stack overflow 风险), **v3 是 O(1) 空间的答案** (面试 follow-up).

6. **🔑 map 用**指针作 key** 而不是 val / Use pointer as key, not val**

    `unordered_map<Node*, Node*>` — key 是**原节点指针**. 为啥不用 `int val`?

    - **两个原节点可能同 val**. 用 val 会撞 key.
    - **指针唯一标识节点身份**, 是"这个具体节点" 的天然 ID.

    > **"key 必须区分不同实体"** — 用 val 时先问"允不允许重复". 这题 val 允许重复 → 必须用指针.

7. **复杂度 / Complexity**

    - **Time**: O(n) 三版都是.
    - **Space**: v1/v2 O(n), **v3 O(1) extra**.

## Interview Walkthrough (Speak Out Loud)

*What I'd literally say while pair-programming this with an interviewer. 5-8 min out loud.*

### 1. Clarify (30s)

> "So I need to **deep-copy a linked list** where each node has a `next` pointer *and* a `random` pointer that can point to **any node in the list — or `null`**. The output should be a completely independent copy: same values, same structure, but zero shared nodes with the original. A few things to confirm:"

- "Can `random` point to **any node, including itself or a node before the current one**?" *(yes — that's the whole difficulty.)*
- "Are node values **unique**? Not that I'd use them for identity, but just checking." *(usually not guaranteed. I'll use pointer identity, not value.)*
- "How's the list terminated — `null` for both `next` and `random`?" *(yes.)*
- "Any **memory constraint**? Because there's a well-known O(1) extra-space trick if you want to see it." *(if they say 'try to optimize', I know they want v3.)*

### 2. Brainstorm approaches (1 min)

> "Three approaches worth naming.
>
> **Approach 1 — DFS + hash map**: treat the list as a graph (each node has two outgoing edges: `next` and `random`) and do a standard clone-graph DFS with a `visited` map from original → clone. O(n) time, O(n) space plus recursion stack. This is basically Clone Graph on a linked list.
>
> **Approach 2 — two-pass with a hash map**: first pass creates all clone nodes and stores `orig → clone` mapping (walking only `next`). Second pass wires up `next` and `random` on each clone using the map. O(n) time, O(n) space, no recursion — cleanest for production code.
>
> **Approach 3 — interleaving trick** for **O(1) extra space**: temporarily weave each clone right after its original (`A → A' → B → B' → C → C'`), so the clone of `X.random` is always `X.random.next`. Wire the randoms, then split the two lists apart. It's a beautiful pointer dance but denser to explain.
>
> I'd default to **approach 2** — it's the most maintainable. If they want O(1) space, I'll walk through **approach 3**."

### 3. Sketch the algorithm before coding (1 min)

> "Two design choices worth flagging before writing v2:
>
> 1. **Map key must be the pointer, not the value.** Values can repeat, but each node is a distinct entity in the copy. `unordered_map<Node*, Node*>` — pointer to pointer.
> 2. **Clean up the null case with a bonus:** In C++, `unordered_map<Node*, Node*>::operator[]` **returns the default-constructed value if the key is missing**, which for pointer types is `nullptr`. So `mp[nullptr]` is automatically `nullptr` — I can write `mp[p]->next = mp[p->next]` without a separate null check for `p->next`.
>
> The plan:
>
> - **Pass 1**: walk with `p = head`, create `new Node(p->val)` for each `p`, store `mp[p] = new_clone`.
> - **Pass 2**: walk again, set `mp[p]->next = mp[p->next]` and `mp[p]->random = mp[p->random]`.
> - Return `mp[head]`."

### 4. Code while narrating (2 min)

```cpp
Node* copyRandomList(Node* head) {
    if (!head) return nullptr;
    unordered_map<Node*, Node*> mp;
    // Pass 1: create all clones (only follow next)
    for (Node* p = head; p; p = p->next)
        mp[p] = new Node(p->val);
    // Pass 2: wire next and random using the map
    for (Node* p = head; p; p = p->next) {
        mp[p]->next   = mp[p->next];      // mp[nullptr] returns nullptr automatically
        mp[p]->random = mp[p->random];
    }
    return mp[head];
}
```

> "About 10 lines. Two clean sweeps — the first handles allocation, the second handles pointer wiring. The `mp[nullptr] == nullptr` trick removes what would otherwise be four `if` guards."

### 5. Trace an example (1 min)

> "Let me trace with a tiny list: three nodes A, B, C where `A.random = C`, `B.random = A`, `C.random = null`.
>
> **Pass 1**: mp becomes `{A: A', B: B', C: C'}`.
>
> **Pass 2**:
> - At A: `A'.next = mp[B] = B'`; `A'.random = mp[C] = C'`. ✓
> - At B: `B'.next = mp[C] = C'`; `B'.random = mp[A] = A'`. ✓
> - At C: `C'.next = mp[null] = null` (that's the trick working); `C'.random = mp[null] = null`. ✓
>
> Return `mp[A] = A'`. Fully independent copy — walking A' gives A'→B'→C' with random pointers into the new list. No pointer accidentally references the original."

### 6. Complexity (20s)

> "**Time O(n)** — two passes, hash lookups O(1) amortized. **Space O(n)** — the map holds n entries.
>
> If the interviewer asks for O(1) extra space, approach 3 gives that at the cost of temporarily mutating the input list."

### 7. Follow-up: O(1) space via interleaving (1 min)

> "Quick sketch of the O(1) trick if they want it:
>
> **Step 1** — Weave clones into the original: for each node `X`, create `X'` and insert right after: `X → X' → next(X) → …`. Result: `A → A' → B → B' → C → C'`.
>
> **Step 2** — Wire randoms using the invariant '`X'` sits right after `X`, so the clone of any `random target` is at `target.next`': for each original `X`, `X'.random = X.random ? X.random.next : nullptr`.
>
> **Step 3** — Un-weave: separate the two interleaved lists, restoring the original and extracting the copy.
>
> **O(1) extra space** — no map, just three linear passes over the (temporarily doubled) list. The insight is that interleaving encodes the mapping *in the structure itself*."

> "Related problem worth mentioning: [Clone Graph (0133)](../../12-graph/0133-clone-graph/README.md) is approach 1 taken to its natural home — an arbitrary graph. Same map-based DFS. Any follow-ups you'd like me to code?"

## Solution

=== "C++"

    **v1: DFS + hash map (跟 0133 同款)**

    ```cpp
    class Solution {
    public:
        Node* copyRandomList(Node* head) {
            if (!head) return nullptr;
            unordered_map<Node*, Node*> visited;
            return dfs(head, visited);
        }
    private:
        Node* dfs(Node* node, unordered_map<Node*, Node*>& visited) {
            if (!node) return nullptr;
            if (visited.count(node)) return visited[node];
            Node* copy = new Node(node->val);
            visited[node] = copy;                                    // 先存映射
            copy->next   = dfs(node->next,   visited);
            copy->random = dfs(node->random, visited);
            return copy;
        }
    };
    ```

    **v2: 两遍扫 + hash map (工业首选)**

    ```cpp
    class Solution {
    public:
        Node* copyRandomList(Node* head) {
            if (!head) return nullptr;
            unordered_map<Node*, Node*> mp;
            // Pass 1: 造所有 clone
            for (Node* p = head; p; p = p->next) mp[p] = new Node(p->val);
            // Pass 2: 连 next 和 random. mp[null] 默认 nullptr, 无需判
            for (Node* p = head; p; p = p->next) {
                mp[p]->next   = mp[p->next];
                mp[p]->random = mp[p->random];
            }
            return mp[head];
        }
    };
    ```

    **v3: 穿插原地 (O(1) extra space)**

    ```cpp
    class Solution {
    public:
        Node* copyRandomList(Node* head) {
            if (!head) return nullptr;
            // Step 1: A → B → C  ⇒  A → A' → B → B' → C → C'
            for (Node* cur = head; cur; cur = cur->next->next) {
                Node* copy = new Node(cur->val);
                copy->next = cur->next;
                cur->next = copy;
            }
            // Step 2: 连 random. A' 就在 A 旁边, 所以 A.random.next = A' 的 target
            for (Node* cur = head; cur; cur = cur->next->next) {
                if (cur->random) cur->next->random = cur->random->next;
            }
            // Step 3: 拆两条链. 恢复原, 生成新
            Node* newHead = head->next;
            for (Node* cur = head; cur; cur = cur->next) {
                Node* copy = cur->next;
                cur->next = copy->next;                              // 恢复原
                copy->next = copy->next ? copy->next->next : nullptr; // 连新
            }
            return newHead;
        }
    };
    ```

=== "Python"
    ```python
    class Solution:
        # v2 首选 — 无 stack 隐患, 代码干净
        def copyRandomList(self, head: 'Node') -> 'Node':
            if not head: return None
            # dict 缺 key 时 .get(k, default) 兜底. 也可用 defaultdict + 手动 lambda
            # 这里用 dict.get(k, None) 语义等价 C++ unordered_map operator[]=null
            mp = {}
            p = head
            while p:
                mp[p] = Node(p.val)
                p = p.next
            p = head
            while p:
                mp[p].next   = mp.get(p.next)       # get 缺 key 返 None
                mp[p].random = mp.get(p.random)
                p = p.next
            return mp[head]
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {_Node} head
     * @return {_Node}
     */
    var copyRandomList = function(head) {
        // v2. JS 的 Map 缺 key 返 undefined, 我们要 null (跟原链 null 语义一致)
        if (!head) return null;
        const mp = new Map();
        for (let p = head; p; p = p.next) mp.set(p, new _Node(p.val));
        for (let p = head; p; p = p.next) {
            // mp.get(null) 是 undefined, 得手动兜 null (JS 的 undefined 跟 null 语义弱区分)
            mp.get(p).next   = mp.get(p.next)   || null;
            mp.get(p).random = mp.get(p.random) || null;
        }
        return mp.get(head);
    };
    ```

## Complexity

| Version | Time | Extra Space |
|---|---|---|
| v1 DFS + map | O(n) | O(n) map + O(n) 栈 |
| v2 两遍扫 + map | O(n) | O(n) map |
| **v3 穿插原地** | O(n) | **O(1)** |

## 相关题目

- [0133. Clone Graph](../../12-graph/0133-clone-graph/README.md) — 图深拷贝, v1 同款 map 模板
- [0430. Flatten a Multilevel Doubly Linked List](../0430-flatten-a-multilevel-doubly-linked-list/README.md) — 多级 DLL 拉平
- [0426. Convert BST to Sorted DLL](../0426-convert-bst-to-sorted-doubly-linked-list/README.md) — BST 转 DLL
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 反转母题
- [0025. Reverse Nodes in k-Group](../0025-reverse-nodes-in-k-group/README.md) — 半开区间反转
- 0116\. Populating Next Right Pointers in Each Node (待补) — 树的 next 指针填充
- 0117\. Populating Next Right Pointers II (待补) — 非完美二叉树版
- 0287\. Find the Duplicate Number (待补) — Floyd 判环用在数组
- 1490\. Clone N-ary Tree (待补) — N 叉树深拷贝
