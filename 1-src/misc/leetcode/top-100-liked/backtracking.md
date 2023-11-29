# The Backtracking of [*Top 100 Liked*](https://leetcode.com/studyplan/top-100-liked/) in Rust

This is a Rust version of [The Backtracking of *Top 100 Liked* in C++](https://github.com/harvey-lau/road2cs/blob/main/1-src/programming/leetcode/top-100-liked/backtracking.md).

## 0x01 Common Pattern

[BackTracking (BT)](https://en.wikipedia.org/wiki/Backtracking) is a class of algorithms for finding solutions to some problems, notably [constraint satisfaction problems](https://en.wikipedia.org/wiki/Constraint_satisfaction_problem), that incrementally builds candidates to the solutions, and abandons a candidate ("backtracks") as soon as it determines that the candidate cannot possibly be completed to a valid solution.

A BT problem has 3 key elements:

- BT search tree
- BT candidate extension step
- BT boundary

To solve a BT problem, there are 3 steps:

1. Build the **BT search tree**.
2. Follow the **BT candidate extension step** recursively.
3. Return all solutions at the **BT boundary**.

The template of a BT problem is:

```Rust
impl Solution {
    fn bt_problem(inputs: Type) -> Vec<Type> {
        let mut ans: Vec<Type> = Vec::new();
        let mut ans_i: Type;
        // Step 1: build the BT search tree.
        Solution::bt(0, ans_i, &mut ans, inputs);
        ans
    }
    
    fn bt(d: usize, ins: Type, s: &mut Type, ss: &mut Vec<Type>) {
        // Step 3: return a solutions at the BT boundary.
        if d == ins.len() {
            ss.push(s.clone());
            return;
        }
        // Step 2: follow the BT candidate extension step recursively.
        for in_val in ins {
            s.push(in_val);
            Solution::bt(d + 1, ins.clone(), s, ss);
            s.pop();
        }
    }
}
```

## 0x02 Problems

### 1. [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

---

Given a string containing digits from `2-9` inclusive, return all possible letter combinations that the number could represent. Return the answer in **any order**.

A mapping of digits to letters (just like on the telephone buttons) is given [in the description of this problem](https://leetcode.com/problems/letter-combinations-of-a-phone-number/description). Note that 1 does not map to any letters.

**Constraints**:

- `0 <= digits.length <= 4`
- `digits[i]` is a digit in the range `['2', '9']`.

---

- BT search tree: the 0th layer is empty; the 1st layer is the mapping letters of `digits[0]`; the 2nd layer is the mapping letters of `digits[1]` which are repeated 3 or 4 times under the different nodes of the 1st layer... Without the 0th layer, the height of BT search tree is `digits.length`.
- BT candidate extension step: no pruning.
- BT boundary: `the searching depth == digits.length`

The code:

```Rust
use std::collections::HashMap;

impl Solution {
    pub fn letter_combinations(digits: String) -> Vec<String> {
        let mut ans: Vec<String> = Vec::new();
        let mut ans_i: String = String::new();
        let n = digits.len();
        if n == 0 {
            return ans;
        }
        let keyboard_9: HashMap<char, &str> = [
            ('2', "abc"),
            ('3', "def"),
            ('4', "ghi"),
            ('5', "jkl"),
            ('6', "mno"),
            ('7', "pqrs"),
            ('8', "tuv"),
            ('9', "wxyz")
        ].iter().cloned().collect();
        Solution::bt(0, &digits, &keyboard_9, &mut ans_i, &mut ans);
        ans
    }
    
    fn bt(d: usize, nums: &str, num_to_letter: &HashMap<char, &str>, s: &mut String, ss: &mut Vec<String>) {
        if d == nums.len() {
            ss.push(s.clone());
            return;
        }
        let now_num = nums.chars().nth(d).unwrap();
        let now_letters = num_to_letter.get(&now_num).unwrap();
        for now_letter in now_letters.chars() {
            s.push(now_letter);
            Solution::bt(d + 1, nums, num_to_letter, s, ss);
            s.pop();
        }
    }
}
```

### 2. [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/)

---

Given `n` pairs of parentheses, write a function to *generate all combinations of well-formed parentheses*.

**Constraints**:

- `1 <= n <= 8`

---

- BT search tree: the 0th layer is empty; the 1st layer is `(` and `)`; the 2nd layer is `(` and `)` which are repeated 2 times under the `(` and `)` of the 1st layer... Without the 0th layer, the height of BT search tree is `n`.
- BT candidate extension step: no pruning.
- BT boundary: `the searching depth == 2 * n`

However, we should judge whether a candidate is well-formed or not at the BT boundary.

The code:

```Rust
impl Solution {
    pub fn generate_parenthesis(n: i32) -> Vec<String> {
        let mut ans: Vec<String> = Vec::new();
        let mut ans_i: String = String::new();
        Solution::bt(0, n, &mut ans_i, &mut ans);
        ans
    }
    
    fn bt(d: i32, n: i32, s: &mut String, ss: &mut Vec<String>) {
        if d == 2 * n {
            if Solution::is_valid(s) {
                ss.push(s.clone());
            }
            return;
        }
        s.push('(');
        Solution::bt(d + 1, n, s, ss);
        s.pop();
        s.push(')');
        Solution::bt(d + 1, n, s, ss);
        s.pop();
    }
    
    fn is_valid(s: &str) -> bool {
        let mut st: Vec<char> = Vec::new();
        let n = s.len();
        for i in 0..n {
            if s.chars().nth(i).unwrap() == '(' {
                st.push('(');
            } else {
                if !st.is_empty() && st.last().unwrap() == &'(' {
                    st.pop();
                } else {
                    st.push(')');
                }
            }
        }
        st.is_empty()
    }
}
```

### 3. [Combination Sum](https://leetcode.com/problems/combination-sum/)

---

Given an array of **distinct** integers `candidates` and a target integer `target`, return *a list of all **unique combinations** of `candidates` where the chosen numbers sum to `target`*. You may return the combinations in **any order**.

The **same** number may be chosen from candidates an **unlimited number of times**. Two combinations are unique if the frequency of at least one of the chosen numbers is different.

The test cases are generated such that the number of unique combinations that sum up to `target` is less than `150` combinations for the given input.

> The **frequency** of an element `x` is the number of times it occurs in the array.

**Constraints**:

- `1 <= candidates.length <= 30`
- `2 <= candidates[i] <= 40`
- All elements of `candidates` are **distinct**.
- `1 <= target <= 40`

---

- BT search tree: the 0th layer is empty; the 1st layer is choosing candidates[i]; the 2nd layer is choosing candidates[i] which are repeated `candidates.length` times under the 1st layer... The height of BT search tree depends on `target`. For example, if `min(candidates[i]) = d` and `target = l`, the height of BT (without the 0th layer) is `l/d`.
- BT candidate extension step: no pruning.
- BT boundary: `Sigma(combination[i]) >= target`

We should collect combinations MECE at the BT boundary.

The code:

```Rust
impl Solution {
    pub fn combination_sum(candidates: Vec<i32>, target: i32) -> Vec<Vec<i32>> {
        let mut ans: Vec<Vec<i32>> = Vec::new();
        let mut ans_i: Vec<i32> = Vec::new();
        let mut sorted_candidates = candidates.clone();
        sorted_candidates.sort();
        Solution::bt(target, &sorted_candidates, &mut ans_i, &mut ans);
        ans
    }

    fn bt(d: i32, nums: &Vec<i32>, v: &mut Vec<i32>, vv: &mut Vec<Vec<i32>>) {
        if d <= 0 {
            if d == 0 {
                for i in 1..v.len() {
                    if v[i] > v[i - 1] {
                        return;
                    }
                }
                vv.push(v.clone());
            }
            return;
        }
        for i in 0..nums.len() {
            v.push(nums[i]);
            Solution::bt(d - nums[i], nums, v, vv);
            v.pop();
        }
    }
}
```

### 4. [Permutations](https://leetcode.com/problems/permutations/)

---

Given an array `nums` of distinct integers, return *all the possible permutations*. You can return the answer in **any order**.

**Constraints**:

- `1 <= nums.length <= 6`
- `-10 <= nums[i] <= 10`
- All the integers of `nums` are **unique**.

---

- BT search tree: the 0th layer is empty; the 1st layer is choosing `nums[i]`; the 2nd layer is choosing `nums[i]` which are repeated `nums.length` times under the 1st layer... Without the 0th layer, the height of BT search tree is `nums.length`.
- BT candidate extension step: prune all chosen `nums[i]`(s). Therefore, we need a vector `visited` to record the chosen `nums[i]`(s).
- BT boundary: `the searching depth == nums.length`

The code:

```Rust
impl Solution {
    pub fn permute(nums: Vec<i32>) -> Vec<Vec<i32>> {
        let mut ans: Vec<Vec<i32>> = Vec::new();
        let mut ans_i: Vec<i32> = Vec::new();
        let n = nums.len();
        let mut visited: Vec<bool> = vec![false; n];
        Solution::bt(0, &nums, &mut visited, &mut ans_i, &mut ans);
        ans
    }
    
    fn bt(d: usize, nums: &Vec<i32>, mark: &mut Vec<bool>, v: &mut Vec<i32>, vv: &mut Vec<Vec<i32>>) {
        let n = nums.len();
        if d == n {
            vv.push(v.clone());
            return;
        }
        for i in 0..n {
            if !mark[i] {
                v.push(nums[i]);
                mark[i] = true;
                Solution::bt(d + 1, nums, mark, v, vv);
                v.pop();
                mark[i] = false;
            }
        }
    }
}
```

### 5. [N-Queens](https://leetcode.com/problems/n-queens/)

---

The **n-queens** puzzle is the problem of placing `n` queens on an `n x n` chessboard such that no two queens attack each other.

Given an integer `n`, return *all distinct solutions to the **n-queens puzzle***. You may return the answer in **any order**.

Each solution contains a distinct board configuration of the n-queens' placement, where `'Q'` and `'.'` both indicate a queen and an empty space, respectively.

**Constraints**:

- `1 <= n <= 9`

---

If we give all queens a fixed row order like `1...n`, a permutation of `1...n` can represent the placing of queens. For example, `2431` means 4 queens are placed at (1, 2), (2, 4), (3, 3), and (4, 1) respectively. Based on this idea, we can further prune the solutions of last problem. The `permutations` problem satisfies the constraints of moving vertically and horizontally. The constraint of moving diagonally can be satisfied at the BT boundary by checking `j - i == v[i] - v[j] || i - j == v[i] - v[j]`. At last, mapping permutation to placement is the final step of solving the `N-Queens` problem.

The code:

```Rust
impl Solution {
    pub fn solve_n_queens(n: i32) -> Vec<Vec<String>> {
        let mut ans_s: Vec<Vec<String>> = Vec::new();
        let mut ans: Vec<Vec<i32>> = Vec::new();
        let mut ans_i: Vec<i32> = Vec::new();
        let mut visited: Vec<bool> = vec![false; n as usize];
        Solution::bt(0, n, &mut visited, &mut ans_i, &mut ans);
        if ans.len() > 0 {
            Solution::trans(&mut ans_s, &ans);
        }
        ans_s
    }

    fn bt(d: i32, n: i32, mark: &mut Vec<bool>, v: &mut Vec<i32>, vv: &mut Vec<Vec<i32>>) {
        if d == n {
            for i in 0..n {
                for j in (i + 1)..n {
                    if j - i == v[i as usize] - v[j as usize] || i - j == v[i as usize] - v[j as usize] {
                        return;
                    }
                }
            }
            vv.push(v.clone());
            return;
        }
        for i in 0..n {
            if !mark[i as usize] {
                v.push(i);
                mark[i as usize] = true;
                Solution::bt(d + 1, n, mark, v, vv);
                v.pop();
                mark[i as usize] = false;
            }
        }
    }

    fn trans(sss: &mut Vec<Vec<String>>, vv: &Vec<Vec<i32>>) {
        let m = vv.len();
        let n = vv[0].len();
        for i in 0..m {
            let mut ss: Vec<String> = Vec::new();
            for j in 0..n {
                let mut s = String::from_utf8(vec![b'.'; n]).unwrap();
                for k in 0..n {
                    if vv[i][j] == k as i32 {
                        unsafe {s.as_bytes_mut()[k] = b'Q';}
                    }
                }
                ss.push(s);
            }
            sss.push(ss);
        }
    }
}
```

### 6. [Subsets](https://leetcode.com/problems/subsets/)

---

Given an integer array `nums` of **unique** elements, return *all possible subsets (the power set)*.

The solution set **must not** contain duplicate subsets. Return the solution in **any order**.

> A **subset** of an array is a selection of elements (possibly none) of the array.

**Constraints**:

- `1 <= nums.length <= 10`
- `-10 <= nums[i] <= 10`
- All the numbers of `nums` are **unique**.

---

- BT search tree: the i-th layer is choosing `nums[i]` or not.
- BT candidate extension step: no pruning.
- BT boundary: `the searching depth == nums.length`

The code:

```Rust
impl Solution {
    pub fn subsets(nums: Vec<i32>) -> Vec<Vec<i32>> {
        let mut ans: Vec<Vec<i32>> = Vec::new();
        let mut ans_i: Vec<i32> = Vec::new();
        Solution::bt(0, &nums, &mut ans_i, &mut ans);
        ans
    }
    
    fn bt(d: usize, nums: &Vec<i32>, v: &mut Vec<i32>, vv: &mut Vec<Vec<i32>>) {
        let n = nums.len();
        if d == n {
            vv.push(v.clone());
            return;
        }
        Solution::bt(d + 1, nums, v, vv);
        v.push(nums[d]);
        Solution::bt(d + 1, nums, v, vv);
        v.pop();
    }
}
```

### 7. [Word Search](https://leetcode.com/problems/word-search/)

---

Given an `m x n` grid of characters `board` and a string `word`, return *`true` if `word` exists in the grid*.

The word can be constructed from letters of sequentially adjacent cells, where adjacent cells are horizontally or vertically neighboring. The same letter cell may not be used more than once.

**Constraints**:

- `m == board.length`
- `n = board[i].length`
- `1 <= m, n <= 6`
- `1 <= word.length <= 15`
- `board` and `word` consists of only lowercase and uppercase English letters.

---

- BT search tree: the i-th layer is `up`, `down`, `left`, and `right`.
- BT candidate extension step: prune the unsatisfied directions.
- BT boundary: `the searching depth == word.length`

The code:

```Rust
impl Solution {
    pub fn exist(board: Vec<Vec<char>>, word: String) -> bool {
        let first = word.chars().next().unwrap();
        let m = board.len();
        let n = board[0].len();
        let mut visited = vec![vec![false; n]; m];
        let mut flag = false;
        for i in 0..m {
            for j in 0..n {
                if board[i][j] == first {
                    visited[i][j] = true;
                    flag = flag || Self::bt(1, &board, &word, i, j, &mut visited);
                    visited[i][j] = false;
                }
            }
        }
        flag
    }
    
    fn bt(d: usize, board: &Vec<Vec<char>>, word: &String, a: usize, b: usize, mark: &mut Vec<Vec<bool>>) -> bool {
        let m = board.len();
        let n = board[0].len();
        let len = word.len();
        if d == len {
            return true;
        }
        let mut flag = false;
        let dir = [0, 1, 0, -1, 0];
        for i in 0..4 {
            let aa = (a as i32 + dir[i]) as usize;
            let bb = (b as i32 + dir[i + 1]) as usize;
            if aa < m && bb < n {
                if !mark[aa][bb] && board[aa][bb] == word.chars().nth(d).unwrap() {
                    mark[aa][bb] = true;
                    flag = flag || Self::bt(d + 1, board, word, aa, bb, mark);
                    mark[aa][bb] = false;
                }
            }
        }
        flag
    }
}
```

### 8. [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

---

Given a string `s`, partition `s` such that every substring of the partition is a palindrome. Return *all possible palindrome partitioning of `s`*.

> A **substring** is a contiguous non-empty sequence of characters within a string.
>
> A **palindrome** is a string that reads the same forward and backward.

**Constraints**:

- `1 <= s.length <= 16`
- `s` contains only lowercase English letters.

---

- BT search tree: the i-th layer is `s-i.substr(0, l)` and `1 <= l <= s-i.length`.
- BT candidate extension step: prune the un-palindrome `s-i(1)` and further partition `s-i(2)` when `s-i(1)` is a palindrome string.
- BT boundary: `s-i(2).length == 0`

The code:

```Rust
impl Solution {
    pub fn partition(s: String) -> Vec<Vec<String>> {
        let mut ans: Vec<Vec<String>> = Vec::new();
        let mut ans_i: Vec<String> = Vec::new();
        Self::bt(s, &mut ans_i, &mut ans);
        ans
    }
    
    fn bt(s: String, ss: &mut Vec<String>, sss: &mut Vec<Vec<String>>) {
        let n = s.len();
        if n == 0 {
            sss.push(ss.clone());
            return;
        }
        for i in 1..=n {
            let s1 = s[..i].to_string();
            let s2 = s[i..].to_string();
            if Self::is_pal(&s1) {
                ss.push(s1);
                Self::bt(s2, ss, sss);
                ss.pop();
            }
        }
    }
    
    fn is_pal(s: &str) -> bool {
        let n = s.len();
        for i in 0..n/2 {
            if s.chars().nth(i) != s.chars().nth(n - 1 - i) {
                return false;
            }
        }
        true
    }
}
```

## 0x03 Summary

As mentioned above, there are 3 key elements of a BT problem. We can deconstruct its solution into 3 steps.

### 1. BT Search Tree

- Fixed depth and degree: 17-Letter Combinations of a Phone Number, 22-Generate Parentheses, 46-Permutations, 51-N-Queens, 78-Subsets, 79-Word Search.
- Unfixed depth and degree: 39-Combination Sum and 131-Palindrome Partitioning.

### 2. BT Candidate Extension Step

- `visited[]`
- `bool isSatisfied()`

### 3. BT Boundary

If we cannot prune sub-tree at the BT candidate extension step, the constraint satisfaction "pruning" may occur at the BT boundary.
