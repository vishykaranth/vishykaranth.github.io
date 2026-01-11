# Data Structures & Algorithms for Competitive Programming

## Part 5: Dynamic Programming

---

## Table of Contents

1. [DP Fundamentals](#1-dp-fundamentals)
2. [1D Dynamic Programming](#2-1d-dynamic-programming)
3. [2D Dynamic Programming](#3-2d-dynamic-programming)
4. [DP on Strings](#4-dp-on-strings)
5. [DP on Trees](#5-dp-on-trees)
6. [DP Patterns](#6-dp-patterns)
7. [Practice Problems](#7-practice-problems)

---

## 1. DP Fundamentals

### 1.1 What is Dynamic Programming?

**Definition**: Solving complex problems by breaking them into simpler subproblems and storing results to avoid recomputation.

**Key Characteristics**:
1. **Optimal Substructure**: Optimal solution contains optimal solutions to subproblems
2. **Overlapping Subproblems**: Same subproblems are solved multiple times

### 1.2 DP Approaches

**Top-Down (Memoization)**:
```python
def fibonacci_memo(n, memo={}):
    """
    Fibonacci with memoization
    Time: O(n), Space: O(n)
    """
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    
    memo[n] = fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo)
    return memo[n]
```

**Bottom-Up (Tabulation)**:
```python
def fibonacci_tabulation(n):
    """
    Fibonacci with tabulation
    Time: O(n), Space: O(n)
    """
    if n <= 1:
        return n
    
    dp = [0] * (n + 1)
    dp[1] = 1
    
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]
    
    return dp[n]

# Space-optimized
def fibonacci_optimized(n):
    """
    Time: O(n), Space: O(1)
    """
    if n <= 1:
        return n
    
    prev2, prev1 = 0, 1
    
    for i in range(2, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    
    return prev1
```

### 1.3 DP Problem-Solving Steps

1. **Identify State**: What information do we need?
2. **Define DP Array**: What does dp[i] represent?
3. **State Transition**: How to compute dp[i] from previous states?
4. **Base Cases**: What are the initial values?
5. **Answer**: Where is the final answer?

---

## 2. 1D Dynamic Programming

### 2.1 Climbing Stairs

```python
def climb_stairs(n):
    """
    Ways to climb n stairs (1 or 2 steps at a time)
    Time: O(n), Space: O(1)
    """
    if n <= 2:
        return n
    
    prev2, prev1 = 1, 2
    
    for i in range(3, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current
    
    return prev1

def climb_stairs_k(n, k):
    """
    Climb stairs with at most k steps at a time
    Time: O(n * k), Space: O(n)
    """
    dp = [0] * (n + 1)
    dp[0] = 1
    
    for i in range(1, n + 1):
        for j in range(1, min(i, k) + 1):
            dp[i] += dp[i - j]
    
    return dp[n]
```

### 2.2 House Robber

```python
def rob(nums):
    """
    Maximum money without robbing adjacent houses
    Time: O(n), Space: O(1)
    """
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

def rob_circular(nums):
    """
    Houses arranged in circle (first and last are adjacent)
    Time: O(n), Space: O(1)
    """
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]
    
    # Two cases: rob first house or don't
    return max(rob(nums[:-1]), rob(nums[1:]))
```

### 2.3 Coin Change

```python
def coin_change(coins, amount):
    """
    Minimum coins to make amount
    Time: O(amount * len(coins)), Space: O(amount)
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

def coin_change_ways(coins, amount):
    """
    Number of ways to make amount
    Time: O(amount * len(coins)), Space: O(amount)
    """
    dp = [0] * (amount + 1)
    dp[0] = 1
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] += dp[i - coin]
    
    return dp[amount]
```

### 2.4 Longest Increasing Subsequence

```python
def length_of_lis(nums):
    """
    Length of longest increasing subsequence
    Time: O(n^2), Space: O(n)
    """
    if not nums:
        return 0
    
    dp = [1] * len(nums)
    
    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    
    return max(dp)

def length_of_lis_binary_search(nums):
    """
    Optimized using binary search
    Time: O(n log n), Space: O(n)
    """
    import bisect
    
    tails = []
    
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    
    return len(tails)
```

### 2.5 Decode Ways

```python
def num_decodings(s):
    """
    Number of ways to decode string (A=1, B=2, ..., Z=26)
    Time: O(n), Space: O(1)
    """
    if not s or s[0] == '0':
        return 0
    
    n = len(s)
    prev2 = 1  # dp[i-2]
    prev1 = 1  # dp[i-1]
    
    for i in range(1, n):
        current = 0
        
        # Single digit
        if s[i] != '0':
            current += prev1
        
        # Two digits
        two_digit = int(s[i-1:i+1])
        if 10 <= two_digit <= 26:
            current += prev2
        
        prev2 = prev1
        prev1 = current
    
    return prev1
```

---

## 3. 2D Dynamic Programming

### 3.1 Unique Paths

```python
def unique_paths(m, n):
    """
    Number of unique paths from top-left to bottom-right
    Time: O(m * n), Space: O(n)
    """
    dp = [1] * n
    
    for i in range(1, m):
        for j in range(1, n):
            dp[j] += dp[j - 1]
    
    return dp[n - 1]

def unique_paths_with_obstacles(obstacle_grid):
    """
    Unique paths with obstacles
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(obstacle_grid), len(obstacle_grid[0])
    dp = [[0] * n for _ in range(m)]
    
    # Base case
    dp[0][0] = 1 if obstacle_grid[0][0] == 0 else 0
    
    # First row
    for j in range(1, n):
        if obstacle_grid[0][j] == 0:
            dp[0][j] = dp[0][j - 1]
    
    # First column
    for i in range(1, m):
        if obstacle_grid[i][0] == 0:
            dp[i][0] = dp[i - 1][0]
    
    # Fill rest
    for i in range(1, m):
        for j in range(1, n):
            if obstacle_grid[i][j] == 0:
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1]
    
    return dp[m - 1][n - 1]
```

### 3.2 Minimum Path Sum

```python
def min_path_sum(grid):
    """
    Minimum path sum from top-left to bottom-right
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]
    
    # Base case
    dp[0][0] = grid[0][0]
    
    # First row
    for j in range(1, n):
        dp[0][j] = dp[0][j - 1] + grid[0][j]
    
    # First column
    for i in range(1, m):
        dp[i][0] = dp[i - 1][0] + grid[i][0]
    
    # Fill rest
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j]
    
    return dp[m - 1][n - 1]

# Space-optimized
def min_path_sum_optimized(grid):
    """Time: O(m * n), Space: O(n)"""
    m, n = len(grid), len(grid[0])
    dp = [0] * n
    dp[0] = grid[0][0]
    
    for j in range(1, n):
        dp[j] = dp[j - 1] + grid[0][j]
    
    for i in range(1, m):
        dp[0] += grid[i][0]
        for j in range(1, n):
            dp[j] = min(dp[j], dp[j - 1]) + grid[i][j]
    
    return dp[n - 1]
```

### 3.3 Edit Distance

```python
def min_distance(word1, word2):
    """
    Minimum operations to convert word1 to word2
    Operations: insert, delete, replace
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Base cases
    for i in range(m + 1):
        dp[i][0] = i  # Delete all characters
    for j in range(n + 1):
        dp[0][j] = j  # Insert all characters
    
    # Fill DP table
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

### 3.4 Longest Common Subsequence

```python
def longest_common_subsequence(text1, text2):
    """
    Length of longest common subsequence
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    return dp[m][n]

def lcs_sequence(text1, text2):
    """Return actual LCS string"""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i - 1] == text2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    
    # Reconstruct LCS
    i, j = m, n
    lcs = []
    
    while i > 0 and j > 0:
        if text1[i - 1] == text2[j - 1]:
            lcs.append(text1[i - 1])
            i -= 1
            j -= 1
        elif dp[i - 1][j] > dp[i][j - 1]:
            i -= 1
        else:
            j -= 1
    
    return ''.join(reversed(lcs))
```

---

## 4. DP on Strings

### 4.1 Longest Palindromic Substring

```python
def longest_palindrome(s):
    """
    Longest palindromic substring
    Time: O(n^2), Space: O(n^2)
    """
    n = len(s)
    dp = [[False] * n for _ in range(n)]
    start = 0
    max_len = 1
    
    # Single character palindromes
    for i in range(n):
        dp[i][i] = True
    
    # Two character palindromes
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            dp[i][i + 1] = True
            start = i
            max_len = 2
    
    # Palindromes of length 3 and more
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and dp[i + 1][j - 1]:
                dp[i][j] = True
                start = i
                max_len = length
    
    return s[start:start + max_len]

# Space-optimized
def longest_palindrome_optimized(s):
    """Time: O(n^2), Space: O(1)"""
    def expand_around_center(left, right):
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        return right - left - 1
    
    start = 0
    max_len = 0
    
    for i in range(len(s)):
        # Odd length palindromes
        len1 = expand_around_center(i, i)
        # Even length palindromes
        len2 = expand_around_center(i, i + 1)
        
        length = max(len1, len2)
        if length > max_len:
            max_len = length
            start = i - (length - 1) // 2
    
    return s[start:start + max_len]
```

### 4.2 Palindrome Partitioning

```python
def min_cut(s):
    """
    Minimum cuts to partition string into palindromes
    Time: O(n^2), Space: O(n^2)
    """
    n = len(s)
    # Precompute palindrome table
    is_palindrome = [[False] * n for _ in range(n)]
    
    for i in range(n):
        is_palindrome[i][i] = True
    
    for i in range(n - 1):
        if s[i] == s[i + 1]:
            is_palindrome[i][i + 1] = True
    
    for length in range(3, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j] and is_palindrome[i + 1][j - 1]:
                is_palindrome[i][j] = True
    
    # DP for minimum cuts
    dp = [0] * n
    
    for i in range(n):
        if is_palindrome[0][i]:
            dp[i] = 0
        else:
            dp[i] = i
            for j in range(i):
                if is_palindrome[j + 1][i]:
                    dp[i] = min(dp[i], dp[j] + 1)
    
    return dp[n - 1]
```

### 4.3 Regular Expression Matching

```python
def is_match(s, p):
    """
    Match string with pattern (supports '.' and '*')
    '.' matches any single character
    '*' matches zero or more of preceding element
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    
    # Empty string matches empty pattern
    dp[0][0] = True
    
    # Handle patterns like a*, a*b*, etc.
    for j in range(2, n + 1):
        if p[j - 1] == '*':
            dp[0][j] = dp[0][j - 2]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j - 1] == '*':
                # Zero occurrences
                dp[i][j] = dp[i][j - 2]
                # One or more occurrences
                if p[j - 2] == s[i - 1] or p[j - 2] == '.':
                    dp[i][j] = dp[i][j] or dp[i - 1][j]
            elif p[j - 1] == '.' or p[j - 1] == s[i - 1]:
                dp[i][j] = dp[i - 1][j - 1]
    
    return dp[m][n]
```

---

## 5. DP on Trees

### 5.1 House Robber III

```python
def rob_tree(root):
    """
    Maximum money without robbing connected nodes
    Time: O(n), Space: O(h)
    """
    def dfs(node):
        if not node:
            return (0, 0)  # (rob, not_rob)
        
        left_rob, left_not_rob = dfs(node.left)
        right_rob, right_not_rob = dfs(node.right)
        
        # Rob current node
        rob = node.val + left_not_rob + right_not_rob
        
        # Don't rob current node
        not_rob = max(left_rob, left_not_rob) + max(right_rob, right_not_rob)
        
        return (rob, not_rob)
    
    rob, not_rob = dfs(root)
    return max(rob, not_rob)
```

### 5.2 Binary Tree Maximum Path Sum

```python
def max_path_sum(root):
    """
    Maximum path sum (path can start and end anywhere)
    Time: O(n), Space: O(h)
    """
    max_sum = [float('-inf')]
    
    def dfs(node):
        if not node:
            return 0
        
        # Max gain from left and right (can be negative)
        left_gain = max(dfs(node.left), 0)
        right_gain = max(dfs(node.right), 0)
        
        # Path through this node
        current_path = node.val + left_gain + right_gain
        max_sum[0] = max(max_sum[0], current_path)
        
        # Return max gain for parent
        return node.val + max(left_gain, right_gain)
    
    dfs(root)
    return max_sum[0]
```

---

## 6. DP Patterns

### 6.1 Knapsack Problems

**0/1 Knapsack**:
```python
def knapsack_01(weights, values, capacity):
    """
    0/1 Knapsack: Each item can be taken at most once
    Time: O(n * capacity), Space: O(capacity)
    """
    n = len(weights)
    dp = [0] * (capacity + 1)
    
    for i in range(n):
        for w in range(capacity, weights[i] - 1, -1):
            dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[capacity]
```

**Unbounded Knapsack**:
```python
def knapsack_unbounded(weights, values, capacity):
    """
    Unbounded Knapsack: Each item can be taken multiple times
    Time: O(n * capacity), Space: O(capacity)
    """
    n = len(weights)
    dp = [0] * (capacity + 1)
    
    for w in range(1, capacity + 1):
        for i in range(n):
            if weights[i] <= w:
                dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
    
    return dp[capacity]
```

### 6.2 Stock Problems

**Best Time to Buy and Sell Stock**:
```python
def max_profit(prices):
    """
    One transaction allowed
    Time: O(n), Space: O(1)
    """
    min_price = float('inf')
    max_profit = 0
    
    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)
    
    return max_profit

def max_profit_multiple(prices):
    """
    Multiple transactions allowed
    Time: O(n), Space: O(1)
    """
    profit = 0
    for i in range(1, len(prices)):
        if prices[i] > prices[i - 1]:
            profit += prices[i] - prices[i - 1]
    return profit

def max_profit_with_cooldown(prices):
    """
    Multiple transactions with cooldown
    Time: O(n), Space: O(1)
    """
    sold = 0
    hold = float('-inf')
    rest = 0
    
    for price in prices:
        prev_sold = sold
        sold = hold + price
        hold = max(hold, rest - price)
        rest = max(rest, prev_sold)
    
    return max(sold, rest)
```

---

## 7. Practice Problems

### Problem Set 1: 1D DP

1. **Word Break**
```python
def word_break(s, word_dict):
    """
    Check if string can be segmented into dictionary words
    Time: O(n^2), Space: O(n)
    """
    word_set = set(word_dict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break
    
    return dp[n]
```

2. **Partition Equal Subset Sum**
```python
def can_partition(nums):
    """
    Check if array can be partitioned into two equal sum subsets
    Time: O(n * sum), Space: O(sum)
    """
    total = sum(nums)
    if total % 2 != 0:
        return False
    
    target = total // 2
    dp = [False] * (target + 1)
    dp[0] = True
    
    for num in nums:
        for j in range(target, num - 1, -1):
            dp[j] = dp[j] or dp[j - num]
    
    return dp[target]
```

### Problem Set 2: 2D DP

1. **Interleaving String**
```python
def is_interleave(s1, s2, s3):
    """
    Check if s3 is interleaving of s1 and s2
    Time: O(m * n), Space: O(m * n)
    """
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False
    
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    
    for i in range(m + 1):
        for j in range(n + 1):
            if i > 0 and s1[i - 1] == s3[i + j - 1]:
                dp[i][j] = dp[i][j] or dp[i - 1][j]
            if j > 0 and s2[j - 1] == s3[i + j - 1]:
                dp[i][j] = dp[i][j] or dp[i][j - 1]
    
    return dp[m][n]
```

---

## Summary of Part 5

### Key DP Concepts

1. **Memoization vs Tabulation**: Top-down vs bottom-up
2. **1D DP**: Linear problems (stairs, coins, LIS)
3. **2D DP**: Grid problems, string matching
4. **DP on Strings**: Palindrome, matching, partitioning
5. **DP on Trees**: Tree DP with state

### Common Patterns

- **Knapsack**: 0/1, unbounded, subset sum
- **Stock Problems**: Buy/sell with constraints
- **String DP**: Edit distance, LCS, matching
- **Grid DP**: Path problems, obstacles

### Next Steps

**Part 6** will cover:
- Greedy Algorithms
- Backtracking
- Divide and Conquer

---

**Practice DP problems - they're crucial for competitive programming!**

