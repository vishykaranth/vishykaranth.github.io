# Core DS/Algo Patterns: Solve Almost Any Problem

## Part 4: Backtracking and Greedy Patterns

---

## Table of Contents

1. [Backtracking Core Logic](#1-backtracking-core-logic)
2. [Backtracking Patterns](#2-backtracking-patterns)
3. [Greedy Core Logic](#3-greedy-core-logic)
4. [Greedy Patterns](#4-greedy-patterns)
5. [When to Use Each](#5-when-to-use-each)
6. [Master Pattern Selection](#6-master-pattern-selection)

---

## 1. Backtracking Core Logic

### 1.1 When to Use Backtracking

**Key Indicators**:
- Need to try all possibilities
- Generate all combinations/permutations
- Constraint satisfaction problems
- "Find all" problems
- Problems with choices and constraints

**Decision Framework**:
```python
"""
Is it a Backtracking problem?

1. Need to try all possibilities?
   YES → Backtracking or Brute Force
   NO → Not Backtracking

2. Have constraints to satisfy?
   YES → Backtracking
   NO → Maybe not

3. Need to generate all solutions?
   YES → Backtracking
   NO → Maybe DP or Greedy

4. Can prune invalid paths early?
   YES → Backtracking
   NO → Brute Force
"""
```

### 1.2 Universal Backtracking Template

**Core Template**:
```python
def backtracking_template(choices):
    """
    Universal backtracking template
    """
    result = []
    
    def backtrack(current_state, remaining_choices):
        # Base case: solution found
        if is_solution(current_state):
            result.append(current_state[:])  # Copy, not reference
            return
        
        # Try all choices
        for choice in remaining_choices:
            # Check if choice is valid
            if is_valid(choice, current_state):
                # Make choice
                current_state.append(choice)
                
                # Recurse with updated state
                backtrack(current_state, get_new_choices(choice))
                
                # Undo choice (backtrack)
                current_state.pop()
    
    backtrack([], choices)
    return result
```

### 1.3 Key Components

**Components**:
1. **State**: Current partial solution
2. **Choices**: Available options at each step
3. **Constraints**: Rules that must be satisfied
4. **Goal**: Complete solution
5. **Backtrack**: Undo choice when path doesn't lead to solution

---

## 2. Backtracking Patterns

### 2.1 Pattern 1: Permutations

**Template**:
```python
def permutations_pattern(nums):
    """Generate all permutations"""
    result = []
    
    def backtrack(current):
        # Base case
        if len(current) == len(nums):
            result.append(current[:])
            return
        
        # Try all choices
        for num in nums:
            if num not in current:  # Constraint: no duplicates
                current.append(num)
                backtrack(current)
                current.pop()
    
    backtrack([])
    return result

# Optimized with used array
def permutations_optimized(nums):
    result = []
    used = [False] * len(nums)
    
    def backtrack(current):
        if len(current) == len(nums):
            result.append(current[:])
            return
        
        for i in range(len(nums)):
            if not used[i]:
                used[i] = True
                current.append(nums[i])
                backtrack(current)
                current.pop()
                used[i] = False
    
    backtrack([])
    return result
```

### 2.2 Pattern 2: Combinations

**Template**:
```python
def combinations_pattern(n, k):
    """Generate all combinations of k from n"""
    result = []
    
    def backtrack(current, start):
        # Base case
        if len(current) == k:
            result.append(current[:])
            return
        
        # Try choices from start to n
        for i in range(start, n + 1):
            current.append(i)
            backtrack(current, i + 1)  # Next start is i+1
            current.pop()
    
    backtrack([], 1)
    return result

# Subsets (all combinations)
def subsets_pattern(nums):
    """Generate all subsets"""
    result = []
    
    def backtrack(current, start):
        result.append(current[:])  # Add current subset
        
        for i in range(start, len(nums)):
            current.append(nums[i])
            backtrack(current, i + 1)
            current.pop()
    
    backtrack([], 0)
    return result
```

### 2.3 Pattern 3: Constraint Satisfaction

**Template**:
```python
def constraint_satisfaction_template():
    """Backtracking with constraints"""
    result = []
    
    def backtrack(current_state):
        # Check if current state violates constraints
        if violates_constraints(current_state):
            return  # Prune this path
        
        # Base case
        if is_complete(current_state):
            result.append(current_state[:])
            return
        
        # Try all choices
        for choice in get_choices(current_state):
            if is_valid_choice(choice, current_state):
                # Make choice
                apply_choice(current_state, choice)
                
                # Recurse
                backtrack(current_state)
                
                # Undo choice
                undo_choice(current_state, choice)
    
    backtrack(initial_state)
    return result
```

**Example: N-Queens**
```python
def solve_n_queens(n):
    """Place n queens on n×n board"""
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

### 2.4 Pattern 4: Combination Sum

**Template**:
```python
def combination_sum_pattern(candidates, target):
    """Find combinations that sum to target"""
    result = []
    
    def backtrack(current, remaining, start):
        if remaining == 0:
            result.append(current[:])
            return
        
        if remaining < 0:
            return  # Prune
        
        for i in range(start, len(candidates)):
            current.append(candidates[i])
            backtrack(current, remaining - candidates[i], i)  # Can reuse
            current.pop()
    
    backtrack([], target, 0)
    return result

# Without reusing (each candidate used once)
def combination_sum_no_reuse(candidates, target):
    result = []
    candidates.sort()  # Sort to handle duplicates
    
    def backtrack(current, remaining, start):
        if remaining == 0:
            result.append(current[:])
            return
        
        if remaining < 0:
            return
        
        for i in range(start, len(candidates)):
            # Skip duplicates
            if i > start and candidates[i] == candidates[i - 1]:
                continue
            
            current.append(candidates[i])
            backtrack(current, remaining - candidates[i], i + 1)
            current.pop()
    
    backtrack([], target, 0)
    return result
```

### 2.5 Pattern 5: Word Search

**Template**:
```python
def word_search_pattern(board, word):
    """Find word in 2D board"""
    rows, cols = len(board), len(board[0])
    
    def backtrack(row, col, index):
        # Base case
        if index == len(word):
            return True
        
        # Check bounds and character match
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

---

## 3. Greedy Core Logic

### 3.1 When to Use Greedy

**Key Indicators**:
- Optimal substructure
- Greedy choice property (local optimum → global optimum)
- No need to reconsider previous choices
- Optimization problems

**Decision Framework**:
```python
"""
Is it a Greedy problem?

1. Need optimal solution?
   YES → Greedy or DP
   NO → Not Greedy

2. Can make locally optimal choice?
   YES → Continue
   NO → Not Greedy

3. Local optimum leads to global optimum?
   YES → Greedy
   NO → DP

4. Don't need to reconsider choices?
   YES → Greedy
   NO → Backtracking
"""
```

### 3.2 Universal Greedy Template

**Core Template**:
```python
def greedy_template(choices):
    """
    Universal greedy template
    """
    # Sort choices by some criteria
    choices.sort(key=lambda x: sort_criteria(x))
    
    result = []
    current_state = initial_state
    
    for choice in choices:
        # Check if choice is valid
        if is_valid_choice(choice, current_state):
            # Make greedy choice
            result.append(choice)
            current_state = update_state(current_state, choice)
    
    return result
```

### 3.3 Greedy Strategy

**Steps**:
1. **Sort**: Sort choices by criteria (value/weight, deadline, etc.)
2. **Select**: Make locally optimal choice
3. **Update**: Update state
4. **Repeat**: Continue until goal reached

---

## 4. Greedy Patterns

### 4.1 Pattern 1: Activity Selection

**Template**:
```python
def activity_selection_pattern(start, finish):
    """Select maximum non-overlapping activities"""
    # Sort by finish time
    activities = sorted(zip(start, finish), key=lambda x: x[1])
    
    selected = [activities[0]]
    last_finish = activities[0][1]
    
    for s, f in activities[1:]:
        if s >= last_finish:  # No overlap
            selected.append((s, f))
            last_finish = f
    
    return selected
```

### 4.2 Pattern 2: Interval Problems

**Template**:
```python
def merge_intervals_pattern(intervals):
    """Merge overlapping intervals"""
    if not intervals:
        return []
    
    # Sort by start time
    intervals.sort(key=lambda x: x[0])
    merged = [intervals[0]]
    
    for current in intervals[1:]:
        last = merged[-1]
        if current[0] <= last[1]:  # Overlapping
            merged[-1] = [last[0], max(last[1], current[1])]
        else:
            merged.append(current)
    
    return merged

def erase_overlap_intervals(intervals):
    """Remove minimum intervals for non-overlapping"""
    if not intervals:
        return 0
    
    # Sort by end time
    intervals.sort(key=lambda x: x[1])
    count = 0
    end = intervals[0][1]
    
    for start, finish in intervals[1:]:
        if start < end:  # Overlapping
            count += 1
        else:
            end = finish
    
    return count
```

### 4.3 Pattern 3: Fractional Knapsack

**Template**:
```python
def fractional_knapsack_pattern(weights, values, capacity):
    """Fractional knapsack (can take fraction of items)"""
    # Sort by value/weight ratio
    items = [(v / w, w, v) for w, v in zip(weights, values)]
    items.sort(reverse=True)
    
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

### 4.4 Pattern 4: Jump Game

**Template**:
```python
def jump_game_pattern(nums):
    """Check if can reach last index"""
    max_reach = 0
    
    for i, num in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + num)
        if max_reach >= len(nums) - 1:
            return True
    
    return True

def jump_game_ii_pattern(nums):
    """Minimum jumps to reach last index"""
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

### 4.5 Pattern 5: Gas Station

**Template**:
```python
def gas_station_pattern(gas, cost):
    """Find starting gas station to complete circuit"""
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

---

## 5. When to Use Each

### 5.1 Backtracking vs DP vs Greedy

**Decision Matrix**:
```python
"""
Problem Type → Approach:

1. Need all solutions?
   YES → Backtracking
   NO → Continue

2. Need optimal solution?
   YES → Continue
   NO → Not optimization problem

3. Can make greedy choice?
   YES → Greedy
   NO → Continue

4. Overlapping subproblems?
   YES → DP
   NO → Divide & Conquer

5. Need to try all possibilities?
   YES → Backtracking
   NO → Not Backtracking
"""
```

### 5.2 Comparison Table

| Feature | Backtracking | DP | Greedy |
|---------|-------------|----|----|
| **Solutions** | All solutions | Optimal solution | Optimal solution |
| **Time** | Exponential | Polynomial | Polynomial |
| **Space** | O(depth) | O(n) or O(n²) | O(1) |
| **Reconsider** | Yes (backtrack) | No (memoized) | No |
| **Use When** | All possibilities | Overlapping subproblems | Greedy choice property |

---

## 6. Master Pattern Selection

### 6.1 Problem Analysis Framework

```python
def select_approach(problem_description):
    """
    Select appropriate approach based on problem
    """
    # Step 1: Understand problem
    # - What is asked? (all solutions, optimal, count, etc.)
    # - What are constraints?
    # - What is input format?
    
    # Step 2: Identify pattern
    if "all" in problem_description.lower():
        return "Backtracking"
    
    if "maximum" in problem_description.lower() or "minimum" in problem_description.lower():
        # Check if greedy applies
        if has_greedy_choice_property(problem):
            return "Greedy"
        else:
            return "DP"
    
    if "count" in problem_description.lower() or "how many" in problem_description.lower():
        return "DP"
    
    if "generate" in problem_description.lower() or "find all" in problem_description.lower():
        return "Backtracking"
    
    return "Unknown"
```

### 6.2 Universal Problem Solver

```python
def solve_problem(input_data, problem_type):
    """
    Universal problem solver based on type
    """
    if problem_type == "Backtracking":
        return backtracking_solver(input_data)
    elif problem_type == "DP":
        return dp_solver(input_data)
    elif problem_type == "Greedy":
        return greedy_solver(input_data)
    elif problem_type == "Two Pointers":
        return two_pointers_solver(input_data)
    elif problem_type == "Sliding Window":
        return sliding_window_solver(input_data)
    # ... other patterns
    
    return None
```

### 6.3 Pattern Recognition Checklist

```python
"""
Pattern Recognition Checklist:

1. Subarray/Substring?
   → Sliding Window

2. Sorted Array + Pairs?
   → Two Pointers

3. Need all solutions?
   → Backtracking

4. Need optimal solution + overlapping subproblems?
   → DP

5. Need optimal solution + greedy choice property?
   → Greedy

6. Tree/Graph problem?
   → DFS/BFS

7. Range queries?
   → Prefix Sum

8. Fast lookup?
   → Hash Table

9. Matching pairs?
   → Stack
"""
```

---

## Summary: Backtracking and Greedy

### Core Patterns

**Backtracking**:
1. Permutations
2. Combinations/Subsets
3. Constraint Satisfaction (N-Queens, Sudoku)
4. Combination Sum
5. Word Search

**Greedy**:
1. Activity Selection
2. Interval Problems
3. Fractional Knapsack
4. Jump Game
5. Gas Station

### When to Use

**Backtracking**: All solutions, constraint satisfaction, generate all possibilities

**Greedy**: Optimal solution, greedy choice property, no reconsideration needed

**DP**: Optimal solution, overlapping subproblems, need to store results

### Key Takeaways

1. **Backtracking**: Try all, backtrack when invalid
2. **Greedy**: Make locally optimal choice
3. **DP**: Store results of subproblems
4. **Choose wisely**: Understand problem requirements

### Next Steps

**Part 5** will cover:
- Complete Problem-Solving Framework
- Pattern Combinations
- Master Strategy for Any Problem

---

**Master backtracking and greedy to solve constraint and optimization problems!**

