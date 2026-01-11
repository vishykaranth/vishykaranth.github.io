# Core DS/Algo Patterns: Solve Almost Any Problem

## Part 5: Master Problem-Solving Framework

---

## Table of Contents

1. [Complete Problem-Solving Framework](#1-complete-problem-solving-framework)
2. [Pattern Combination Strategies](#2-pattern-combination-strategies)
3. [Universal Decision Tree](#3-universal-decision-tree)
4. [Problem Classification System](#4-problem-classification-system)
5. [Master Templates](#5-master-templates)
6. [Practice Strategy](#6-practice-strategy)

---

## 1. Complete Problem-Solving Framework

### 1.1 The 5-Step Process

**Step 1: Understand the Problem**
```python
def understand_problem(problem_statement):
    """
    Questions to answer:
    1. What is the input format?
    2. What is the output format?
    3. What are the constraints? (n ≤ 10^5, etc.)
    4. What are edge cases? (empty, single element, etc.)
    5. What does the problem actually ask?
    """
    # Read problem carefully
    # Identify key information
    # Note constraints
    # List edge cases
    pass
```

**Step 2: Identify Pattern**
```python
def identify_pattern(problem_description, constraints):
    """
    Pattern identification:
    1. Look for keywords
    2. Analyze problem type
    3. Consider data structure
    4. Check complexity requirements
    """
    # Use decision tree (see section 3)
    pattern = analyze_problem(problem_description)
    return pattern
```

**Step 3: Design Algorithm**
```python
def design_algorithm(pattern, constraints):
    """
    Algorithm design:
    1. Choose data structures
    2. Design approach
    3. Analyze time complexity
    4. Analyze space complexity
    5. Verify it fits constraints
    """
    # Select appropriate template
    # Adapt to problem
    # Verify complexity
    pass
```

**Step 4: Implement**
```python
def implement_solution(algorithm_design):
    """
    Implementation:
    1. Use template
    2. Write clean code
    3. Handle edge cases
    4. Add comments for complex logic
    """
    # Code implementation
    pass
```

**Step 5: Test and Optimize**
```python
def test_and_optimize(solution):
    """
    Testing:
    1. Test with examples
    2. Test edge cases
    3. Check for bugs
    4. Optimize if needed
    """
    # Test cases
    # Debug
    # Optimize
    pass
```

### 1.2 Complete Framework Template

```python
def solve_any_problem(problem_input):
    """
    Universal problem-solving framework
    """
    # STEP 1: Understand
    # - Input: ?
    # - Output: ?
    # - Constraints: n ≤ ?
    # - Edge cases: empty, single, etc.
    
    # STEP 2: Identify Pattern
    # - Keywords: subarray, sorted, tree, graph, etc.
    # - Pattern: Two Pointers, Sliding Window, DP, etc.
    
    # STEP 3: Design
    # - Data structures: array, hash table, stack, etc.
    # - Approach: template adaptation
    # - Complexity: O(?), O(?)
    
    # STEP 4: Implement
    # - Use template
    # - Handle edge cases
    # - Write clean code
    
    # STEP 5: Test
    # - Examples
    # - Edge cases
    # - Optimize
    
    return result
```

---

## 2. Pattern Combination Strategies

### 2.1 Common Combinations

**Combination 1: Hash Table + Two Pointers**
```python
def two_pointers_with_hash(arr, target):
    """Two pointers with hash table for lookups"""
    seen = {}
    left, right = 0, len(arr) - 1
    
    while left < right:
        current_sum = arr[left] + arr[right]
        complement = target - current_sum
        
        if complement in seen:
            return [seen[complement], left, right]
        
        seen[current_sum] = left
        # Continue two pointers logic
        if current_sum < target:
            left += 1
        else:
            right -= 1
    
    return []
```

**Combination 2: Stack + Hash Table**
```python
def stack_with_hash(s):
    """Stack operations with hash table for fast lookup"""
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)
    
    return len(stack) == 0
```

**Combination 3: Sliding Window + Hash Table**
```python
def sliding_window_with_hash(s, t):
    """Sliding window with hash table for counting"""
    need = {}
    for char in t:
        need[char] = need.get(char, 0) + 1
    
    window = {}
    left = 0
    valid = 0
    
    for right in range(len(s)):
        char = s[right]
        if char in need:
            window[char] = window.get(char, 0) + 1
            if window[char] == need[char]:
                valid += 1
        
        while valid == len(need):
            # Process window
            # Shrink window
            pass
    
    return result
```

**Combination 4: DP + Two Pointers**
```python
def dp_with_two_pointers(arr):
    """DP with two pointers optimization"""
    n = len(arr)
    # Use two pointers to reduce DP state space
    # Example: Palindrome DP with two pointers
    pass
```

### 2.2 Multi-Pattern Problems

**Example: Complex Problem**
```python
def complex_problem_solver(input_data):
    """
    Problems often require multiple patterns:
    1. Use one pattern for main logic
    2. Use another for optimization
    3. Combine as needed
    """
    # Pattern 1: Main approach (e.g., DP)
    # Pattern 2: Optimization (e.g., Two Pointers)
    # Pattern 3: Data structure (e.g., Hash Table)
    
    pass
```

---

## 3. Universal Decision Tree

### 3.1 Complete Decision Tree

```python
"""
UNIVERSAL PROBLEM-SOLVING DECISION TREE

1. Is it a subarray/substring problem?
   YES → Sliding Window
   NO → Continue

2. Is the array sorted (or can be sorted)?
   YES → Two Pointers (opposite ends)
   NO → Continue

3. Is it a linked list problem?
   YES → Fast/Slow Pointers
   NO → Continue

4. Need range queries?
   YES → Prefix Sum
   NO → Continue

5. Need fast lookup?
   YES → Hash Table
   NO → Continue

6. Need matching pairs or nested structures?
   YES → Stack
   NO → Continue

7. Is it a tree problem?
   YES → 
      a. Traversal? → DFS/BFS
      b. Construction? → Divide & Conquer
      c. Properties? → Tree DP
   NO → Continue

8. Is it a graph problem?
   YES →
      a. Shortest path? → BFS/Dijkstra
      b. Connectivity? → DFS/BFS
      c. Ordering? → Topological Sort
   NO → Continue

9. Need all solutions?
   YES → Backtracking
   NO → Continue

10. Need optimal solution?
    YES →
       a. Greedy choice property? → Greedy
       b. Overlapping subproblems? → DP
       c. Neither? → Try both
    NO → Continue

11. Need to count/optimize?
    YES → DP
    NO → Continue

12. String matching?
    YES → KMP, Z-algorithm, Rabin-Karp
    NO → Unknown pattern
"""
```

### 3.2 Pattern Selection Algorithm

```python
def select_pattern(problem_description, constraints, input_type):
    """
    Automated pattern selection
    """
    keywords = problem_description.lower()
    
    # Pattern 1: Sliding Window
    if any(word in keywords for word in ['subarray', 'substring', 'window', 'consecutive']):
        return "Sliding Window"
    
    # Pattern 2: Two Pointers
    if 'sorted' in keywords or ('pair' in keywords and 'sum' in keywords):
        return "Two Pointers"
    
    # Pattern 3: Hash Table
    if any(word in keywords for word in ['frequency', 'count', 'duplicate', 'group']):
        return "Hash Table"
    
    # Pattern 4: Stack
    if any(word in keywords for word in ['parentheses', 'bracket', 'nested']):
        return "Stack"
    
    # Pattern 5: Tree
    if 'tree' in keywords or 'binary tree' in keywords:
        if 'traverse' in keywords:
            return "Tree DFS/BFS"
        elif 'construct' in keywords:
            return "Tree Construction"
        else:
            return "Tree Property"
    
    # Pattern 6: Graph
    if 'graph' in keywords or ('node' in keywords and 'edge' in keywords):
        if 'shortest' in keywords:
            return "Graph BFS/Dijkstra"
        elif 'order' in keywords:
            return "Topological Sort"
        else:
            return "Graph DFS/BFS"
    
    # Pattern 7: Backtracking
    if any(word in keywords for word in ['all', 'generate', 'permutation', 'combination']):
        return "Backtracking"
    
    # Pattern 8: DP
    if any(word in keywords for word in ['maximum', 'minimum', 'optimal', 'how many ways']):
        return "DP"
    
    # Pattern 9: Greedy
    if 'greedy' in keywords or ('optimal' in keywords and 'local' in keywords):
        return "Greedy"
    
    return "Unknown - Analyze further"
```

---

## 4. Problem Classification System

### 4.1 Problem Categories

**Category 1: Array/String Problems**
```python
array_patterns = {
    'two_pointers': ['Two Sum', '3Sum', 'Container With Water'],
    'sliding_window': ['Longest Substring', 'Minimum Window'],
    'prefix_sum': ['Subarray Sum', 'Range Sum Query'],
    'hash_table': ['Group Anagrams', 'Two Sum'],
    'stack': ['Next Greater Element', 'Valid Parentheses']
}
```

**Category 2: Tree Problems**
```python
tree_patterns = {
    'dfs': ['Tree Traversal', 'Path Sum'],
    'bfs': ['Level Order', 'Right Side View'],
    'construction': ['Build Tree', 'BST from Array'],
    'properties': ['Height', 'Diameter', 'LCA']
}
```

**Category 3: Graph Problems**
```python
graph_patterns = {
    'dfs': ['Connected Components', 'Cycle Detection'],
    'bfs': ['Shortest Path', 'Level Order'],
    'topological': ['Course Schedule', 'Alien Dictionary'],
    'mst': ['Minimum Spanning Tree']
}
```

**Category 4: DP Problems**
```python
dp_patterns = {
    '1d': ['Climbing Stairs', 'House Robber', 'Coin Change'],
    '2d': ['Unique Paths', 'Edit Distance', 'LCS'],
    'string': ['Palindrome', 'Word Break'],
    'tree': ['House Robber III', 'Max Path Sum']
}
```

**Category 5: Backtracking Problems**
```python
backtracking_patterns = {
    'permutations': ['Permutations', 'N-Queens'],
    'combinations': ['Combinations', 'Subsets'],
    'constraint': ['Sudoku', 'Word Search']
}
```

### 4.2 Classification Algorithm

```python
def classify_problem(problem):
    """
    Classify problem into category and pattern
    """
    category = None
    pattern = None
    
    # Determine category
    if 'array' in problem or 'string' in problem:
        category = 'Array/String'
    elif 'tree' in problem:
        category = 'Tree'
    elif 'graph' in problem:
        category = 'Graph'
    elif 'dp' in problem or 'dynamic' in problem:
        category = 'DP'
    elif 'backtrack' in problem or 'generate' in problem:
        category = 'Backtracking'
    
    # Determine pattern (use decision tree)
    pattern = select_pattern(problem, constraints, input_type)
    
    return (category, pattern)
```

---

## 5. Master Templates

### 5.1 Universal Template Collection

**Template 1: Array Problem Solver**
```python
def solve_array_problem(arr, target=None):
    """
    Universal array problem template
    """
    # Step 1: Sort if needed
    # arr.sort()
    
    # Step 2: Choose pattern
    # - Two Pointers
    # - Sliding Window
    # - Hash Table
    # - Prefix Sum
    
    # Step 3: Implement
    result = []
    # ... implementation
    
    return result
```

**Template 2: Tree Problem Solver**
```python
def solve_tree_problem(root):
    """
    Universal tree problem template
    """
    def dfs(node):
        if not node:
            return base_value
        
        left = dfs(node.left)
        right = dfs(node.right)
        
        result = process(node, left, right)
        update_global(result)
        
        return result
    
    return dfs(root)
```

**Template 3: Graph Problem Solver**
```python
def solve_graph_problem(graph, start):
    """
    Universal graph problem template
    """
    visited = set()
    result = []
    
    def dfs(node):
        visited.add(node)
        process(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs(neighbor)
    
    # Or BFS
    def bfs(start):
        queue = deque([start])
        visited.add(start)
        
        while queue:
            node = queue.popleft()
            process(node)
            
            for neighbor in graph.get(node, []):
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)
    
    return result
```

**Template 4: DP Problem Solver**
```python
def solve_dp_problem(input_data):
    """
    Universal DP problem template
    """
    # Step 1: Identify state
    # dp[i] = ?
    
    # Step 2: Define DP array
    n = len(input_data)
    dp = [0] * (n + 1)
    
    # Step 3: Base cases
    dp[0] = base_value
    
    # Step 4: State transition
    for i in range(1, n + 1):
        dp[i] = transition(dp, i, input_data)
    
    # Step 5: Return answer
    return dp[n]
```

**Template 5: Backtracking Problem Solver**
```python
def solve_backtracking_problem(choices):
    """
    Universal backtracking template
    """
    result = []
    
    def backtrack(current_state, remaining_choices):
        if is_solution(current_state):
            result.append(current_state[:])
            return
        
        for choice in remaining_choices:
            if is_valid(choice, current_state):
                current_state.append(choice)
                backtrack(current_state, get_new_choices(choice))
                current_state.pop()
    
    backtrack([], choices)
    return result
```

### 5.2 Quick Reference Card

```python
"""
QUICK REFERENCE: Pattern → Template

Two Pointers → left, right = 0, len(arr)-1; while left < right
Sliding Window → left = 0; for right in range(len(s)): while condition
Hash Table → seen = {}; for item in arr: if item in seen
Stack → stack = []; for item in arr: while stack and condition
Prefix Sum → prefix = [0]; for i: prefix[i+1] = prefix[i] + arr[i]
Tree DFS → def dfs(node): if not node: return; left = dfs(node.left)
Graph BFS → queue = deque([start]); while queue: node = queue.popleft()
DP → dp = [0] * n; for i: dp[i] = transition(dp, i)
Backtracking → def backtrack(state): if solution: result.append(state[:])
Greedy → arr.sort(); for item in arr: if valid: result.append(item)
"""
```

---

## 6. Practice Strategy

### 6.1 Systematic Practice Plan

**Phase 1: Master Core Patterns (Weeks 1-4)**
```python
week1_patterns = [
    'Two Pointers',
    'Sliding Window',
    'Hash Table'
]
# Practice: 20-30 problems per pattern

week2_patterns = [
    'Stack',
    'Prefix Sum',
    'Tree DFS/BFS'
]
# Practice: 20-30 problems per pattern

week3_patterns = [
    'Graph DFS/BFS',
    'DP 1D',
    'DP 2D'
]
# Practice: 20-30 problems per pattern

week4_patterns = [
    'Backtracking',
    'Greedy',
    'Advanced Patterns'
]
# Practice: 20-30 problems per pattern
```

**Phase 2: Pattern Combinations (Weeks 5-8)**
```python
# Practice problems requiring multiple patterns
# Focus on recognizing when to combine patterns
# Solve 50-100 medium problems
```

**Phase 3: Speed and Accuracy (Weeks 9-12)**
```python
# Timed practice sessions
# Contest simulation
# Focus on speed and accuracy
# Solve 100-150 problems
```

### 6.2 Problem Selection Strategy

```python
def select_practice_problems(current_level, target_pattern):
    """
    Select problems based on:
    1. Current skill level
    2. Target pattern to learn
    3. Problem difficulty
    4. Time available
    """
    if current_level == 'beginner':
        return easy_problems[target_pattern]
    elif current_level == 'intermediate':
        return medium_problems[target_pattern]
    else:
        return hard_problems[target_pattern]
```

### 6.3 Learning Workflow

```python
"""
Daily Practice Workflow:

1. Read Problem (5 min)
   - Understand requirements
   - Identify pattern
   - Note constraints

2. Design Solution (10 min)
   - Choose pattern
   - Design algorithm
   - Analyze complexity

3. Implement (15-20 min)
   - Use template
   - Write code
   - Handle edge cases

4. Test & Debug (10 min)
   - Test examples
   - Fix bugs
   - Verify correctness

5. Review Solution (10 min)
   - Compare with optimal
   - Learn new techniques
   - Note improvements

Total: ~50-60 minutes per problem
"""
```

---

## 7. Master Checklist

### 7.1 Problem-Solving Checklist

```python
"""
PROBLEM-SOLVING CHECKLIST:

Before Coding:
□ Understand problem completely
□ Identify input/output format
□ Note constraints
□ List edge cases
□ Identify pattern
□ Design algorithm
□ Analyze complexity
□ Verify it fits constraints

During Coding:
□ Use appropriate template
□ Handle edge cases
□ Write clean code
□ Add comments for complex logic
□ Test as you code

After Coding:
□ Test with examples
□ Test edge cases
□ Check for bugs
□ Optimize if needed
□ Review solution
"""
```

### 7.2 Pattern Mastery Checklist

```python
"""
PATTERN MASTERY CHECKLIST:

Core Patterns:
□ Two Pointers (20+ problems)
□ Sliding Window (20+ problems)
□ Hash Table (20+ problems)
□ Stack (15+ problems)
□ Prefix Sum (15+ problems)

Tree/Graph:
□ Tree DFS/BFS (30+ problems)
□ Graph DFS/BFS (25+ problems)
□ Tree Construction (15+ problems)
□ Graph Algorithms (20+ problems)

Advanced:
□ DP 1D (30+ problems)
□ DP 2D (25+ problems)
□ Backtracking (25+ problems)
□ Greedy (20+ problems)

Total: 250+ problems for solid foundation
"""
```

---

## Summary: Master Framework

### Complete System

1. **5-Step Process**: Understand → Identify → Design → Implement → Test
2. **Decision Tree**: Systematic pattern selection
3. **Templates**: Ready-to-use code templates
4. **Combinations**: Multi-pattern strategies
5. **Practice Plan**: Systematic learning path

### Key Principles

1. **Pattern Recognition**: Learn to identify patterns quickly
2. **Template Usage**: Adapt templates to problems
3. **Systematic Approach**: Follow framework consistently
4. **Practice Regularly**: Solve problems daily
5. **Review Solutions**: Learn from optimal solutions

### Final Tips

1. **Start with Patterns**: Master core patterns first
2. **Use Templates**: Don't reinvent the wheel
3. **Practice Systematically**: Follow practice plan
4. **Review Regularly**: Learn from mistakes
5. **Stay Consistent**: Daily practice > occasional marathon

---

## Complete Pattern Reference

### All Patterns Covered

**Part 1**: Two Pointers, Sliding Window, Fast/Slow, Prefix Sum, Hash Table, Stack
**Part 2**: Tree DFS/BFS, Tree Construction, Tree Properties, Graph Traversals
**Part 3**: DP 1D, DP 2D, DP on Strings, State Transitions
**Part 4**: Backtracking, Greedy, Pattern Selection
**Part 5**: Complete Framework, Decision Tree, Master Templates

### Master This System

With this complete framework, you can:
- **Identify** the right pattern for any problem
- **Apply** the appropriate template
- **Combine** patterns when needed
- **Solve** almost any DS/Algo problem
- **Excel** in competitive programming

---

**You now have a complete system to solve almost any DS/Algo problem!**

**Remember**: 
- **Patterns are your tools** - Master them
- **Templates are your starting point** - Adapt them
- **Practice is your path** - Do it daily
- **Framework is your guide** - Follow it

**Good luck with your competitive programming journey!** 🚀

