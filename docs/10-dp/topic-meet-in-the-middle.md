# 技巧 · 折半搜索 (Meet in the Middle)

**EN**: Brute-force optimization technique: split the enumeration in half, solve each half independently, merge. Turns `O(2^n)` into `O(2^(n/2))` — exponential becomes square root.

**中文**: 把"枚举整体 2^n" 拆成"枚举两半各 2^(n/2) + 合并", 把指数复杂度**开方**. 典型的**空间换时间**.

---

## 核心思想: 2^n → 2 × 2^(n/2)

```
原始: O(2^n)              n=40 → 2^40 ≈ 10^12  ❌
折半: O(2^(n/2) × 合并)    n=40 → 2^20 ≈ 10^6   ✓
```

**指数减半** — 从不可行变可行. 代价是 `O(2^(n/2))` 空间存一半结果.

---

## 什么时候用?

**数字感** (背下来):

```
n ≤ 20  → 直接 2^n 枚举 (10^6, 够快)
n ≤ 40  → 折半搜索       (2^20, 可行)
n > 40  → 折半也不行, 找多项式算法
```

**触发信号**:

- `n` 在 **20~40** 之间, 普通 `2^n` 暴力会爆.
- 需要枚举**子集 / 组合**.
- 没有更优的多项式算法 (DP / 贪心都不适用).
- 问题能"分两半独立处理 + 合并".

**vs 背包 DP**:

| | 背包 DP | 折半搜索 |
|---|---|---|
| 适用 | 容量/和**范围小** | n **个数小** (≤40), 值范围大 |
| 复杂度 | O(n × 容量) | O(2^(n/2)) |
| 限制 | 容量爆就崩 | n 太大就崩 |

> **何时弃背包用折半?** 当"和/容量太大" 导致背包状态爆炸, 但 `n` 较小 (≤40) 时. 例: [LC 2035] 元素可达 10^5, 和的范围太大 → 背包不行; 但 n ≤ 15 → 折半完美.

---

## 三步走

```
1. 分半:  把 n 个元素分成两组 (各 n/2)
2. 枚举:  分别枚举每组所有子集 (各 2^(n/2), 用 bitmask)
3. 合并:  把两组结果配对/查找, 得到答案
```

**第 3 步"合并" 是关键** — 不同问题合并方式不同:

| 问题类型 | 合并方式 |
|---|---|
| 子集和判定 (能否凑到 target) | 查找互补 — `unordered_set::count` |
| 最接近 target | 排序 + 二分 — `upper_bound` |
| 方案数 | 哈希计数 — `unordered_map<int,int>` |

---

## 基础工具: bitmask 枚举子集

折半的前提是"枚举一半的所有子集" — 标准写法就是 bitmask:

```cpp
// 枚举 half 个元素的所有 2^half 个子集
for (int mask = 0; mask < (1 << half); mask++) {
    int sum = 0;
    for (int i = 0; i < half; i++) {
        if (mask & (1 << i)) {              // 第 i 位为 1 → 选第 i 个元素
            sum += nums[i];
        }
    }
    // 处理这个子集的和 sum
}
```

`mask` 从 `0` 到 `2^half - 1`, 每个值对应一种"选/不选" 组合. 例子 (half=3):

```
mask=000(0): {}                 — 都不选
mask=001(1): {nums[0]}
mask=011(3): {nums[0], nums[1]}
mask=111(7): {nums[0],nums[1],nums[2]}
```

**搭配**: `__builtin_popcount(mask)` 数 `mask` 里有几个 1 = 选了几个元素 (GCC 内置, O(1)). Python 对应 `bin(mask).count('1')` 或 `mask.bit_count()` (3.10+); JS 用 `mask.toString(2).split('1').length - 1`.

---

## 模板 1: 子集和判定 (查找互补)

**问题**: 给 n 个数 (n ≤ 40), 能否选若干和为 `target`?

```cpp
bool canReachTarget(vector<int>& nums, int target) {
    int n = nums.size();
    int half = n / 2;

    // 1. 枚举左半所有子集和, 入 set
    unordered_set<int> leftSums;
    for (int mask = 0; mask < (1 << half); mask++) {
        int sum = 0;
        for (int i = 0; i < half; i++) {
            if (mask & (1 << i)) sum += nums[i];
        }
        leftSums.insert(sum);
    }

    // 2. 枚举右半, 查 target - sumR 是否在 leftSums
    for (int mask = 0; mask < (1 << (n - half)); mask++) {
        int sumR = 0;
        for (int i = 0; i < n - half; i++) {
            if (mask & (1 << i)) sumR += nums[half + i];
        }
        if (leftSums.count(target - sumR)) return true;
    }
    return false;
}
```

**复杂度**: 枚举 + 查找各 `O(2^(n/2))` → 整体 `O(2^(n/2))`. 远小于 `2^n`.

---

## 模板 2: 最接近 target (排序 + 二分)

**问题**: 选若干数, 和**尽量接近** target (不超过).

```cpp
// 1. 枚举左半, 存所有和并排序
vector<int> leftSums;
for (int mask = 0; mask < (1 << half); mask++) {
    leftSums.push_back(/* 计算和 */);
}
sort(leftSums.begin(), leftSums.end());

// 2. 枚举右半, 对每个 sumR 二分查 leftSums 中 ≤ (target - sumR) 的最大值
int best = INT_MAX;
for (int mask = 0; mask < (1 << (n - half)); mask++) {
    int sumR = /* 计算 */;
    int need = target - sumR;
    auto it = upper_bound(leftSums.begin(), leftSums.end(), need);
    if (it != leftSums.begin()) {
        int sumL = *prev(it);
        best = min(best, target - (sumL + sumR));
    }
}
```

**合并复杂度**: `O(2^(n/2) × log(2^(n/2)))` = `O(2^(n/2) × n/2)` — 多了个 log.

---

## 进阶: 带"个数约束" 的折半 (LC 2035 套路)

LC 2035 在基础折半上加了"必须**各选 n 个**" 的约束 — 配对时只能"左半选 k 个 + 右半选 n-k 个".

**改动**: 枚举每半子集时, 按"选了几个" 分桶:

```cpp
// 左半: leftSums[k] = 左半选 k 个元素的所有子集和
vector<vector<int>> leftSums(half + 1);
for (int mask = 0; mask < (1 << half); mask++) {
    int cnt = __builtin_popcount(mask);     // 选了几个
    int sum = /* 计算和 */;
    leftSums[cnt].push_back(sum);
}
// 右半同样按个数分桶 + 各自排序

// 配对: 左半选 k 个 + 右半选 (n-k) 个
for (int k = 0; k <= half; k++) {
    for (int sumL : leftSums[k]) {
        // rightSums[n-k] 里二分找最接近 target-sumL 的
    }
}
```

> **LC 2035 = 折半搜索 + 按个数分桶 + 排序二分** — 比基础折半多了"个数约束" 维度.

---

## 应用题

| 题号 | 题目 | 折半用法 | 状态 |
|---|---|---|---|
| [1755](./1755-closest-subsequence-sum/README.md) | Closest Subsequence Sum / 最接近目标值的子序列和 | 折半 + 排序二分 (纯模板) | ✅ |
| [2035](./2035-partition-array-into-two-arrays-to-minimize-sum-difference/README.md) | Partition Array Into Two Arrays to Minimize Sum Diff / 将数组分成两个数组并最小化和的差 | 折半 + 个数分桶 + 二分 | ✅ |
| 956 | Tallest Billboard / 最高的广告牌 | 折半 (或 三状态 DP) | (待补) |
| 1170+ | 子集和 (n ≤ 40 经典) | 折半 + 查找 | — |

**学习路径建议**:

1. 先掌握 **bitmask 枚举子集** — LC 78 (Subsets) 用 bitmask 写一遍, 熟悉 `mask & (1<<i)`, `__builtin_popcount`.
2. 折半入门: **LC 1755** — 纯模板, 折半 + 排序二分.
3. 折半进阶: **LC 2035** — 加个数约束, 用桶 + 二分.

---

## 一句话总结

> 整体枚举 `2^n` 太大 → 分两半独立枚举 `2^(n/2)` + 合并 → **指数开方**. 代价是 `O(2^(n/2))` 空间存一半结果. 是"空间换时间" 的极致暴力优化.

记忆五点:

1. **核心**: `2^n → 2 × 2^(n/2)` (指数减半).
2. **三步**: 分半 → bitmask 枚举每半 → 合并.
3. **合并**: 查找互补 (set) / 排序二分 (最接近) / 哈希计数 (方案数).
4. **信号**: `n ∈ [20, 40]` + 枚举子集 + 普通 DP 不行.
5. **工具**: bitmask + `__builtin_popcount`.

---

## 相关章节

- [§10 DP 主页](./index.md) — 折半是背包 DP 的替代方案
- [0416. Partition Equal Subset Sum](./0416-partition-equal-subset-sum/README.md) — 同 "分两堆" 模型, 但 n 大值小 → 用 0/1 背包
- [1049. Last Stone Weight II](./1049-last-stone-weight-ii/README.md) — 同上, 0/1 背包路线
