# 0127. Word Ladder / 单词接龙

!!! info "Meta"
    - **Difficulty**: Hard
    - **Tags**: Graph, BFS, Hash Table, String · 图, 广度优先, 哈希表, 字符串
    - **Link**: [LeetCode](https://leetcode.com/problems/word-ladder/)
    - **Status**: ✅ Solved
    - **Reviewed**: ☐ ☐ ☐

## Problem

**EN**: 给 `beginWord`, `endWord`, 词典 `wordList`. 每次可以**改一个字母** 变成词典里另一个词. 求 `beginWord` 到 `endWord` 的**最短转换序列长度** (含首尾), 不能到达返 `0`.

**中文**: 一次改一个字母, 中间词必须在词典里. 求 begin → end 最短转换序列长度.

## Key Insights

1. **🔑 隐式图 + 最短路径 → BFS (不是 DFS!) / Implicit graph + shortest path = BFS**

    把每个单词看成图的**节点**, 两个差**一个字母** 的词之间有一条边. 求最短路径 = BFS 的天然场景. 第一次到达 endWord 的 BFS 层数 + 1 就是答案 (含 beginWord 自己).

    > **关键: 最短路径不要用 DFS**. DFS 找路径但不保证最短, 而 BFS 第一次到达就是最短.

    > 跟之前 [§12 Graph](../index.md) 的 DFS 题不同 — 那些题 (0200/0695/...) 都是 flood fill, 没有"距离" 概念. **求最短** 必须 BFS.

2. **🔑 邻居生成: 26 字母 × 单词长度 / Generate neighbors by mutation**

    对当前词 `word`, 枚举每个位置 i 和每个字母 c (a-z), 形成 `newWord`. 若 `newWord` 在词典里且没访问过, 就是邻居.

    时间: 每词 O(L × 26) = O(L) (L = 词长). 总 O(N × L), N = 词典大小.

    > **不预建图**, 边动态生成. 因为词典里词的"邻居关系" 反复算成本太高, 预建邻接表 O(N² × L), 不如直接 mutate.

3. **`visitMap[word] = step` 双用 / Visited + distance in one map**

    Yang 用 `unordered_map<string, int>` 同时记**已访问** + **该词的距离**. 比"另开 visited set + 队列存 (word, step)" 更省内存.

    > 是不是 visited 由 `visitMap.count(word)` 判断; 距离由 `visitMap[word]` 取. **一举两得**.

4. **🔑 BFS 层数 → 答案 / Distance + 1 at endWord**

    BFS 从 beginWord 开始, `visitMap[beginWord] = 1` (路径含自己 = 1). 每访问一个邻居 step + 1. 第一次邻居等于 endWord → 立刻返回 `step + 1`.

    > Yang 的早返 `if (newWord == endWord) return path + 1;` 在邻居生成时就检测, 不用等再次出队. **快一层**.

5. **`if (!dict.count(endWord)) return 0;` 早返 / Early-exit if endWord absent**

    词典里没 endWord, 无解, 直接返 0. 题目允许 (`endWord must be in wordList` 不是 LC 的硬约束).

6. **复杂度 O(N × L²) (经典分析) / Complexity**

    - 词数 N, 词长 L.
    - 邻居生成每词 O(L × 26 × L) — 内层 `newWord == word` 比较 O(L), `dict.count(newWord)` 哈希 O(L).
    - 总 O(N × L²).

    > **进阶**: 双向 BFS (两端同时扩展) 把搜索空间从 `2^k` 降到 `2 × 2^(k/2)`, 大幅提速. LC 后端测数据下 BFS 也能过, 双向 BFS 优化留作进阶.

7. **跟 §07 Binary Tree 的 BFS 模板对照 / Same BFS template as binary tree**

    队列 + visited + 层数计数, 跟 [§07 BFS 题组](../../07-binary-tree/index.md) 一字不差. 图论 BFS 多一步**邻居生成** (从节点直接得到下一层), 树的"邻居" 就是 `left, right`.

## Solution

=== "C++"
    ```cpp
    class Solution {
    public:
        int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
            unordered_set<string> dict(wordList.begin(), wordList.end());
            if (!dict.count(endWord)) return 0;                                 // 早返

            queue<string> q;
            unordered_map<string, int> visitMap;                                // 既是 visited 又是 distance
            q.push(beginWord);
            visitMap[beginWord] = 1;                                            // 含自己 = 1

            while (!q.empty()) {
                string word = q.front(); q.pop();
                int step = visitMap[word];
                // 枚举每位 × 26 字母生成邻居
                for (int i = 0; i < (int)word.size(); i++) {
                    string newWord = word;
                    for (int c = 0; c < 26; c++) {
                        newWord[i] = 'a' + c;
                        if (newWord == word) continue;                          // 没变跳过
                        if (newWord == endWord) return step + 1;                // 早返
                        if (dict.count(newWord) && !visitMap.count(newWord)) {
                            q.push(newWord);
                            visitMap[newWord] = step + 1;
                        }
                    }
                }
            }
            return 0;                                                           // BFS 穷尽未到达
        }
    };
    ```

=== "Python"
    ```python
    from collections import deque

    class Solution:
        def ladderLength(self, beginWord: str, endWord: str, wordList: list[str]) -> int:
            words = set(wordList)
            if endWord not in words:
                return 0
            q = deque([(beginWord, 1)])                                         # (词, 距离)
            visited = {beginWord}

            while q:
                word, step = q.popleft()
                # 枚举每位 × 26 字母
                for i in range(len(word)):
                    # 切片拼接代替字符替换 (Python 字符串不可变)
                    for c in 'abcdefghijklmnopqrstuvwxyz':
                        new_word = word[:i] + c + word[i + 1:]
                        if new_word == word:
                            continue
                        if new_word == endWord:
                            return step + 1
                        if new_word in words and new_word not in visited:
                            visited.add(new_word)
                            q.append((new_word, step + 1))
            return 0
    ```

=== "JavaScript"
    ```javascript
    /**
     * @param {string} beginWord
     * @param {string} endWord
     * @param {string[]} wordList
     * @return {number}
     */
    var ladderLength = function(beginWord, endWord, wordList) {
        const dict = new Set(wordList);
        if (!dict.has(endWord)) return 0;
        const q = [[beginWord, 1]];
        const visited = new Set([beginWord]);
        while (q.length) {
            const [word, step] = q.shift();
            for (let i = 0; i < word.length; i++) {
                for (let c = 97; c <= 122; c++) {                              // a-z
                    const newWord = word.slice(0, i) + String.fromCharCode(c) + word.slice(i + 1);
                    if (newWord === word) continue;
                    if (newWord === endWord) return step + 1;
                    if (dict.has(newWord) && !visited.has(newWord)) {
                        visited.add(newWord);
                        q.push([newWord, step + 1]);
                    }
                }
            }
        }
        return 0;
    };
    ```

## Complexity

- **Time**: O(N × L²) — N 个词, 每词 O(L × 26) 邻居 × O(L) 比较/哈希.
- **Space**: O(N × L) — 队列 + visited.

## 相关题目

- [0797. All Paths From Source to Target](../0797-all-paths-from-source-to-target/README.md) — DAG 上 DFS 找所有路径 (本题 BFS 找最短)
- [0200. Number of Islands](../0200-number-of-islands/README.md) — DFS 网格连通
- 0126\. Word Ladder II (待补) — **找所有最短转换序列**, BFS 算距离 + DFS 回溯
- 0433\. Minimum Genetic Mutation (待补) — 本题完全同模板, 字符集换成 ACGT
- 0752\. Open the Lock (待补) — 数字版 BFS, 同邻居 mutate 思想
- §07 Binary Tree BFS 题组 — BFS 模板的大本营 → [0102 等](../../07-binary-tree/index.md)
