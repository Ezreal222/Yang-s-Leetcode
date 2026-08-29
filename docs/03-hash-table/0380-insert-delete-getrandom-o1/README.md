# 0380. Insert Delete GetRandom O(1) / O(1) 时间插入、删除和获取随机元素

!!! info "Meta"
    - **Difficulty**: Medium
    - **Tags**: Hash Table, Array, Math, Design, Randomized · 哈希表, 数组, 设计, 随机化
    - **Link**: [LeetCode](https://leetcode.com/problems/insert-delete-getrandom-o1/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## TL;DR / 一句话

> **EN**: **All three ops O(1), including uniform random pick** → **hash map + `vector`**. Hash gives O(1) lookup + O(1) update; `vector` gives O(1) random access. **Delete trick: swap-with-last then pop_back** — avoids O(n) middle-erase.
>
> **中文**: **三操作都要 O(1), 含随机取** → **哈希 + `vector`**. 哈希 O(1) 查, vector O(1) 随机下标. **删除靠"和末位交换再 pop_back"** 避免 O(n) 中间删.
>
> *Template / 模版*: **Two structures collaborate** — 类同 [0146 LRU](../0146-lru-cache/README.md), 只是一个偏"顺序" 一个偏"随机".

## Problem

**EN**:

- `insert(val)` — 若已存在返 false, 否则插入返 true.
- `remove(val)` — 若不存在返 false, 否则删除返 true.
- `getRandom()` — 等概率返当前集合任一元素.

**均须 O(1)** 平均时间.

**中文**: O(1) 集合 + 均匀随机.

## Key Insights

1. **🔑 灵魂: 为什么"哈希 + vector" / Why hash + vector?**

    每个操作都要 O(1), 单结构做不到:

    | 单结构 | insert | remove | getRandom | 死在哪 |
    |---|---|---|---|---|
    | 哈希表 | ✅ | ✅ | ❌ | 无法 O(1) 均匀采样 (要枚举 bucket) |
    | vector | ❌ (要判重 O(n)) | ❌ (中间删 O(n)) | ✅ (`vec[rand()%n]`) | 慢在查/删 |
    | set<T> (红黑树) | O(log n) | O(log n) | O(n) | 全不 O(1) |

    **组合**: `vector` 作**紧凑数组** 给随机采样; `hash` 存 `val → vec 下标` 给 O(1) 查. 一个负责"顺序/位置", 一个负责"存在/索引".

    > **对比 0146 LRU**: 那题 hash + **doubly linked list** (要保持"最近用" 顺序). 本题 hash + **vector** (要 O(1) 随机下标). **看主操作是啥, 就配啥容器**.

2. **🔑 灵魂: 删除的 swap-with-last trick / Swap-with-last then pop_back**

    直接 `vec.erase(vec.begin() + i)` = **O(n)** (后面全前移). 换法:

    ```cpp
    int i = mp[val];          // 要删的位置
    vec[i] = vec.back();      // 把末位搬到 i, 覆盖掉 val
    mp[vec[i]] = i;           // ⚠️ 关键: 更新末位元素在 map 中的新下标
    vec.pop_back();           // O(1) 删末位
    mp.erase(val);            // 删 hash 条目
    ```

    → 打乱了 vector 顺序, 但**本题不要求顺序** — 只要求集合语义 + 随机采样. 免费降到 O(1).

    > **"删除中间元素" 的通用招**: 若不需保序, **swap-with-last** 永远 O(1). 也见 0026 Remove Duplicates / 0027 Remove Element (待补) 的双指针 in-place 家族.

3. **🔑 别忘更新末位元素的 map / Update the moved element's index in map**

    Yang 代码:

    ```cpp
    vec[i] = vec.back();
    mp[vec[i]] = i;            // 末位元素现在住 i, map 得改
    ```

    最容易忘的一步. 如果不更新, 下次 `remove(vec.back())` 时 map 说它在旧下标 → 越界或错删.

    **边界**: 若 `i == vec.size() - 1` (删的就是末位), `vec[i] = vec.back()` 是自赋值, `mp[vec[i]] = i` 也是重复赋值 — **不会错, 只是多做**. 想优化可加 `if (i != vec.size() - 1)` 但没必要.

    > **易错点 top 1**: 忘更新 map 里被搬动那个元素的下标.

4. **🔑 `getRandom()` — `rand() % n` 均匀性 / Uniform via modulo**

    `vec` 是紧凑数组 (没有洞), 所以 `vec[rand() % vec.size()]` 就是**均匀随机**. 每个元素被选中的概率恰好 `1/n`.

    - 若用 `unordered_set` 想"跳到随机 bucket"会**不均匀** — 因为 bucket 内长度不等.
    - `rand()` 的**分布均匀性**取决于实现. LC 的 C++ 判题用 glibc `rand()`, 足够均匀. 面试可提 `mt19937 + uniform_int_distribution` 更严谨:

        ```cpp
        mt19937 gen{random_device{}()};
        int getRandom() {
            uniform_int_distribution<> d(0, vec.size() - 1);
            return vec[d(gen)];
        }
        ```

    > **"均匀随机" 面试关键词**: 紧凑数组 + 均匀分布 rng.

5. **🔑 用 `mp.count(val)` vs `mp.find(val) != mp.end()` / Style**

    Yang 混用:

    - `insert`: `mp.count(val)` — 短.
    - `remove`: `auto it = mp.find(val); if (it == mp.end()) return false; int i = it->second;` — 一次查找拿 iterator + value, 比 `count` + `[]` 少一次 hash.

    两种都对. 追求性能时**优先 `find` + iterator**, 追求简洁时 `count`.

6. **🔑 相比 `set<int>` — 为啥有序结构不够 / Why not a balanced BST?**

    `std::set` (红黑树) 每操作 O(log n), 且**不支持 O(1) 随机采样** — 你得中序 walk 到第 k 个才行. → 只能用哈希 + vector.

    > **"O(1) 随机" 是本题独有约束**, 逼你放弃有序结构.

7. **🔑 复杂度 / Complexity**

    - **Time**: `insert` / `remove` / `getRandom` 均 **O(1) 摊销**.
    - **Space**: O(n).

8. **🔑 后续变体 / Follow-ups**

    - [0381 Insert Delete GetRandom O(1) — Duplicates allowed](待补) — 值可重复. 改成 `unordered_map<int, unordered_set<int>>` 存"这个值出现在哪些下标". remove 时随便挑一个下标 swap-with-last.
    - **加权随机**: 每个 val 有权重. → 前缀和 + `lower_bound` (O(log n) 采样), 或 alias method (O(1) 采样, 预处理 O(n)).

## Solution

=== "C++"
    ```cpp
    class RandomizedSet {
        vector<int> vec;                       // 紧凑数组, 给 getRandom 用
        unordered_map<int, int> mp;            // val → vec 下标
    public:
        RandomizedSet() {}

        bool insert(int val) {
            if (mp.count(val)) return false;
            mp[val] = vec.size();
            vec.push_back(val);
            return true;
        }

        bool remove(int val) {
            auto it = mp.find(val);
            if (it == mp.end()) return false;
            int i = it->second;
            vec[i] = vec.back();               // 末位盖过来
            mp[vec[i]] = i;                    // 关键: 更新被搬元素的下标
            vec.pop_back();                    // O(1) 弹尾
            mp.erase(val);
            return true;
        }

        int getRandom() {
            return vec[rand() % vec.size()];
        }
    };
    ```

=== "Python"
    ```python
    import random

    class RandomizedSet:
        # dict + list 的组合. Python 没有 vector, 但 list 底层是动态数组:
        # - list[i]        O(1) 随机访问
        # - list.pop()     O(1) 弹末位
        # - list.pop(i)    O(n) 中间弹 → 别用! 用 swap-with-last
        # - list.append()  O(1) 摊销

        def __init__(self):
            self.arr: list[int] = []
            self.idx: dict[int, int] = {}    # val → arr 下标

        def insert(self, val: int) -> bool:
            if val in self.idx:
                return False
            self.idx[val] = len(self.arr)
            self.arr.append(val)
            return True

        def remove(self, val: int) -> bool:
            if val not in self.idx:
                return False
            i, last = self.idx[val], self.arr[-1]
            # swap-with-last: 把末位搬到 i, 弹末位
            self.arr[i] = last
            self.idx[last] = i               # 更新末位元素的新下标
            self.arr.pop()                   # O(1)
            del self.idx[val]                # dict 的 del 是 O(1)
            return True

        def getRandom(self) -> int:
            # random.choice 内部就是 seq[randrange(len(seq))], 均匀
            return random.choice(self.arr)
    ```

=== "JavaScript"
    ```javascript
    // JS 用 Array + Map. Map 保持插入顺序 (本题不利用),
    // 但 O(1) has/get/set 的哈希语义就够了.

    var RandomizedSet = function() {
        this.arr = [];
        this.idx = new Map();     // val → arr 下标
    };

    RandomizedSet.prototype.insert = function(val) {
        if (this.idx.has(val)) return false;
        this.idx.set(val, this.arr.length);
        this.arr.push(val);
        return true;
    };

    RandomizedSet.prototype.remove = function(val) {
        if (!this.idx.has(val)) return false;
        const i = this.idx.get(val);
        const last = this.arr[this.arr.length - 1];
        this.arr[i] = last;                       // 末位盖过来
        this.idx.set(last, i);                    // 更新末位在 map 的下标
        this.arr.pop();                           // O(1)
        this.idx.delete(val);
        return true;
    };

    RandomizedSet.prototype.getRandom = function() {
        // Math.floor(Math.random() * n) 均匀
        // Math.random() 返 [0,1) 双精度浮点, 分布足够均匀
        return this.arr[Math.floor(Math.random() * this.arr.length)];
    };
    ```

## Complexity

| 操作 | Time | Space |
|---|---|---|
| `insert` / `remove` / `getRandom` | **O(1)** 摊销 | O(n) |

## 相关题目

- [0706. Design HashMap](../0706-design-hashmap/README.md) — 哈希设计基础
- [0146. LRU Cache](../0146-lru-cache/README.md) — hash + 双向链表 (顺序型多结构)
- 0027\. Remove Element (待补) — swap-with-last 母题
- 0026\. Remove Duplicates from Sorted Array (待补) — 类似 in-place 技巧
- [0001. Two Sum](../../01-array/0001-two-sum/README.md) — hash 反查
- 0381\. Insert Delete GetRandom O(1) - Duplicates allowed (待补) — 直接升级
- 0710\. Random Pick with Blacklist (待补) — 随机 + 排除集
- 0528\. Random Pick with Weight (待补) — 加权随机, 前缀和
- 0398\. Random Pick Index (待补) — 蓄水池采样
- 0382\. Linked List Random Node (待补) — 蓄水池采样母题
