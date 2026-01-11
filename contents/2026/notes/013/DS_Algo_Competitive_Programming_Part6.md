# Data Structures & Algorithms for Competitive Programming

## Part 6: Greedy Algorithms and Backtracking

---

## Table of Contents

1. [Greedy Algorithms](#1-greedy-algorithms)
2. [Backtracking](#2-backtracking)
3. [Practice Problems](#3-practice-problems)

---

## 1. Greedy Algorithms

### 1.1 Greedy Strategy

**Principle**: Make locally optimal choice at each step, hoping it leads to global optimum.

**When to Use**:
- Problem has optimal substructure
- Greedy choice property (local optimum → global optimum)
- No need to reconsider previous choices

### 1.2 Activity Selection

```python
def activity_selection(start, finish):
    """
    Maximum number of non-overlapping activities
    Time: O(n log n), Space: O(1)
    """
    # Sort by finish time
    activities = sorted(zip(start, finish), key=lambda x: x[1])
    
    selected = [activities[0]]
    last_finish = activities[0][1]
    
    for s, f in activities[1:]:
        if s >= last_finish:
            selected.append((s, f))
            last_finish = f
    
    return selected
```

### 1.3 Fractional Knapsack

```python
def fractional_knapsack(weights, values, capacity):
    """
    Fractional knapsack (can take fraction of items)
    Time: O(n log n), Space: O(n)
    """
    items = [(v / w, w, v) for w, v in zip(weights, values)]
    items.sort(reverse=True)  # Sort by value/weight ratio
    
    total_value = 0
    remaining = capacity
    
    for ratio, weight, value in items:
        if remaining >= weight:
            total_value += value
            remaining -= weight
        else:
            total_value += ratio * remaining
            break
    
    return total_value
```

### 1.4 Minimum Platforms

```python
def min_platforms(arrivals, departures):
    """
    Minimum platforms needed for trains
    Time: O(n log n), Space: O(1)
    """
    arrivals.sort()
    departures.sort()
    
    platforms = 0
    max_platforms = 0
    i = j = 0
    
    while i < len(arrivals):
        if arrivals[i] <= departures[j]:
            platforms += 1
            max_platforms = max(max_platforms, platforms)
            i += 1
        else:
            platforms -= 1
            j += 1
    
    return max_platforms
```

### 1.5 Job Scheduling

```python
def job_scheduling(jobs):
    """
    Schedule jobs to maximize profit (each job has deadline and profit)
    Time: O(n^2), Space: O(n)
    """
    # Sort by profit descending
    jobs.sort(key=lambda x: x[2], reverse=True)
    
    max_deadline = max(job[1] for job in jobs)
    schedule = [None] * (max_deadline + 1)
    total_profit = 0
    
    for job_id, deadline, profit in jobs:
        # Find latest available slot before deadline
        for slot in range(deadline, 0, -1):
            if schedule[slot] is None:
                schedule[slot] = job_id
                total_profit += profit
                break
    
    return total_profit, schedule
```

### 1.6 Huffman Coding

```python
import heapq

class Node:
    def __init__(self, char=None, freq=0, left=None, right=None):
        self.char = char
        self.freq = freq
        self.left = left
        self.right = right
    
    def __lt__(self, other):
        return self.freq < other.freq

def huffman_coding(frequencies):
    """
    Build Huffman tree for optimal prefix coding
    Time: O(n log n), Space: O(n)
    """
    heap = [Node(char, freq) for char, freq in frequencies.items()]
    heapq.heapify(heap)
    
    while len(heap) > 1:
        left = heapq.heappop(heap)
        right = heapq.heappop(heap)
        
        merged = Node(freq=left.freq + right.freq, left=left, right=right)
        heapq.heappush(heap, merged)
    
    root = heap[0]
    codes = {}
    
    def build_codes(node, code=""):
        if node.char:
            codes[node.char] = code
        else:
            build_codes(node.left, code + "0")
            build_codes(node.right, code + "1")
    
    build_codes(root)
    return codes
```

### 1.7 Gas Station

```python
def can_complete_circuit(gas, cost):
    """
    Find starting gas station to complete circuit
    Time: O(n), Space: O(1)
    """
    total_gas = total_cost = 0
    start = 0
    current_gas = 0
    
    for i in range(len(gas)):
        total_gas += gas[i]
        total_cost += cost[i]
        current_gas += gas[i] - cost[i]
        
        if current_gas < 0:
            start = i + 1
            current_gas = 0
    
    return start if total_gas >= total_cost else -1
```

### 1.8 Merge Intervals

```python
def merge_intervals(intervals):
    """
    Merge overlapping intervals
    Time: O(n log n), Space: O(n)
    """
    if not intervals:
        return []
    
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    
    for current in intervals[1:]:
        last = merged[-1]
        if current[0] <= last[1]:
            # Overlapping, merge
            merged[-1] = [last[0], max(last[1], current[1])]
        else:
            merged.append(current)
    
    return merged
```

---

## 2. Backtracking

### 2.1 Backtracking Template

```python
def backtrack_template():
    """
    General backtracking template
    """
    result = []
    
    def backtrack(current_state, remaining_choices):
        # Base case: solution found
        if is_solution(current_state):
            result.append(current_state[:])
            return
        
        # Try all choices
        for choice in remaining_choices:
            # Make choice
            if is_valid(choice, current_state):
                current_state.append(choice)
                # Recurse
                backtrack(current_state, get_remaining_choices(choice))
                # Undo choice (backtrack)
                current_state.pop()
    
    backtrack([], initial_choices)
    return result
```

### 2.2 N-Queens

```python
def solve_n_queens(n):
    """
    Place n queens on n×n board so no two attack each other
    Time: O(n!), Space: O(n)
    """
    result = []
    board = [['.'] * n for _ in range(n)]
    
    def is_safe(row, col):
        # Check column
        for i in range(row):
            if board[i][col] == 'Q':
                return False
        
        # Check diagonal (top-left to bottom-right)
        i, j = row - 1, col - 1
        while i >= 0 and j >= 0:
            if board[i][j] == 'Q':
                return False
            i -= 1
            j -= 1
        
        # Check diagonal (top-right to bottom-left)
        i, j = row - 1, col + 1
        while i >= 0 and j < n:
            if board[i][j] == 'Q':
                return False
            i -= 1
            j += 1
        
        return True
    
    def backtrack(row):
        if row == n:
            result.append([''.join(row) for row in board])
            return
        
        for col in range(n):
            if is_safe(row, col):
                board[row][col] = 'Q'
                backtrack(row + 1)
                board[row][col] = '.'
    
    backtrack(0)
    return result
```

### 2.3 Sudoku Solver

```python
def solve_sudoku(board):
    """
    Solve 9×9 Sudoku puzzle
    Time: O(9^m) where m is empty cells, Space: O(1)
    """
    def is_valid(row, col, num):
        # Check row
        for j in range(9):
            if board[row][j] == num:
                return False
        
        # Check column
        for i in range(9):
            if board[i][col] == num:
                return False
        
        # Check 3×3 box
        box_row = (row // 3) * 3
        box_col = (col // 3) * 3
        for i in range(box_row, box_row + 3):
            for j in range(box_col, box_col + 3):
                if board[i][j] == num:
                    return False
        
        return True
    
    def solve():
        for i in range(9):
            for j in range(9):
                if board[i][j] == '.':
                    for num in '123456789':
                        if is_valid(i, j, num):
                            board[i][j] = num
                            if solve():
                                return True
                            board[i][j] = '.'
                    return False
        return True
    
    solve()
    return board
```

### 2.4 Subsets

```python
def subsets(nums):
    """
    Generate all subsets
    Time: O(2^n), Space: O(2^n)
    """
    result = []
    
    def backtrack(start, current):
        result.append(current[:])
        
        for i in range(start, len(nums)):
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()
    
    backtrack(0, [])
    return result

def subsets_with_dup(nums):
    """
    Subsets with duplicates (avoid duplicates)
    Time: O(2^n), Space: O(2^n)
    """
    nums.sort()
    result = []
    
    def backtrack(start, current):
        result.append(current[:])
        
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:
                continue
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()
    
    backtrack(0, [])
    return result
```

### 2.5 Permutations

```python
def permutations(nums):
    """
    Generate all permutations
    Time: O(n!), Space: O(n!)
    """
    result = []
    
    def backtrack(current):
        if len(current) == len(nums):
            result.append(current[:])
            return
        
        for num in nums:
            if num not in current:
                current.append(num)
                backtrack(current)
                current.pop()
    
    backtrack([])
    return result

def permutations_ii(nums):
    """
    Permutations with duplicates
    Time: O(n!), Space: O(n!)
    """
    nums.sort()
    result = []
    used = [False] * len(nums)
    
    def backtrack(current):
        if len(current) == len(nums):
            result.append(current[:])
            return
        
        for i in range(len(nums)):
            if used[i] or (i > 0 and nums[i] == nums[i - 1] and not used[i - 1]):
                continue
            used[i] = True
            current.append(nums[i])
            backtrack(current)
            current.pop()
            used[i] = False
    
    backtrack([])
    return result
```

### 2.6 Combination Sum

```python
def combination_sum(candidates, target):
    """
    Find all combinations that sum to target (can reuse candidates)
    Time: O(2^target), Space: O(target)
    """
    result = []
    
    def backtrack(remaining, current, start):
        if remaining == 0:
            result.append(current[:])
            return
        if remaining < 0:
            return
        
        for i in range(start, len(candidates)):
            current.append(candidates[i])
            backtrack(remaining - candidates[i], current, i)
            current.pop()
    
    backtrack(target, [], 0)
    return result

def combination_sum_ii(candidates, target):
    """
    Combination sum (each candidate used once, avoid duplicates)
    Time: O(2^n), Space: O(target)
    """
    candidates.sort()
    result = []
    
    def backtrack(remaining, current, start):
        if remaining == 0:
            result.append(current[:])
            return
        if remaining < 0:
            return
        
        for i in range(start, len(candidates)):
            if i > start and candidates[i] == candidates[i - 1]:
                continue
            current.append(candidates[i])
            backtrack(remaining - candidates[i], current, i + 1)
            current.pop()
    
    backtrack(target, [], 0)
    return result
```

### 2.7 Word Search

```python
def exist(board, word):
    """
    Check if word exists in 2D board (can move up, down, left, right)
    Time: O(m * n * 4^L), Space: O(L)
    """
    rows, cols = len(board), len(board[0])
    
    def backtrack(row, col, index):
        if index == len(word):
            return True
        
        if (row < 0 or row >= rows or col < 0 or col >= cols or
            board[row][col] != word[index]):
            return False
        
        # Mark as visited
        temp = board[row][col]
        board[row][col] = '#'
        
        # Try all 4 directions
        found = (backtrack(row + 1, col, index + 1) or
                backtrack(row - 1, col, index + 1) or
                backtrack(row, col + 1, index + 1) or
                backtrack(row, col - 1, index + 1))
        
        # Restore
        board[row][col] = temp
        return found
    
    for i in range(rows):
        for j in range(cols):
            if backtrack(i, j, 0):
                return True
    
    return False
```

### 2.8 Generate Parentheses

```python
def generate_parenthesis(n):
    """
    Generate all valid parentheses combinations
    Time: O(4^n / sqrt(n)), Space: O(4^n / sqrt(n))
    """
    result = []
    
    def backtrack(current, open_count, close_count):
        if len(current) == 2 * n:
            result.append(current)
            return
        
        if open_count < n:
            backtrack(current + '(', open_count + 1, close_count)
        
        if close_count < open_count:
            backtrack(current + ')', open_count, close_count + 1)
    
    backtrack("", 0, 0)
    return result
```

---

## 3. Practice Problems

### Problem Set 1: Greedy

1. **Jump Game**
```python
def can_jump(nums):
    """
    Check if can reach last index (can jump at most nums[i] steps)
    Time: O(n), Space: O(1)
    """
    max_reach = 0
    
    for i, num in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + num)
        if max_reach >= len(nums) - 1:
            return True
    
    return True

def jump_game_ii(nums):
    """
    Minimum jumps to reach last index
    Time: O(n), Space: O(1)
    """
    jumps = 0
    current_end = 0
    farthest = 0
    
    for i in range(len(nums) - 1):
        farthest = max(farthest, i + nums[i])
        
        if i == current_end:
            jumps += 1
            current_end = farthest
    
    return jumps
```

2. **Non-overlapping Intervals**
```python
def erase_overlap_intervals(intervals):
    """
    Minimum intervals to remove for non-overlapping
    Time: O(n log n), Space: O(1)
    """
    if not intervals:
        return 0
    
    intervals.sort(key=lambda x: x[1])
    count = 0
    end = intervals[0][1]
    
    for i in range(1, len(intervals)):
        if intervals[i][0] < end:
            count += 1
        else:
            end = intervals[i][1]
    
    return count
```

### Problem Set 2: Backtracking

1. **Letter Combinations of Phone Number**
```python
def letter_combinations(digits):
    """
    Generate letter combinations from phone number
    Time: O(4^n), Space: O(4^n)
    """
    if not digits:
        return []
    
    mapping = {
        '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
        '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
    }
    
    result = []
    
    def backtrack(index, current):
        if index == len(digits):
            result.append(current)
            return
        
        for letter in mapping[digits[index]]:
            backtrack(index + 1, current + letter)
    
    backtrack(0, "")
    return result
```

2. **Restore IP Addresses**
```python
def restore_ip_addresses(s):
    """
    Restore valid IP addresses from string
    Time: O(1), Space: O(1)
    """
    result = []
    
    def backtrack(index, current, segments):
        if segments == 4:
            if index == len(s):
                result.append(current[:-1])  # Remove last dot
            return
        
        for length in range(1, 4):
            if index + length > len(s):
                break
            
            segment = s[index:index + length]
            
            # Check validity
            if (segment.startswith('0') and len(segment) > 1) or int(segment) > 255:
                continue
            
            backtrack(index + length, current + segment + '.', segments + 1)
    
    backtrack(0, "", 0)
    return result
```

---

## Summary of Part 6

### Greedy Algorithms

- **Strategy**: Local optimum → Global optimum
- **Applications**: Activity selection, scheduling, knapsack
- **Key**: Prove greedy choice property

### Backtracking

- **Strategy**: Try choices, backtrack if invalid
- **Template**: Make choice → Recurse → Undo choice
- **Applications**: N-Queens, Sudoku, Permutations, Combinations

### Next Steps

**Part 7** will cover:
- Advanced Data Structures (Trie, Segment Tree, Fenwick Tree)
- String Algorithms (KMP, Z-algorithm)

---

**Practice greedy and backtracking problems!**

