# 0146. LRU Cache / LRU 缓存

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Table, Linked List, Design, Doubly Linked List · 哈希表, 链表, 设计, 双向链表
    - **Link**: [LeetCode](https://leetcode.com/problems/lru-cache/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **O(1) get + O(1) put with eviction** → **hash map + doubly linked list**. Hash gives O(1) lookup; DLL gives O(1) move-to-front & pop-back. Front = MRU, back = LRU. Every access **removes then re-inserts at front**.
>
> **中文**: **get/put 都 O(1) 且需淘汰** → **哈希表 + 双向链表**. 哈希 O(1) 定位, 双向链表 O(1) 移动与删尾. **前=最近用, 后=最久未用**. 每次访问都**先删后插头**.
>
> *Template / 模版*: **Two structures collaborate** — 哈希索引 + 链表顺序. 一个负责"在哪", 一个负责"顺序".

## Problem

**EN**: 设计 `LRUCache(capacity)`:

- `get(key)` — 返值, 不存在返 -1. **命中算一次使用**.
- `put(key, value)` — 插或更新. 满了则淘汰**最久未使用**的.
- 两个操作**均须 O(1)**.

**中文**: LRU 淘汰策略缓存, get/put 都要 O(1).

## Key Insights

1. **🔑 灵魂: 为什么必须"哈希 + 双向链表"两种结构 / Why both hash and DLL?**

    单一结构做不到 O(1):

    | 单结构 | get O(1)? | put + 淘汰 O(1)? | 死在哪 |
    |---|---|---|---|
    | 哈希表 | ✅ | ❌ | 不知道谁最久未用 (无顺序) |
    | 双向链表 | ❌ (线性查) | ✅ (移头/删尾 O(1)) | 找 key 得扫 |
    | 数组 | ❌ (线性查) | ❌ (中间删 O(n)) | 全死 |
    | 单链表 | ❌ | ❌ (删除需前驱, O(n)) | 全死 |

    **组合**: 哈希 `key → 链表节点指针`, 链表**只管顺序**. → **get 用哈希找到节点**, **删/移** 用链表指针 O(1). 双方各干各的.

    > **"两个结构合作" 是 LRU 的灵魂**. 面试主动提: "one for lookup, one for order".

2. **🔑 为什么必须双向链表, 单向不行 / Why doubly, not singly?**

    删除任意节点 `n` 要**改前驱的 next**. 单链表得从头扫找前驱 → O(n). 双向链表 `n->prev->next = n->next` → **O(1)**.

    > **想 O(1) 中间删除, 必须双向**. `unordered_map` 存的是节点指针, 拿到 n 之后要能立刻断链.

3. **🔑 哨兵头尾 (dummy head/tail) — 简化边界 / Sentinel head+tail — no null checks**

    Yang v1 用 `head` + `tail` 两个 dummy 节点. 好处:

    - **加头**: `head->next = n; n->prev = head` — 不用判空.
    - **删尾**: `tail->prev` 就是 LRU, 不用判空.
    - **remove(n)**: `n->prev->next = n->next` 永远合法, 因为 `prev`/`next` 都不会是 null.

    → 代码短且**无 nullptr 分支**. 面试写 LRU **必用 sentinel**.

    > 也见 [0206 Reverse Linked List](../../02-linked-list/0206-reverse-linked-list/README.md), [0021 Merge Two Sorted](../../02-linked-list/0021-merge-two-sorted-lists/README.md) — dummy 是链表面试通用招.

4. **🔑 每次访问 = 删 + 插头 / Every access: remove then reinsert at front**

    ```cpp
    // get 命中
    remove(n); addFront(n);
    // put 更新
    n->val = value; remove(n); addFront(n);
    ```

    → 无论新旧, 只要"用过", 就**搬到最前**. 淘汰时永远淘汰 `tail->prev`.

    > **不要试图 in-place 更新位置** — 双向链表移动就是"断开 + 重插" 两步, 已 O(1), 别优化.

5. **🔑 淘汰时 map 也要 erase — 别忘 sync / Erase from map, not just list**

    Yang v1:

    ```cpp
    if (mp.size() == cap) {
        Node* lru = tail->prev;
        remove(lru);
        mp.erase(lru->key);   // ⚠️ 必须同步删 map
        delete lru;           // ⚠️ 手动 new 就要手动 delete
    }
    ```

    - 忘 `mp.erase` → 内存泄漏 + map 越涨越大.
    - 忘 `delete` → 内存泄漏 (LC 不检查但生产要).

    → 这是**为啥节点里存 key** (不只 value): 淘汰时**反查 key 去删 map**.

    > **易错点 top 1**: 忘同步 map. Top 2: 节点不存 key 反查不到.

6. **🔑 v2: STL `list` + `splice` — 少写一半代码 / STL list with splice**

    Yang v2:

    ```cpp
    list<pair<int,int>> lst;                              // front = MRU
    unordered_map<int, list<pair<int,int>>::iterator> mp;
    lst.splice(lst.begin(), lst, it->second);  // 把节点搬到最前 O(1)
    ```

    - **`std::list` = 双向链表**. `splice` = **常数时间**把节点从任意位置搬到任意位置 (**不动内存, 只改指针**).
    - **map 存 iterator** (不是节点指针) — `splice` **不使 iterator 失效**! 这是 `list` 的关键保证, `vector` 就没有.
    - `pop_back` / `emplace_front` 直接用 STL, 不用手写 DLL.

    → **代码短 40%, 无 new/delete, 无 sentinel**. 面试推荐写这版, 除非面试官要求"手写 DLL".

    > **`splice` 是 `list` 的杀手锏**. 常见于 LRU、任务队列重排、undo/redo 栈.

7. **🔑 iterator 不失效是关键保证 / Iterator stability enables the trick**

    `std::list` 的所有操作 (insert/erase/splice) **只让被删的 iterator 失效**, 其他都活着. 所以 map 里存的 iterator 移动后依然指对.

    - **`vector`** 就不行 — 任意 insert 可能 realloc, 所有 iterator 崩.
    - **`deque`** 也不行 — insert 中间可能挪 block.

    → **数据结构选型时**看 "operation stability guarantees", 不只是"是否支持某操作".

8. **🔑 复杂度 / Complexity**

    - **Time**: `get` / `put` 都 **O(1)** 摊销.
    - **Space**: O(capacity) — hash + list 各存 capacity 项.

## Solution

=== "C++ (v2: STL list + splice, 推荐)"
    ```cpp
    class LRUCache {
        list<pair<int, int>> lst;                              // front = MRU, back = LRU
        unordered_map<int, list<pair<int, int>>::iterator> mp; // key → 节点位置
        int cap;
    public:
        LRUCache(int capacity) : cap(capacity) {}

        int get(int key) {
            auto it = mp.find(key);
            if (it == mp.end()) return -1;
            lst.splice(lst.begin(), lst, it->second);          // 搬到最前, O(1)
            return it->second->second;
        }

        void put(int key, int value) {
            auto it = mp.find(key);
            if (it != mp.end()) {                              // 命中: 更新 + 提前
                it->second->second = value;
                lst.splice(lst.begin(), lst, it->second);
                return;
            }
            if ((int)mp.size() == cap) {                       // 满: 淘汰尾
                mp.erase(lst.back().first);                    // 节点存 key 才能反查
                lst.pop_back();
            }
            lst.emplace_front(key, value);
            mp[key] = lst.begin();
        }
    };
    ```

=== "C++ (v1: 手写 DLL + sentinel, 面试展示 DS 功底)"
    ```cpp
    class LRUCache {
        struct Node {
            int key, val;
            Node *prev, *next;
            Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
        };
        unordered_map<int, Node*> mp;
        Node *head, *tail;                                     // sentinel
        int cap;

        void remove(Node* n) {                                 // 摘链
            n->prev->next = n->next;
            n->next->prev = n->prev;
        }
        void addFront(Node* n) {                               // 插头
            n->next = head->next;
            n->prev = head;
            head->next->prev = n;
            head->next = n;
        }
    public:
        LRUCache(int capacity) : cap(capacity) {
            head = new Node(0, 0);
            tail = new Node(0, 0);
            head->next = tail;
            tail->prev = head;
        }

        int get(int key) {
            auto it = mp.find(key);
            if (it == mp.end()) return -1;
            Node* n = it->second;
            remove(n); addFront(n);                            // 提到最前
            return n->val;
        }

        void put(int key, int value) {
            auto it = mp.find(key);
            if (it != mp.end()) {
                Node* n = it->second;
                n->val = value;
                remove(n); addFront(n);
                return;
            }
            if ((int)mp.size() == cap) {
                Node* lru = tail->prev;
                remove(lru);
                mp.erase(lru->key);                            // 同步删 map
                delete lru;
            }
            Node* n = new Node(key, value);
            addFront(n);
            mp[key] = n;
        }
    };
    ```

=== "Python (OrderedDict 一行秒杀)"
    ```python
    from collections import OrderedDict

    class LRUCache:
        # OrderedDict = dict + 双向链表 (CPython 实现).
        # 天生支持 O(1) move_to_end / popitem(last=False).
        # → Python 面试 LRU 用它, 但要主动说"知道底层是 hash+DLL".

        def __init__(self, capacity: int):
            self.cap = capacity
            self.cache: OrderedDict[int, int] = OrderedDict()

        def get(self, key: int) -> int:
            if key not in self.cache:
                return -1
            # move_to_end: 把 key 移到末尾 (定义"末尾 = 最近用")
            # 等价 v1 的 remove + addFront, 但一行 O(1)
            self.cache.move_to_end(key)
            return self.cache[key]

        def put(self, key: int, value: int) -> None:
            if key in self.cache:
                self.cache.move_to_end(key)                    # 命中: 提前
            self.cache[key] = value
            if len(self.cache) > self.cap:
                # popitem(last=False): 弹**最前** (最久未用), O(1)
                # 若不给 last=False, 默认弹末尾 (LIFO), 那就是 stack 语义
                self.cache.popitem(last=False)
    ```

=== "JavaScript (Map 保序 + 删再插)"
    ```javascript
    // JS Map 保持插入顺序. 想把 key 提到"最新"?
    // → 只能 delete + set (Map 没有 move_to_end).
    // 两次操作各 O(1), 摊销仍是 O(1).

    var LRUCache = function(capacity) {
        this.cap = capacity;
        this.cache = new Map();          // Map = 保序 hash, 底层近似 hash + DLL
    };

    LRUCache.prototype.get = function(key) {
        if (!this.cache.has(key)) return -1;
        const val = this.cache.get(key);
        this.cache.delete(key);           // 删
        this.cache.set(key, val);         // 再插 → 现在它最新
        return val;
    };

    LRUCache.prototype.put = function(key, value) {
        if (this.cache.has(key)) {
            this.cache.delete(key);       // 命中: 先删好让 set 变成"最新"
        } else if (this.cache.size >= this.cap) {
            // Map.keys() 返 iterator 按插入顺序, .next().value = 最旧
            const lru = this.cache.keys().next().value;
            this.cache.delete(lru);
        }
        this.cache.set(key, value);
    };
    ```

## Complexity

| 操作 | Time | Space |
|---|---|---|
| `get` / `put` | **O(1)** 摊销 | O(capacity) |

## 相关题目

- [0706. Design HashMap](../0706-design-hashmap/README.md) — 哈希表设计基础, 姐妹题
- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — 另一种 lookup DS 设计
- [0707. Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) — 手写链表基本功
- [0206. Reverse Linked List](../../02-linked-list/0206-reverse-linked-list/README.md) — 链表操作 + dummy 母题
- [0021. Merge Two Sorted Lists](../../02-linked-list/0021-merge-two-sorted-lists/README.md) — sentinel 应用
- 0460\. LFU Cache (待补) — 升级版, 加频次统计, hash + 频次桶 + 双向链表
- 0432\. All O`one Data Structure (待补) — hash + 双向链表桶
- 0355\. Design Twitter (待补) — 多结构组合设计
