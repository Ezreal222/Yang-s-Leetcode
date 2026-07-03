# 0707. Design Linked List / 设计链表

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Linked List, Design, Dummy Head · 链表, 设计, 虚拟头节点
    - **Link**: [LeetCode](https://leetcode.com/problems/design-linked-list/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Implement singly linked list with get/insert/delete** → **dummy head + size counter**; find `prev` (dummy walks `index` steps) then mutate; `addAtHead` / `addAtTail` **just forward to `addAtIndex(0, ...)` / `addAtIndex(size, ...)`** — one logic to test.
>
> **中文**: **手写链表 (get/insert/delete)** → **dummy + size**; 每次找 prev (dummy 走 index 步) 再动指针; **addAtHead / addAtTail 直接转 addAtIndex** — 三合一, 只测一份逻辑.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 实现 `MyLinkedList`, 支持 `get(index)`, `addAtHead(val)`, `addAtTail(val)`, `addAtIndex(index, val)`, `deleteAtIndex(index)`. 越界返 `-1` 或忽略.

**中文**: 从零撸单链表, 5 个方法.

## Key Insights

1. **🔑 Design 题的核心思维: 抽公共子操作 / Design principle: extract shared subroutine**

    Yang 的**关键设计**:

    ```cpp
    void addAtHead(int val) { addAtIndex(0, val); }
    void addAtTail(int val) { addAtIndex(size, val); }
    ```

    三个 add API **只有一份真正的逻辑** — `addAtIndex`. 头 = 索引 0, 尾 = 索引 size. **DRY** (Don't Repeat Yourself). 一处对了全对, 一处错了全错 (至少调试集中).

    > **面试的 design 题**看你会不会**找抽象层次**. 三个 add 各写一遍 = 新手; 转发一处 = 有设计思维.

2. **🔑 dummy head + size 是 API 型链表的标配 / Dummy head + size counter: the standard toolkit**

    | 好处 | 用在哪 |
    |---|---|
    | dummy → 头节点删/插不用特判 (见 [0203](../0203-remove-linked-list-elements/README.md)) | `addAtIndex`, `deleteAtIndex` |
    | size → 越界判 O(1) | 所有 API 的边界检查 |
    | size → tail insert 不用扫到底 (走 size 步就是尾) | `addAtTail` (via `addAtIndex(size, ...)`) |

    > **没 size** 的话, 尾插要**每次扫全链** → O(n²) 累积. 加一个 `int size` 计数器就是**空间换时间**.

3. **🔑 前驱定位模式: prev = dummy + index 步 / Predecessor pattern**

    要在**位置 index 插入 / 删除**, 都得**站在 index - 1**:

    ```cpp
    Node* prev = dummy;
    for (int i = 0; i < index; i++) prev = prev->next;
    // 现在 prev 指向"index 的前驱"
    ```

    - **index = 0** → prev = dummy (不进循环), 插到 dummy 后 = 插头
    - **index = size** → prev = 最后一个真节点, 插它后 = 插尾

    > **单链表所有修改都从前驱出手** — dummy 让"头前也有前驱" 这一契约成立, 三种情况 (头 / 中 / 尾) 统一.

4. **🔑 边界判定的三种口径 / Three flavors of bounds check**

    | API | 合法范围 | 越界 |
    |---|---|---|
    | `get(index)` | `[0, size - 1]` (读现有) | 返 -1 |
    | `deleteAtIndex(index)` | `[0, size - 1]` (删现有) | 忽略 |
    | `addAtIndex(index, val)` | **`[0, size]`** (插入允许尾后) | 忽略 |

    > **插入允许 `index == size`** — 因为"最后一个位置的后面" 是合法插入点. 删/读没这待遇. 记住这个**差 1 的区别**.

5. **🔑 插入的"新节点 4 步" / Insert dance in 4 lines**

    ```cpp
    Node* newNode = new Node(val);
    newNode->next = prev->next;      // 1. 新节点先接上"未来的下家"
    prev->next = newNode;            // 2. 前驱指向新节点
    size++;                          // 3. 记账
    ```

    **顺序不能反** — 若先 `prev->next = newNode`, 就丢失了原来的 `prev->next` 指针. **必须先接下家再改前驱**.

    > 双链表还得多两步 (`newNode->prev` / 下家的 `prev`), 但顺序原则一样: **先接后改**.

6. **🔑 析构缺失是**唯一**的工程隐患 / Destructor missing**

    Yang 的 `MyLinkedList` **没有析构函数** → LeetCode 不检测, 但工程上会**泄漏所有节点** + dummy. 加一个:

    ```cpp
    ~MyLinkedList() {
        Node* cur = dummy;
        while (cur) { Node* nx = cur->next; delete cur; cur = nx; }
    }
    ```

    > 面试**主动提**"应该加析构" 加分. Python / JS 的 GC 自动清理.

7. **复杂度 / Complexity**

    | 方法 | Time | Space |
    |---|---|---|
    | `get` | O(index) | O(1) |
    | `addAtHead` | **O(1)** (via addAtIndex(0)) | O(1) 每次 |
    | `addAtTail` | O(size) (仍要走到 size, size 只加快了 bounds check) | O(1) 每次 |
    | `addAtIndex` | O(index) | O(1) 每次 |
    | `deleteAtIndex` | O(index) | O(1) |

    > **想 tail insert O(1)?** → 加**tail 指针** (双端单链表), 或改**双向链表**. 面试的进阶回答.

## Solution

=== "C++"
    ```cpp
    class MyLinkedList {
        struct Node {
            int val;
            Node* next;
            Node(int v) : val(v), next(nullptr) {}
        };
        Node* dummy;
        int size;
    public:
        MyLinkedList() {
            dummy = new Node(0);
            size = 0;
        }

        int get(int index) {
            if (index < 0 || index > size - 1) return -1;
            Node* cur = dummy->next;
            for (int i = 0; i < index; i++) cur = cur->next;
            return cur->val;
        }

        void addAtHead(int val) { addAtIndex(0, val); }              // 转发
        void addAtTail(int val) { addAtIndex(size, val); }           // 转发

        void addAtIndex(int index, int val) {
            if (index < 0 || index > size) return;                   // 插入: [0, size]
            Node* prev = dummy;
            for (int i = 0; i < index; i++) prev = prev->next;       // 找前驱
            Node* newNode = new Node(val);
            newNode->next = prev->next;                              // 先接下家
            prev->next = newNode;                                    // 再改前驱
            size++;
        }

        void deleteAtIndex(int index) {
            if (index < 0 || index > size - 1) return;               // 删除: [0, size - 1]
            Node* prev = dummy;
            for (int i = 0; i < index; i++) prev = prev->next;
            Node* toDel = prev->next;
            prev->next = toDel->next;
            delete toDel;                                            // 工程素养: 手动释放
            size--;
        }

        // 面试加分项: 显式析构
        ~MyLinkedList() {
            Node* cur = dummy;
            while (cur) { Node* nx = cur->next; delete cur; cur = nx; }
        }
    };
    ```

=== "Python"
    ```python
    class MyLinkedList:
        # Python 用内嵌类, 或直接 (val, next) 元组. 这里保持类结构对齐 C++ 教学
        class _Node:
            __slots__ = ('val', 'next')     # __slots__ 省内存, 禁止动态属性 — 类似 C++ 的 struct 只有固定字段
            def __init__(self, val=0, nxt=None):
                self.val = val
                self.next = nxt

        def __init__(self):
            self.dummy = self._Node(0)
            self.size = 0

        def get(self, index: int) -> int:
            if index < 0 or index >= self.size: return -1
            cur = self.dummy.next
            # Python 没 for(;;)  — for _ in range(index) 就是 C++ 等价
            for _ in range(index): cur = cur.next
            return cur.val

        def addAtHead(self, val: int) -> None: self.addAtIndex(0, val)
        def addAtTail(self, val: int) -> None: self.addAtIndex(self.size, val)

        def addAtIndex(self, index: int, val: int) -> None:
            if index < 0 or index > self.size: return
            prev = self.dummy
            for _ in range(index): prev = prev.next
            # 新节点直接构造时接下家, 一行搞定. Python 无需先分配再改字段
            prev.next = self._Node(val, prev.next)
            self.size += 1

        def deleteAtIndex(self, index: int) -> None:
            if index < 0 or index >= self.size: return
            prev = self.dummy
            for _ in range(index): prev = prev.next
            # GC 帮我们清理被删节点, 不用手 delete
            prev.next = prev.next.next
            self.size -= 1
    ```

=== "JavaScript"
    ```javascript
    var MyLinkedList = function() {
        // JS 没内嵌类语法糖. 用普通对象作节点最省事 — 属性 val, next
        // class Node {} 也行, 但对这题来说是 overkill
        this.dummy = {val: 0, next: null};
        this.size = 0;
    };

    /**
     * @param {number} index
     * @return {number}
     */
    MyLinkedList.prototype.get = function(index) {
        if (index < 0 || index >= this.size) return -1;
        let cur = this.dummy.next;
        for (let i = 0; i < index; i++) cur = cur.next;
        return cur.val;
    };

    MyLinkedList.prototype.addAtHead = function(val) { this.addAtIndex(0, val); };
    MyLinkedList.prototype.addAtTail = function(val) { this.addAtIndex(this.size, val); };

    MyLinkedList.prototype.addAtIndex = function(index, val) {
        if (index < 0 || index > this.size) return;
        let prev = this.dummy;
        for (let i = 0; i < index; i++) prev = prev.next;
        // 对象字面量直接接下家, 简洁
        prev.next = {val, next: prev.next};
        this.size++;
    };

    MyLinkedList.prototype.deleteAtIndex = function(index) {
        if (index < 0 || index >= this.size) return;
        let prev = this.dummy;
        for (let i = 0; i < index; i++) prev = prev.next;
        prev.next = prev.next.next;         // GC 自动清理
        this.size--;
    };
    ```

## Complexity

| API | Time | Space |
|---|---|---|
| `get(index)` | O(index) | O(1) |
| `addAtIndex(index, val)` | O(index) | O(1) 每次 |
| `addAtHead(val)` | **O(1)** | O(1) |
| `addAtTail(val)` | O(size) | O(1) |
| `deleteAtIndex(index)` | O(index) | O(1) |

## 相关题目

- [0203. Remove Linked List Elements](../0203-remove-linked-list-elements/README.md) — dummy head 母题
- [0206. Reverse Linked List](../0206-reverse-linked-list/README.md) — 迭代三指针母题
- 0146\. LRU Cache (待补) — 设计题, 双向链表 + 哈希, 更复杂的 API 设计
- 0460\. LFU Cache (待补) — 双哈希 + 频次链表
- 0432\. All O`one Data Structure (待补) — 频次桶 + 双向链表, API 设计考察
- 0705\. Design HashSet (待补) — 设计题, 链地址法用链表
- 0706\. Design HashMap (待补) — 同上
