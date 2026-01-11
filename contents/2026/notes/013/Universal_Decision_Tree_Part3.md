# Universal Decision Tree: Solve Any DS/Algo Problem

## Part 3: DP, Backtracking, Greedy, and Complete Framework

---

## Table of Contents

1. [Dynamic Programming Decision Tree](#1-dynamic-programming-decision-tree)
2. [Backtracking Decision Tree](#2-backtracking-decision-tree)
3. [Greedy Decision Tree](#3-greedy-decision-tree)
4. [Complete Universal Decision Framework](#4-complete-universal-decision-framework)
5. [Problem-Solving Workflow](#5-problem-solving-workflow)
6. [Master Decision Algorithm](#6-master-decision-algorithm)

---

## 1. Dynamic Programming Decision Tree

### 1.1 Complete DP Decision Tree

```python
"""
DYNAMIC PROGRAMMING DECISION TREE

Question 1: Is it a DP problem?
│
├─ Check DP Indicators:
│   ├─ Optimal substructure? → YES
│   ├─ Overlapping subproblems? → YES
│   ├─ Maximization/Minimization? → YES
│   └─ Counting problems? → YES
│
├─ YES → Continue to Question 2
└─ NO → Not DP, check other patterns

Question 2: What is the problem structure?
│
├─ 1D Array → Go to 1D DP Decision Tree (1.2)
│
├─ 2D Grid → Go to 2D Grid DP Decision Tree (1.3)
│
├─ Two Strings → Go to String DP Decision Tree (1.4)
│
├─ Tree → Go to Tree DP Decision Tree (1.5)
│
└─ Other → Analyze specific structure
"""
```

### 1.2 1D DP Decision Tree

```python
"""
1D DP DECISION TREE

Question 1: What is the problem type?
│
├─ Linear progression → Linear DP
│   │
│   ├─ Examples: Climbing Stairs, Fibonacci
│   └─ Pattern: dp[i] = f(dp[i-1], dp[i-2], ...)
│
├─ Choose or skip → House Robber Pattern
│   │
│   ├─ Examples: House Robber, House Robber II
│   └─ Pattern: dp[i] = max(dp[i-1], dp[i-2] + value[i])
│
├─ Coin/Item selection → Coin Change Pattern
│   │
│   ├─ Examples: Coin Change, Coin Change II
│   └─ Pattern: dp[i] = min/max(dp[i], dp[i-coin] + 1)
│
├─ Increasing sequence → LIS Pattern
│   │
│   ├─ Examples: Longest Increasing Subsequence
│   └─ Pattern: dp[i] = max(dp[j] + 1) for j < i and arr[j] < arr[i]
│
└─ Word/Substring → Word Break Pattern
    │
    ├─ Examples: Word Break, Decode Ways
    └─ Pattern: dp[i] = true if substring[0:i] can be formed
"""
```

**Detailed Examples**:

**Example 1: Linear DP Identification**
```python
def identify_linear_dp(problem):
    """
    Identify: "ways to", "steps", "paths", linear progression
    Pattern: Linear DP
    """
    keywords = ['ways to', 'steps', 'paths', 'climb', 'reach']
    if any(kw in problem.lower() for kw in keywords):
        return "Linear DP"
    return None

# Template
def linear_dp_template(n):
    dp = [0] * (n + 1)
    dp[0] = base_value
    dp[1] = base_value
    
    for i in range(2, n + 1):
        dp[i] = dp[i - 1] + dp[i - 2]  # or other transition
    
    return dp[n]
```

**Example 2: House Robber Pattern**
```python
def identify_house_robber_pattern(problem):
    """
    Identify: "robber", "choose or skip", "maximum value"
    Pattern: Choose or Skip DP
    """
    keywords = ['robber', 'choose', 'skip', 'maximum value', 'select']
    if any(kw in problem.lower() for kw in keywords):
        return "House Robber Pattern (Choose or Skip)"
    return None

# Template
def house_robber_template(nums):
    n = len(nums)
    if n == 0:
        return 0
    if n == 1:
        return nums[0]
    
    dp = [0] * n
    dp[0] = nums[0]
    dp[1] = max(nums[0], nums[1])
    
    for i in range(2, n):
        dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])
    
    return dp[n - 1]
```

**Example 3: Coin Change Pattern**
```python
def identify_coin_change_pattern(problem):
    """
    Identify: "coin", "change", "make amount", "minimum/maximum coins"
    Pattern: Coin Change DP
    """
    keywords = ['coin', 'change', 'make amount', 'coins']
    if any(kw in problem.lower() for kw in keywords):
        if 'ways' in problem.lower() or 'how many' in problem.lower():
            return "Coin Change (Count Ways)"
        else:
            return "Coin Change (Minimum/Maximum)"
    return None

# Template: Minimum coins
def coin_change_min_template(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] = min(dp[i], dp[i - coin] + 1)
    
    return dp[amount] if dp[amount] != float('inf') else -1

# Template: Count ways
def coin_change_ways_template(coins, amount):
    dp = [0] * (amount + 1)
    dp[0] = 1
    
    for coin in coins:
        for i in range(coin, amount + 1):
            dp[i] += dp[i - coin]
    
    return dp[amount]
```

### 1.3 2D Grid DP Decision Tree

```python
"""
2D GRID DP DECISION TREE

Question 1: What is the grid problem type?
│
├─ Path counting → Unique Paths Pattern
│   │
│   ├─ Examples: Unique Paths, Unique Paths II
│   └─ Pattern: dp[i][j] = dp[i-1][j] + dp[i][j-1]
│
├─ Path optimization → Minimum Path Sum Pattern
│   │
│   ├─ Examples: Minimum Path Sum
│   └─ Pattern: dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
│
└─ Other grid problems → Analyze specific requirement
"""
```

**Example: Grid DP Identification**
```python
def identify_grid_dp(problem):
    """
    Identify grid DP problems
    """
    if 'unique paths' in problem.lower():
        return "Grid DP (Path Counting)"
    elif 'minimum path sum' in problem.lower() or 'maximum path sum' in problem.lower():
        return "Grid DP (Path Optimization)"
    elif 'grid' in problem.lower() and ('path' in problem.lower() or 'route' in problem.lower()):
        return "Grid DP (Analyze further)"
    return None
```

### 1.4 String DP Decision Tree

```python
"""
STRING DP DECISION TREE

Question 1: What is the string problem type?
│
├─ Two strings comparison → LCS/Edit Distance Pattern
│   │
│   ├─ Longest Common Subsequence → LCS Pattern
│   ├─ Edit Distance → Edit Distance Pattern
│   └─ Interleaving → Interleaving Pattern
│
├─ Single string → Palindrome/Word Break Pattern
│   │
│   ├─ Palindrome → Palindrome DP
│   └─ Word Break → Word Break DP
│
└─ Pattern matching → Regex Matching Pattern
    │
    └─ Examples: Regular Expression Matching
"""
```

**Example: String DP Identification**
```python
def identify_string_dp(problem):
    """
    Identify string DP problems
    """
    if 'longest common' in problem.lower() or 'lcs' in problem.lower():
        return "String DP (LCS)"
    elif 'edit distance' in problem.lower():
        return "String DP (Edit Distance)"
    elif 'palindrome' in problem.lower():
        return "String DP (Palindrome)"
    elif 'word break' in problem.lower():
        return "String DP (Word Break)"
    elif 'regular expression' in problem.lower() or 'regex' in problem.lower():
        return "String DP (Regex Matching)"
    return None
```

### 1.5 Tree DP Decision Tree

```python
"""
TREE DP DECISION TREE

Question 1: What tree optimization is needed?
│
├─ Maximum/Minimum path → Tree DP with return values
│   │
│   ├─ Examples: Maximum Path Sum, House Robber III
│   └─ Pattern: Return value from children, process, return to parent
│
└─ Other tree optimizations → Analyze specific requirement
"""
```

**Example: Tree DP Identification**
```python
def identify_tree_dp(problem):
    """
    Identify tree DP problems
    """
    if 'tree' in problem.lower() and ('maximum' in problem.lower() or 'minimum' in problem.lower()):
        if 'path' in problem.lower():
            return "Tree DP (Maximum/Minimum Path)"
        else:
            return "Tree DP (Optimization)"
    return None

# Template: Tree DP
def tree_dp_template(root):
    def dfs(node):
        if not node:
            return base_value
        
        left_result = dfs(node.left)
        right_result = dfs(node.right)
        
        # Process with children's results
        current_result = process(node, left_result, right_result)
        
        # Update global if needed
        update_global(current_result)
        
        return current_result
    
    return dfs(root)
```

---

## 2. Backtracking Decision Tree

### 2.1 Complete Backtracking Decision Tree

```python
"""
BACKTRACKING DECISION TREE

Question 1: Is it a backtracking problem?
│
├─ Check Backtracking Indicators:
│   ├─ Need all solutions? → YES
│   ├─ Generate all possibilities? → YES
│   ├─ Constraint satisfaction? → YES
│   └─ "Find all" problems? → YES
│
├─ YES → Continue to Question 2
└─ NO → Not Backtracking, check DP or Greedy

Question 2: What type of backtracking?
│
├─ Permutations → Permutation Pattern
│   │
│   └─ Examples: Permutations, Permutations II
│
├─ Combinations → Combination Pattern
│   │
│   └─ Examples: Combinations, Subsets
│
├─ Constraint Satisfaction → Constraint Pattern
│   │
│   └─ Examples: N-Queens, Sudoku Solver
│
├─ Combination Sum → Combination Sum Pattern
│   │
│   └─ Examples: Combination Sum, Combination Sum II
│
└─ Path Finding → Path Backtracking Pattern
    │
    └─ Examples: Word Search, Path Sum II
"""
```

**Detailed Examples**:

**Example 1: Permutation Identification**
```python
def identify_permutation(problem):
    """
    Identify: "permutation", "all arrangements", "all orders"
    Pattern: Permutation Backtracking
    """
    keywords = ['permutation', 'all arrangements', 'all orders', 'arrange']
    if any(kw in problem.lower() for kw in keywords):
        return "Backtracking (Permutations)"
    return None

# Template
def permutation_template(nums):
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

**Example 2: Combination Identification**
```python
def identify_combination(problem):
    """
    Identify: "combination", "subset", "choose k from n"
    Pattern: Combination Backtracking
    """
    keywords = ['combination', 'subset', 'choose', 'select k']
    if any(kw in problem.lower() for kw in keywords):
        return "Backtracking (Combinations)"
    return None

# Template
def combination_template(n, k):
    result = []
    
    def backtrack(current, start):
        if len(current) == k:
            result.append(current[:])
            return
        
        for i in range(start, n + 1):
            current.append(i)
            backtrack(current, i + 1)
            current.pop()
    
    backtrack([], 1)
    return result
```

**Example 3: Constraint Satisfaction**
```python
def identify_constraint_satisfaction(problem):
    """
    Identify: "n-queens", "sudoku", "constraint", "satisfy"
    Pattern: Constraint Satisfaction Backtracking
    """
    keywords = ['n-queens', 'sudoku', 'constraint', 'satisfy', 'valid']
    if any(kw in problem.lower() for kw in keywords):
        return "Backtracking (Constraint Satisfaction)"
    return None

# Template
def constraint_satisfaction_template():
    result = []
    
    def backtrack(state):
        if violates_constraints(state):
            return
        
        if is_complete(state):
            result.append(state[:])
            return
        
        for choice in get_choices(state):
            if is_valid(choice, state):
                apply_choice(state, choice)
                backtrack(state)
                undo_choice(state, choice)
    
    backtrack(initial_state)
    return result
```

---

## 3. Greedy Decision Tree

### 3.1 Complete Greedy Decision Tree

```python
"""
GREEDY DECISION TREE

Question 1: Is it a greedy problem?
│
├─ Check Greedy Indicators:
│   ├─ Optimal substructure? → YES
│   ├─ Greedy choice property? → YES
│   ├─ No need to reconsider? → YES
│   └─ Optimization problem? → YES
│
├─ YES → Continue to Question 2
└─ NO → Check DP (if overlapping subproblems)

Question 2: What type of greedy problem?
│
├─ Activity Selection → Activity Selection Pattern
│   │
│   └─ Examples: Activity Selection, Non-overlapping Intervals
│
├─ Interval Problems → Interval Greedy Pattern
│   │
│   └─ Examples: Merge Intervals, Erase Overlap Intervals
│
├─ Scheduling → Scheduling Greedy Pattern
│   │
│   └─ Examples: Task Scheduler, Meeting Rooms
│
├─ Jump Game → Jump Game Pattern
│   │
│   └─ Examples: Jump Game, Jump Game II
│
├─ Fractional Knapsack → Knapsack Greedy Pattern
│   │
│   └─ Examples: Fractional Knapsack
│
└─ Other → Analyze specific requirement
"""
```

**Detailed Examples**:

**Example 1: Activity Selection Identification**
```python
def identify_activity_selection(problem):
    """
    Identify: "activity", "schedule", "non-overlapping", "maximum"
    Pattern: Activity Selection Greedy
    """
    keywords = ['activity', 'schedule', 'non-overlapping', 'maximum activities']
    if any(kw in problem.lower() for kw in keywords):
        return "Greedy (Activity Selection)"
    return None

# Template
def activity_selection_template(start, finish):
    activities = sorted(zip(start, finish), key=lambda x: x[1])
    selected = [activities[0]]
    last_finish = activities[0][1]
    
    for s, f in activities[1:]:
        if s >= last_finish:
            selected.append((s, f))
            last_finish = f
    
    return selected
```

**Example 2: Interval Problems**
```python
def identify_interval_greedy(problem):
    """
    Identify: "interval", "merge", "overlap", "erase"
    Pattern: Interval Greedy
    """
    keywords = ['interval', 'merge', 'overlap', 'erase']
    if any(kw in problem.lower() for kw in keywords):
        return "Greedy (Intervals)"
    return None
```

**Example 3: Jump Game**
```python
def identify_jump_game(problem):
    """
    Identify: "jump", "reach", "can jump", "minimum jumps"
    Pattern: Jump Game Greedy
    """
    keywords = ['jump', 'reach', 'can jump', 'minimum jumps']
    if any(kw in problem.lower() for kw in keywords):
        return "Greedy (Jump Game)"
    return None

# Template: Can Jump
def jump_game_template(nums):
    max_reach = 0
    
    for i, num in enumerate(nums):
        if i > max_reach:
            return False
        max_reach = max(max_reach, i + num)
        if max_reach >= len(nums) - 1:
            return True
    
    return True
```

---

## 4. Complete Universal Decision Framework

### 4.1 Master Decision Algorithm

```python
def master_decision_algorithm(problem_description, constraints, input_type):
    """
    Complete universal decision algorithm
    """
    # Step 1: Classify problem category
    category = classify_problem_category(problem_description)
    
    # Step 2: Identify specific pattern based on category
    if category == "Array/String":
        pattern = array_string_decision_tree(problem_description, constraints)
    elif category == "Linked List":
        pattern = linked_list_decision_tree(problem_description)
    elif category == "Tree":
        pattern = tree_decision_tree(problem_description)
    elif category == "Graph":
        pattern = graph_decision_tree(problem_description)
    elif category == "DP":
        pattern = dp_decision_tree(problem_description, constraints)
    elif category == "Backtracking":
        pattern = backtracking_decision_tree(problem_description)
    elif category == "Greedy":
        pattern = greedy_decision_tree(problem_description)
    else:
        pattern = "Unknown - Analyze further"
    
    # Step 3: Verify pattern fits constraints
    if verify_pattern(pattern, constraints):
        return pattern
    else:
        # Try alternative patterns
        return find_alternative_pattern(problem_description, constraints)
```

### 4.2 Complete Decision Flowchart

```python
"""
COMPLETE UNIVERSAL DECISION FLOWCHART

START
  │
  ├─ Read Problem Statement
  │
  ├─ Extract Keywords and Constraints
  │
  ├─ Classify Problem Category
  │   │
  │   ├─ Array/String? → Array/String Decision Tree
  │   ├─ Linked List? → Linked List Decision Tree
  │   ├─ Tree? → Tree Decision Tree
  │   ├─ Graph? → Graph Decision Tree
  │   ├─ DP? → DP Decision Tree
  │   ├─ Backtracking? → Backtracking Decision Tree
  │   └─ Greedy? → Greedy Decision Tree
  │
  ├─ Identify Specific Pattern
  │
  ├─ Verify Pattern Fits Constraints
  │   │
  │   ├─ YES → Use Pattern
  │   └─ NO → Find Alternative
  │
  ├─ Apply Template
  │
  ├─ Test Solution
  │
  └─ END
"""
```

### 4.3 Problem Category Classifier

```python
def classify_problem_category(problem_description):
    """
    Classify problem into main category
    """
    desc_lower = problem_description.lower()
    
    # Priority order matters (more specific first)
    
    # Backtracking (check first - very specific)
    if any(kw in desc_lower for kw in ['all', 'generate all', 'permutation', 'combination']):
        if 'optimal' not in desc_lower:
            return "Backtracking"
    
    # DP (check before Greedy - both are optimization)
    if any(kw in desc_lower for kw in ['maximum', 'minimum', 'optimal', 'how many ways']):
        if 'greedy' not in desc_lower:
            return "DP"
    
    # Greedy
    if any(kw in desc_lower for kw in ['greedy', 'schedule', 'activity', 'interval']):
        return "Greedy"
    
    # Tree
    if any(kw in desc_lower for kw in ['tree', 'binary tree', 'bst', 'node']):
        if 'graph' not in desc_lower:
            return "Tree"
    
    # Graph
    if any(kw in desc_lower for kw in ['graph', 'node', 'edge', 'adjacency', 'shortest path']):
        return "Graph"
    
    # Linked List
    if any(kw in desc_lower for kw in ['linked list', 'listnode', 'node.next', 'cycle']):
        return "Linked List"
    
    # Array/String (default for most problems)
    if any(kw in desc_lower for kw in ['array', 'string', 'subarray', 'substring']):
        return "Array/String"
    
    return "Unknown"
```

### 4.4 Pattern Verification

```python
def verify_pattern(pattern, constraints):
    """
    Verify if pattern fits time/space constraints
    """
    pattern_complexity = get_pattern_complexity(pattern)
    required_complexity = analyze_constraints(constraints)
    
    return pattern_complexity <= required_complexity

def get_pattern_complexity(pattern):
    """
    Get time/space complexity of pattern
    """
    complexities = {
        'Two Pointers': ('O(n)', 'O(1)'),
        'Sliding Window': ('O(n)', 'O(k)'),
        'Hash Table': ('O(n)', 'O(n)'),
        'Binary Search': ('O(log n)', 'O(1)'),
        'DFS': ('O(V+E)', 'O(V)'),
        'BFS': ('O(V+E)', 'O(V)'),
        'DP 1D': ('O(n)', 'O(n)'),
        'DP 2D': ('O(m*n)', 'O(m*n)'),
        'Backtracking': ('O(b^d)', 'O(d)'),
        'Greedy': ('O(n log n)', 'O(1)')
    }
    
    return complexities.get(pattern, ('Unknown', 'Unknown'))
```

---

## 5. Problem-Solving Workflow

### 5.1 Complete Workflow

```python
"""
COMPLETE PROBLEM-SOLVING WORKFLOW

Step 1: Understand Problem (5 minutes)
  ├─ Read problem statement carefully
  ├─ Identify input/output format
  ├─ Note constraints (n ≤ ?)
  ├─ List edge cases
  └─ Understand what problem asks

Step 2: Use Decision Tree (2-3 minutes)
  ├─ Classify problem category
  ├─ Follow decision tree
  ├─ Identify pattern
  └─ Verify pattern fits constraints

Step 3: Design Algorithm (5-10 minutes)
  ├─ Choose data structures
  ├─ Design approach using template
  ├─ Analyze time complexity
  ├─ Analyze space complexity
  └─ Verify it fits constraints

Step 4: Implement (15-20 minutes)
  ├─ Use appropriate template
  ├─ Write clean code
  ├─ Handle edge cases
  └─ Add comments for complex logic

Step 5: Test and Debug (10 minutes)
  ├─ Test with examples
  ├─ Test edge cases
  ├─ Check for bugs
  └─ Optimize if needed

Total Time: ~35-50 minutes per problem
"""
```

### 5.2 Decision Tree Usage Guide

```python
def use_decision_tree(problem):
    """
    Step-by-step guide to using decision tree
    """
    # Step 1: Read problem and extract keywords
    keywords = extract_keywords(problem)
    constraints = extract_constraints(problem)
    
    # Step 2: Start with main category
    category = classify_problem_category(problem)
    
    # Step 3: Follow category-specific decision tree
    if category == "Array/String":
        # Ask: Subarray? → Sorted? → Pairs? → etc.
        pattern = follow_array_decision_tree(keywords, constraints)
    # ... similar for other categories
    
    # Step 4: Verify and return
    return pattern
```

---

## 6. Master Decision Algorithm

### 6.1 Complete Implementation

```python
def solve_any_problem(problem_description, constraints, input_data):
    """
    Master algorithm to solve any DS/Algo problem
    """
    # Step 1: Classify
    category = classify_problem_category(problem_description)
    
    # Step 2: Identify Pattern
    pattern = identify_pattern(category, problem_description, constraints)
    
    # Step 3: Get Template
    template = get_template(pattern)
    
    # Step 4: Adapt Template
    solution = adapt_template(template, input_data, constraints)
    
    # Step 5: Verify
    if verify_solution(solution, constraints):
        return solution
    else:
        # Try alternative
        return try_alternative_pattern(category, problem_description)
```

### 6.2 Pattern Identification Matrix

```python
"""
PATTERN IDENTIFICATION MATRIX

Problem Characteristic → Pattern

Subarray/Substring → Sliding Window
Sorted Array + Pairs → Two Pointers
Unsorted + Pairs → Hash Table
Cycle Detection → Fast/Slow Pointers
Matching Pairs → Stack
Next Greater/Smaller → Monotonic Stack
Range Queries → Prefix Sum / Segment Tree
Frequency Count → Hash Table (Counter)
Group Elements → Hash Table (Dict)
Tree Traversal → DFS/BFS
Tree Construction → Divide & Conquer
Tree Properties → Tree DP
Graph Shortest Path → BFS/Dijkstra
Graph Connectivity → DFS/BFS/Union-Find
Topological Order → Topological Sort
Optimal + Overlapping → DP
All Solutions → Backtracking
Optimal + Greedy Choice → Greedy
"""
```

### 6.3 Quick Reference Decision Table

| Problem Type | Key Indicator | Pattern | Template |
|-------------|---------------|---------|----------|
| Subarray | "subarray", "window" | Sliding Window | Fixed/Variable window |
| Sorted + Pairs | "sorted", "two sum" | Two Pointers | Opposite ends |
| Cycle | "cycle", "loop" | Fast/Slow | Floyd's algorithm |
| Parentheses | "parentheses", "bracket" | Stack | Matching pairs |
| Tree Traversal | "traverse", "level order" | DFS/BFS | Recursive/Iterative |
| Shortest Path | "shortest", "path" | BFS/Dijkstra | Queue/Heap |
| Maximum/Minimum | "maximum", "optimal" | DP | 1D/2D DP |
| All Solutions | "all", "generate" | Backtracking | Template backtrack |
| Schedule | "schedule", "activity" | Greedy | Sort + Select |

---

## Summary: Complete Universal Decision Tree

### Three-Level Decision System

**Level 1: Category Classification**
- Array/String, Linked List, Tree, Graph, DP, Backtracking, Greedy

**Level 2: Pattern Identification**
- Category-specific decision trees
- Keyword matching
- Constraint analysis

**Level 3: Template Application**
- Pattern-specific templates
- Adaptation to problem
- Verification

### Key Principles

1. **Start Broad**: Classify category first
2. **Narrow Down**: Use decision trees
3. **Verify**: Check constraints and complexity
4. **Apply**: Use templates
5. **Test**: Verify solution

### Complete Coverage

This 3-part guide covers:
- **Part 1**: Arrays, Strings, Linked Lists, Stacks, Queues, Hash Tables
- **Part 2**: Trees, Graphs, Advanced Arrays
- **Part 3**: DP, Backtracking, Greedy, Complete Framework

### Master This System

With this complete decision tree system, you can:
- **Identify** the right pattern for any problem in minutes
- **Apply** the appropriate template immediately
- **Solve** almost any DS/Algo problem systematically
- **Excel** in competitive programming

---

## Final Decision Tree Quick Reference

```python
"""
QUICK DECISION REFERENCE

1. Subarray/Substring? → Sliding Window
2. Sorted Array? → Two Pointers or Binary Search
3. Linked List? → Fast/Slow Pointers
4. Tree? → DFS/BFS
5. Graph? → DFS/BFS/Dijkstra
6. Maximum/Minimum? → DP
7. All Solutions? → Backtracking
8. Schedule/Activity? → Greedy
9. Matching Pairs? → Stack
10. Fast Lookup? → Hash Table
"""
```

---

**You now have a complete Universal Decision Tree to solve ANY DS/Algo problem!**

**Remember**:
- **Follow the tree systematically**
- **Verify pattern fits constraints**
- **Use templates as starting points**
- **Practice to build intuition**

**Master this system and you'll solve problems faster and more accurately!** 🚀

