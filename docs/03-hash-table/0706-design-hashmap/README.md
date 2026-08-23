# 0706. Design HashMap / 设计哈希映射

!!! info "Meta"
    - **Difficulty**: Easy
    - **Tags**: Hash Table, Design, Separate Chaining · 哈希表, 设计, 链地址法
    - **Link**: [LeetCode](https://leetcode.com/problems/design-hashmap/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **Build a HashMap from scratch** → **separate chaining**: fixed-size `vector<list<pair<int, int>>>` (buckets), hash = `key % PRIME_SIZE`. On collision, linear-scan the bucket's list. DRY `find` helper returns an iterator shared by `put/get/remove`.
>
> **中文**: **从零实现 HashMap** → **链地址法**: 定长 `vector<list<pair>>` 桶数组, hash = `key % 质数`. 冲突时链表内查 key. **`find` helper 返 iterator** 供 put/get/remove 共用.
>
> *Template / 模版*: **Structure observation → Algorithm choice → Key operation** (结构观察 → 算法决策 → 关键操作).

## Problem

**EN**: 实现 `MyHashMap`, 支持:

- `put(key, value)` — 插入或更新.
- `get(key)` — 返值, 不存在返 -1.
- `remove(key)` — 删除.

不用现成的库.

**中文**: 从头实现 HashMap.

## Key Insights

1. **🔑 灵魂: 冲突处理 = 链地址法 (separate chaining) / Collision resolution: chaining**

    HashMap 的核心难点是**冲突** — 不同 key `hash()` 到同一 index. 两大策略:

    | 策略 | 做法 | 优点 | 缺点 |
    |---|---|---|---|
    | **链地址法** (chaining, Yang) | 每 bucket 是**链表**, 冲突就往后接 | 实现简单, 扩容灵活 | 内存不连续 (cache miss 多) |
    | 开放寻址 (open addressing) | 冲突时探测下一空 slot (线性/二次/双哈希) | 缓存友好 | 需要标记删除, load factor 敏感 |

    Yang 用 **chaining** — 教学最清晰. Java `HashMap` 也用 chaining (JDK 8+ 转红黑树).

    > **面试写 HashMap 通常选 chaining** — 短且不容易错. 若追问"缓存性能" 再提开放寻址.

2. **🔑 `SIZE = 769` 质数减冲突 / Prime bucket count minimizes collisions**

    - `769` 是**质数**. `key % 769` 分布比 `key % 1000` 更均匀 (质数模不共享因子).
    - 常见质数选择: 17, 79, 769, 3079, 12289, 49157 — 都是"接近 2 的幂但不是 2 的幂".
    - 用**2 的幂** (如 1024) 反而**差**: 因为很多 key 分布带模式 (如都是偶数), 低位重复.

    > **"hash size 用质数" 是数据结构的传统智慧**. Yang 选 769 = 3 × 256 + 1, 接近 2^10.

3. **🔑 `list<pair<int, int>>` 而非 `vector<pair>` / Linked list for O(1) mid-erase**

    ```cpp
    vector<list<pair<int, int>>> table;
    ```

    为啥用 **`list`** (双向链表) 而非 `vector`?

    - **`remove(key)`** 要**从中间删除** — `list::erase(it)` **O(1)**, `vector::erase(it)` O(n).
    - **代价**: list 每节点额外 16 bytes 指针 + 内存不连续.
    - **权衡**: 若 bucket 通常很短 (< 10 项), vector 反而更快 (cache). 教学场景**list 清晰**.

    > **数据结构选型看主要操作**. Middle-erase 频繁 → list; sequential access 主导 → vector.

4. **🔑 DRY `find` helper 返 iterator / DRY helper returns iterator**

    Yang 的巧:

    ```cpp
    list<pair<int, int>>::iterator find(int key) {
        auto& bucket = table[hash(key)];
        for (auto it = bucket.begin(); it != bucket.end(); it++) {
            if (it->first == key) return it;
        }
        return bucket.end();
    }
    ```

    put / get / remove **都用它**:

    - **`get`**: 返 iterator, 读 `it->second`.
    - **`put`**: 若 iter 有效, 更新 `it->second`; 否则 emplace_back.
    - **`remove`**: 若 iter 有效, `bucket.erase(it)`.

    → **单一 helper 覆盖三个 API 的搜索**, 避免重复三份线性扫代码.

    > **DRY 思维**: 找出**共同子操作**, 抽成单一函数. 跟 [0707 Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) (`addAtHead/Tail` 转 `addAtIndex`) 和 [0703](../../06-stack-queue/0703-kth-largest-element-in-a-stream/README.md) (构造函数复用 `add`) 同款设计品味.

5. **🔑 返 iterator 而非 pair — 保留删除能力 / Return iterator, not pair**

    若 `find` 只返 pair 值, remove 得**再扫一遍** 找 iter. 返 iterator 让所有 API 都能高效操作.

    - **cost of abstraction**: 返 `bucket.end()` 作 "not found" 需要调用者**知道** iterator 语义.
    - Java / Python 版本通常返 nullable + separate 记录 index, 因为语言没 iterator 概念.

6. **🔑 `put` 的"更新 or 插入" 分支 / Update-or-insert branch**

    ```cpp
    void put(int key, int value) {
        auto& bucket = table[hash(key)];
        auto it = find(key);
        if (it != bucket.end()) it->second = value;   // 已存 → 更新
        else bucket.emplace_back(key, value);          // 不存 → 追加
    }
    ```

    - `emplace_back` 比 `push_back({k, v})` 少一次构造 (直接原地构造).
    - **不删旧 pair 再 push** — 直接改 second 更 efficient.

7. **🔑 复杂度 / Complexity**

    | API | 平均 | 最坏 |
    |---|---|---|
    | `put / get / remove` | **O(1 + α)** α = load factor | O(n) if all collide |

    - **Load factor α = n / SIZE**. Yang 用固定 SIZE = 769, 不 rehash.
    - **生产 hashmap** 会在 α > 阈值 (常 0.75) 时 **rehash** — 双 SIZE, 重新分配.
    - **本题不用 rehash** — LC 数据 ≤ 10⁴ ops, α 保持低.

    > **"rehash" 是生产级 hashmap 必需**, 面试**主动提**加分.

8. **🔑 空间 O(SIZE + n) / Space**

    - SIZE bucket 数组 (即使空).
    - n 个 key-value pair.

## Solution

=== "C++"
    ```cpp
    class MyHashMap {
    public:
        const static int SIZE = 769;                             // 质数减冲突
        vector<list<pair<int, int>>> table;

        int hash(int key) { return key % SIZE; }

        // DRY helper: 返 iterator (bucket.end() = 未找到)
        list<pair<int, int>>::iterator find(int key) {
            auto& bucket = table[hash(key)];
            for (auto it = bucket.begin(); it != bucket.end(); it++) {
                if (it->first == key) return it;
            }
            return bucket.end();
        }

        MyHashMap() : table(SIZE) {}

        void put(int key, int value) {
            auto& bucket = table[hash(key)];
            auto it = find(key);
            if (it != bucket.end()) it->second = value;          // 更新
            else bucket.emplace_back(key, value);                // 新插
        }

        int get(int key) {
            auto& bucket = table[hash(key)];
            auto it = find(key);
            return it != bucket.end() ? it->second : -1;
        }

        void remove(int key) {
            auto& bucket = table[hash(key)];
            auto it = find(key);
            if (it != bucket.end()) bucket.erase(it);            // list::erase O(1)
        }
    };
    ```

=== "Python"
    ```python
    class MyHashMap:
        SIZE = 769

        def __init__(self):
            # list of lists (buckets). Python list 支持中间删除, 但 O(n)
            # 教学清晰, LC 数据 ≤ 10^4 无所谓
            self.table: list[list[tuple[int, int]]] = [[] for _ in range(self.SIZE)]

        def _hash(self, key: int) -> int:
            return key % self.SIZE

        def put(self, key: int, value: int) -> None:
            bucket = self.table[self._hash(key)]
            for i, (k, _) in enumerate(bucket):
                if k == key:
                    bucket[i] = (key, value)          # 更新
                    return
            bucket.append((key, value))               # 新插

        def get(self, key: int) -> int:
            bucket = self.table[self._hash(key)]
            for k, v in bucket:
                if k == key: return v
            return -1

        def remove(self, key: int) -> None:
            bucket = self.table[self._hash(key)]
            for i, (k, _) in enumerate(bucket):
                if k == key:
                    bucket.pop(i)                     # O(n) 中间删, 但 bucket 通常短
                    return
    ```

=== "JavaScript"
    ```javascript
    var MyHashMap = function() {
        this.SIZE = 769;
        // Array of arrays. JS 数组 splice 中间删 O(n), 同 Python
        this.table = Array.from({length: this.SIZE}, () => []);
    };

    MyHashMap.prototype._hash = function(key) { return key % this.SIZE; };

    MyHashMap.prototype.put = function(key, value) {
        const bucket = this.table[this._hash(key)];
        for (const pair of bucket) {
            if (pair[0] === key) { pair[1] = value; return; }
        }
        bucket.push([key, value]);
    };

    MyHashMap.prototype.get = function(key) {
        const bucket = this.table[this._hash(key)];
        for (const [k, v] of bucket) if (k === key) return v;
        return -1;
    };

    MyHashMap.prototype.remove = function(key) {
        const bucket = this.table[this._hash(key)];
        for (let i = 0; i < bucket.length; i++) {
            if (bucket[i][0] === key) { bucket.splice(i, 1); return; }
        }
    };
    ```

## Complexity

| API | 平均 | 最坏 |
|---|---|---|
| `put / get / remove` | **O(1 + α)** | O(n) (all collide) |

- **Space**: O(SIZE + n).

## 相关题目

- [0242. Valid Anagram](../0242-valid-anagram/README.md) — hash 计数应用
- [0049. Group Anagrams](../0049-group-anagrams/README.md) — hash 分桶应用
- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — hash 反查
- [0128. Longest Consecutive Sequence](../0128-longest-consecutive-sequence/README.md) — hash set + 从头扩
- [0208. Implement Trie](../0208-implement-trie-prefix-tree/README.md) — 另一种 lookup 结构设计
- [0707. Design Linked List](../../02-linked-list/0707-design-linked-list/README.md) — 数据结构 API 设计, DRY 母题
- 0705\. Design HashSet (待补) — 姐妹题, 只存 key
- 0146\. LRU Cache (待补) — hash + 双向链表, 更复杂设计
- 0460\. LFU Cache (待补) — hash + 频次桶 + 双向链表
- 0432\. All O`one Data Structure (待补) — hash + 双向链表
