# Core DS/Algo Patterns: Solve Almost Any Problem

## Part 3: Dynamic Programming Patterns

---

## Table of Contents

1. [DP Core Logic](#1-dp-core-logic)
2. [1D DP Patterns](#2-1d-dp-patterns)
3. [2D DP Patterns](#3-2d-dp-patterns)
4. [DP on Strings](#4-dp-on-strings)
5. [DP State Transition Patterns](#5-dp-state-transition-patterns)
6. [DP Optimization Patterns](#6-dp-optimization-patterns)
7. [Master DP Recognition](#7-master-dp-recognition)

---

## 1. DP Core Logic

### 1.1 When to Use DP

**Key Indicators**:
- Optimal substructure (optimal solution contains optimal subproblems)
- Overlapping subproblems (same subproblems solved multiple times)
- Maximization/minimization problems
- Counting problems
- "How many ways" problems

**Decision Framework**:
```python
"""
Is it a DP problem?

1. Can problem be broken into subproblems?
   YES → Continue
   NO → Not DP

2. Do subproblems overlap?
   YES → DP
   NO → Divide & Conquer

3. Need optimal solution?
   YES → DP or Greedy
   NO → Backtracking

4. Can use previous results?
   YES → DP
   NO → Not DP
"""
```

### 1.2 Universal DP Template

**Step-by-Step Process**:
```python
def dp_template(problem_input):
    """
    Universal DP template
    """
    # Step 1: Identify state
    # What information do we need? → dp[i] represents what?
    
    # Step 2: Define DP array
    n = len(problem_input)
    dp = [0] * (n + 1)  # or 2D: [[0] * m for _ in range(n)]
    
    # Step 3: Base cases
    dp[0] = base_value
    # dp[1] = base_value_2
    
    # Step 4: State transition
    for i in range(1, n + 1):
        dp[i] = transition_function(dp, i, problem_input)
    
    # Step 5: Return answer
    return dp[n]  # or dp[m][n] for 2D
```

### 1.3 DP Approaches

**Top-Down (Memoization)**:
```python
def dp_memoization(n, memo={}):
    """Recursive with memoization"""
    if n in memo:
        return memo[n]
    
    if n <= base_case:
        return base_value
    
    memo[n] = dp_memoization(n - 1, memo) + dp_memoization(n - 2, memo)
    return memo[n]
```

**Bottom-Up (Tabulation)**:
```python
def dp_tabulation(n):
    """Iterative tabulation"""
    dp = [0] * (n + 1)
    dp[0] = base_value
    dp[1] = base_value
    
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    
    return dp[n]
```

---

## 2. 1D DP Patterns

### 2.1 Pattern 1: Linear DP

**Template**:
```python
def linear_dp(arr):
    """dp[i] = solution up to index i"""
    n = len(arr)
    dp = [0] * n
    dp[0] = base_value
    
    for i in range(1, n):
        dp[i] = transition(dp, i, arr)
    
    return dp[n - 1]
```

**Example: Climbing Stairs**
```python
def climb_stairs(n):
    """Ways to climb n stairs (1 or 2 steps)"""
    if n <= 2:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    dp[2] = 2
    
    for i in range(3, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    
    return dp[n]

# Space-optimized
def climb_stairs_optimized(n):
    if n <= 2:
        return n
    
    prev2, prev1 = 1, 2
    for i in range(3, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    
    return prev1
```

### 2.2 Pattern 2: House Robber Pattern

**Template**:
```python
def house_robber_pattern(nums):
    """Choose or skip pattern"""
    n = len(nums)
    if n == 0:
        return 0
    if n == 1:
        return nums[0]
    
    # dp[i] = maximum value up to house i
    dp = [0] * n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    
    for i in range(2, n):
        # Choose: rob current + best up to i-2
        # Skip: best up to i-1
        dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
    
    return dp[n - 1]

# Space-optimized
def house_robber_optimized(nums):
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]
    
    prev2 = nums[0]
    prev1 = max(nums[0], nums[1])
    
    for i in range(2, len(nums)):
        current = max(prev1, prev2 + nums[i])
        prev2 = prev1
        prev1 = current
    
    return prev1
```

### 2.3 Pattern 3: Coin Change Pattern

**Template**:
```python
def coin_change_pattern(coins, amount):
    """Minimum coins to make amount"""
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

def coin_change_ways(coins, amount):
    """Number of ways to make amount"""
    dp = [0] * (amount + 1)
    dp[0] = 1
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] += dp[i - coin]
    
    return dp[amount]
```

### 2.4 Pattern 4: Longest Increasing Subsequence

**Template**:
```python
def lis_pattern(nums):
    """Length of longest increasing subsequence"""
    n = len(nums)
    dp = [1] * n  # dp[i] = LIS ending at i
    
    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

# Optimized with binary search
import bisect

def lis_optimized(nums):
    """O(n log n) solution"""
    tails = []
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)
```

---

## 3. 2D DP Patterns

### 3.1 Pattern 1: Grid DP

**Template**:
```python
def grid_dp_pattern(grid):
    """DP on 2D grid"""
    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]
    
    # Base cases (first row and column)
    dp[0][0] = grid[0][0]
    for i in range(1, m):
        dp[i][0] = dp[i - 1][0] + grid[i][0]
    for j in range(1, n):
        dp[0][j] = dp[0][j - 1] + grid[0][j]
    
    # Fill rest
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j]
    
    return dp[m - 1][n - 1]
```

**Example: Unique Paths**
```python
def unique_paths(m, n):
    """Number of unique paths"""
    dp = [[1] * n for _ in range(m)]
    
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
    
    return dp[m - 1][n - 1]

# Space-optimized
def unique_paths_optimized(m, n):
    dp = [1] * n
    
    for i in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]
    
    return dp[n - 1]
```

### 3.2 Pattern 2: String DP

**Template**:
```python
def string_dp_pattern(s1, s2):
    """DP on two strings"""
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Base cases
    for i in range(m + 1):
        dp[i][0] = base_value
    for j in range(n + 1):
        dp[0][j] = base_value
    
    # Fill DP table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i - 1] == s2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1  # or other operation
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    return dp[m][n]
```

**Example: Longest Common Subsequence**
```python
def longest_common_subsequence(text1, text2):
    """LCS length"""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    return dp[m][n]
```

**Example: Edit Distance**
```python
def edit_distance(word1, word2):
    """Minimum operations to convert word1 to word2"""
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Base cases
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i - 1] == word2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(
                    dp[i - 1][j],      # Delete
                    dp[i][j - 1],      # Insert
                    dp[i - 1][j - 1]   # Replace
                )
    
    return dp[m][n]
```

---

## 4. DP on Strings

### 4.1 Pattern: Palindrome DP

**Template**:
```python
def palindrome_dp_pattern(s):
    """DP for palindrome problems"""
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    
    # Single character palindromes
    for i in range(n):
        dp[i][i] = True
    
    # Two character palindromes
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
    
    # Palindromes of length 3+
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
    
    return dp
```

**Example: Longest Palindromic Substring**
```python
def longest_palindrome(s):
    """Longest palindromic substring"""
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start = 0
    max_len = 1
    
    # Single character
    for i in range(n):
        dp[i][i] = True
    
    # Two characters
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
            start = i
            max_len = 2
    
    # Length 3+
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
                start = i
                max_len = length
    
    return s[start:start + max_len]
```

### 4.2 Pattern: Word Break DP

**Template**:
```python
def word_break_dp(s, word_dict):
    """Check if string can be segmented"""
    word_set = set(word_dict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True  # Empty string
    
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[n]
```

---

## 5. DP State Transition Patterns

### 5.1 Common State Transitions

**Pattern 1: Take or Skip**
```python
# dp[i] = max(take, skip)
dp[i] = max(dp[i - 1], dp[i - 2] + value[i])
```

**Pattern 2: Multiple Choices**
```python
# dp[i] = best of multiple options
dp[i] = min(dp[i - 1], dp[i - 2], dp[i - 3]) + cost[i]
```

**Pattern 3: Dependent on Previous States**
```python
# dp[i] depends on dp[i-1], dp[i-2], etc.
dp[i] = dp[i - 1] + dp[i - 2]  # Fibonacci-like
```

**Pattern 4: 2D State Transition**
```python
# dp[i][j] depends on multiple previous states
dp[i][j] = max(
    dp[i - 1][j],           # From above
    dp[i][j - 1],           # From left
    dp[i - 1][j - 1] + 1    # From diagonal
)
```

### 5.2 State Design Patterns

**Pattern 1: Single State**
```python
# dp[i] = solution up to index i
dp = [0] * n
```

**Pattern 2: Two States**
```python
# dp[i][0] = state 0 at i
# dp[i][1] = state 1 at i
dp = [[0] * 2 for _ in range(n)]
```

**Pattern 3: Multiple States**
```python
# dp[i][state] = solution at i with state
dp = [[0] * num_states for _ in range(n)]
```

---

## 6. DP Optimization Patterns

### 6.1 Space Optimization

**Pattern: Reduce 2D to 1D**
```python
# Original 2D
dp = [[0] * n for _ in range(m)]

# Optimized 1D
dp = [0] * n
for i in range(m):
    new_dp = [0] * n
    for j in range(n):
        new_dp[j] = transition(dp, i, j)
    dp = new_dp
```

**Pattern: Reduce 1D to O(1)**
```python
# Original 1D
dp = [0] * n

# Optimized O(1)
prev2, prev1 = 0, 1
for i in range(2, n):
    current = prev1 + prev2
    prev2 = prev1
    prev1 = current
```

### 6.2 Time Optimization

**Pattern: Binary Search in DP**
```python
# LIS optimization
import bisect

def lis_optimized(nums):
    tails = []
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    return len(tails)
```

---

## 7. Master DP Recognition

### 7.1 DP Problem Identification

```python
"""
DP Problem Checklist:

1. Can problem be broken into subproblems?
   ✓ YES → Continue
   ✗ NO → Not DP

2. Do subproblems overlap?
   ✓ YES → DP
   ✗ NO → Divide & Conquer

3. Optimal substructure?
   ✓ YES → DP
   ✗ NO → Not DP

4. Need to count/optimize?
   ✓ YES → DP
   ✗ NO → Maybe not DP

5. Can use previous results?
   ✓ YES → DP
   ✗ NO → Not DP
"""

def is_dp_problem(problem_description):
    """
    Identify if problem is DP
    """
    dp_keywords = [
        'maximum', 'minimum', 'optimal',
        'how many ways', 'count',
        'longest', 'shortest',
        'can be formed', 'possible'
    ]
    
    return any(keyword in problem_description.lower() 
               for keyword in dp_keywords)
```

### 7.2 DP Pattern Selection

```python
"""
DP Pattern Selection:

1. 1D Array → 1D DP
   - dp[i] = solution up to i
   - Examples: Climbing Stairs, House Robber

2. 2D Grid → 2D DP
   - dp[i][j] = solution at (i, j)
   - Examples: Unique Paths, Minimum Path Sum

3. Two Strings → 2D DP
   - dp[i][j] = solution for s1[0:i] and s2[0:j]
   - Examples: LCS, Edit Distance

4. String + Constraints → 2D DP
   - dp[i][state] = solution at i with state
   - Examples: Word Break, Palindrome Partitioning

5. Tree → Tree DP
   - Return multiple values from recursion
   - Examples: House Robber III, Max Path Sum
"""
```

### 7.3 Universal DP Template

```python
def universal_dp_solver(problem_input):
    """
    Universal DP solver template
    """
    # Step 1: Identify state
    # What does dp[i] or dp[i][j] represent?
    
    # Step 2: Define DP array
    # 1D: dp = [0] * n
    # 2D: dp = [[0] * m for _ in range(n)]
    
    # Step 3: Base cases
    # dp[0] = ?
    # dp[0][0] = ?
    
    # Step 4: State transition
    # How to compute dp[i] from previous states?
    # dp[i] = transition(dp[i-1], dp[i-2], ...)
    
    # Step 5: Return answer
    # return dp[n] or dp[m][n]
    
    pass
```

---

## Summary: DP Patterns

### Core DP Patterns

1. **1D DP**: Linear problems, choose/skip
2. **2D DP**: Grid problems, two strings
3. **String DP**: Palindrome, word break
4. **Tree DP**: Multiple return values
5. **State Transitions**: Take/skip, multiple choices

### DP Problem Recognition

```
Problem Type → DP Pattern:
- Maximize/Minimize → 1D/2D DP
- Count Ways → 1D/2D DP
- Longest/Shortest → 1D/2D DP
- Two Strings → 2D DP
- Grid → 2D DP
- Tree Optimization → Tree DP
```

### Key Takeaways

1. **Identify State**: What does dp[i] represent?
2. **Base Cases**: Initial values
3. **State Transition**: How to compute from previous states
4. **Optimize Space**: Reduce dimensions when possible
5. **Practice**: DP requires pattern recognition

### Next Steps

**Part 4** will cover:
- Backtracking Patterns
- Greedy Patterns
- When to use each approach

---

**Master these DP patterns to solve optimization and counting problems!**

