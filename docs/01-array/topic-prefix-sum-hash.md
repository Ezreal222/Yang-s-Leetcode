# Topic · 前缀和 + Hash 六题速查 / Prefix Sum + Hash Cheatsheet

**6 题都是同一套武器**: 一遍扫累加 prefix, 用 hash 反查历史. 差别只在**"存什么 / 查什么 / 何时映射"** 三处. 记熟这张表, 见到子数组和/路径和类题就能秒选正确 pattern.

---

## 🎯 通用套路 / Universal Recipe

```
sum = 0
seed hash with base case (哨兵)
for x in nums (或 DFS 遍树):
    sum += encode(x)                # 可选映射: 原值 / mod / mapping
    query hash for "matching prefix" and record answer
    update hash for current sum      # 顺序: 先查后更 (0437 用 backtrack)
return answer
```

**灵魂**: `sum(区间) = prefix[end] - prefix[start]`. 于是"区间满足某条件" ⇔ "两 prefix 满足某关系" ⇔ "**hash 反查历史 prefix**".

---

## 📊 6 题一览表 / One-Table Comparison

| # | 题 | 求什么 | 累加规则 | Hash 存什么 | 查什么 | 结束操作 |
|---|---|---|---|---|---|---|
| **0560** | Subarray Sum == K | **个数** | `sum += x` | `count[sum]` | `res += cnt[sum − k]` | `cnt[sum]++` |
| **0523** | 存在 sum 是 k 倍数 (长 ≥ 2) | **bool** | `sum = (sum + x) % k` | `firstIdx[sum]` | 若已存: `boundary − stored ≥ 2` ⇒ true | 只在首次插入, 不覆盖 |
| **0974** | 个数 sum 被 k 整除 | **个数** | `sum = ((sum + x) % k + k) % k` | `count[sum]` | `res += cnt[sum]` (**同 mod**) | `cnt[sum]++` |
| **0525** | **最长** 0/1 平衡 | **最长长度** | `sum += (x == 1 ? 1 : -1)` (mapping) | `firstIdx[sum]` | `maxLen = max(maxLen, i − stored)` | 只在首次插入, 不覆盖 |
| **1248** | 恰含 k 个奇数子数组 | **个数** | `sum += (x & 1)` (mapping) | `count[sum]` | `res += cnt[sum − k]` | `cnt[sum]++` |
| **0437** | 树上向下路径 sum == k 个数 | **个数** | DFS 上 `cur += node.val` | `count[cur]` | `res += cnt[cur − k]` | 进入 `++`, **离开 `--`** (回溯) |

---

## 🧠 3 种 hash 语义 / Three Hash Patterns

**决定用哪种 hash 完全看"你在求什么"**:

| 求 | Hash 语义 | 逻辑 |
|---|---|---|
| **个数 (count)** | `hash[key] → 出现次数` | 每次 `res += cnt[matching_key]`, 然后 `cnt[key]++` |
| **存在 / 最长 / 最远** | `hash[key] → 首次出现的 index (or boundary)` | 每次若 key 已存则计算距离; **不覆盖** 保留最早 |
| **树上路径** | 同"个数", **但加 backtrack** (进 ++, 离 --) | 保证路径只走"祖先 → 后代", 不跨兄弟 |

**记忆口诀**:

> **"个数 → count map, 距离/最长 → first-index map, 树 → count map + 回溯"**

---

## 🔄 4 种"累加映射" / Four Value Encodings

**问题的挑战被 encode 到累加规则里, 剩下的模板不变**:

| 映射 | 用途 | 例题 |
|---|---|---|
| `sum += x` | 原和 | [0560](./0560-subarray-sum-equals-k/README.md) |
| `sum = ((sum + x) % k + k) % k` | mod k (被 k 整除类) | [0523](./0523-continuous-subarray-sum/README.md), [0974](./0974-subarray-sums-divisible-by-k/README.md) |
| `sum += (x == 1 ? 1 : -1)` | 0/1 平衡 → 和 = 0 | [0525](./0525-contiguous-array/README.md) |
| `sum += (x & 1)` | 奇偶计数 → 和 = k | [1248](./1248-count-number-of-nice-subarrays/README.md) |

**通用原则**:

> **"约束条件不好数 → 换个数值编码 → 归到基础模板"**. Mapping 是这一族题的**灵魂化简手段**.

---

## 🛡 哨兵 (sentinel) 都必须 / Sentinel is mandatory in all 6

**空前缀 = 0 存在 1 次** — 处理"从起点开始的合法区间/路径":

| 题 | 哨兵 |
|---|---|
| 0560, 0974, 1248, 0437 (count 类) | `cnt[0] = 1` |
| 0523 (first-index) | `firstIdx[0] = 0` |
| **0525 (first-index)** | `firstIdx[0] = -1` ⚠️ 用 **-1** 让 `i − (−1) = i + 1` 正好是长度 |

**记忆**: **count 类哨兵是 1, first-index 类哨兵是 0 (或 -1)** — 数字选择跟"长度公式" 咬合.

---

## ⚡ 决策树 / Decision Tree

**给一道新题, 3 秒内选出正确 pattern**:

```
1. 是不是"子数组 (或树上路径) + 某种和条件"?
   ├─ 是 → prefix + hash 家族
   └─ 否 → 换别的思路

2. 目标是什么?
   ├─ 个数 → count map
   ├─ 存在 (+ 距离约束) 或 最长/最远 → first-index map
   └─ 树上 → count map + backtrack

3. 和的条件是什么?
   ├─ 等于 k → sum += x, 查 sum - k
   ├─ 被 k 整除 → sum = mod, 查 sum (同 mod)
   ├─ 0/1 (或双值) 平衡 → mapping 0→-1, 1→+1, 查 sum
   └─ 恰含 X 个 Y → mapping X→1, 其他→0, 查 sum - k
```

---

## ⚠ 常见陷阱 / Common Pitfalls

1. **`long long` 防溢出** — 值大 (0437: ±10⁹) 或深度大时**必需**. 0560/0523/0974 通常防御性 (LC 不严格需要).
2. **C++/JS `%` 对负数返负** — mod 类必须 `((x % k) + k) % k` 强制正化. Python `%` 天然正, 免这一步.
3. **先查后更, 顺序不能反** — count 类反了会**自匹配**, 尤其 k=0 时立刻挂.
4. **first-index 类不能覆盖** — 求距离最大化, 必须保留最早的 index.
5. **0437 backtrack** — 少 `--cnt[cur]` 就跨兄弟污染. 是**树上专属** 陷阱.
6. **哨兵的数字选择** — count 用 1 (代表"空前缀存在过一次"); first-index 用 0 或 -1 (**看长度公式咬合**).

---

## 🔗 6 题详情链接 / Problem Links

- [0560. Subarray Sum Equals K](./0560-subarray-sum-equals-k/README.md) — count 母题
- [0523. Continuous Subarray Sum](./0523-continuous-subarray-sum/README.md) — first-index + mod
- [0974. Subarray Sums Divisible by K](./0974-subarray-sums-divisible-by-k/README.md) — count + mod
- [0525. Contiguous Array](./0525-contiguous-array/README.md) — first-index + mapping (最长)
- [1248. Count Number of Nice Subarrays](./1248-count-number-of-nice-subarrays/README.md) — count + mapping
- [0437. Path Sum III](../07-binary-tree/0437-path-sum-iii/README.md) — count + tree DFS + **backtrack**

---

## 🎓 一句话背诵 / One-Line Memorization

> **"prefix + hash: `count` 求个数, `first-index` 求距离. 累加规则里塞映射: `+x` 原和, `% k` 整除, `±1` 平衡, `&1` 奇偶. 哨兵永远设. 树上加回溯."**
