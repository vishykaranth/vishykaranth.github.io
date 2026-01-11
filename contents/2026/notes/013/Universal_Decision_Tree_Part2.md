# Universal Decision Tree: Solve Any DS/Algo Problem

## Part 2: Trees, Graphs, and Advanced Patterns

---

## Table of Contents

1. [Tree Problem Decision Tree](#1-tree-problem-decision-tree)
2. [Graph Problem Decision Tree](#2-graph-problem-decision-tree)
3. [Advanced Array Patterns](#3-advanced-array-patterns)
4. [Multi-Pattern Problems](#4-multi-pattern-problems)
5. [Optimization Decision Tree](#5-optimization-decision-tree)
6. [Complete Flowchart Integration](#6-complete-flowchart-integration)

---

## 1. Tree Problem Decision Tree

### 1.1 Complete Tree Decision Tree

```python
"""
TREE PROBLEM DECISION TREE

Question 1: What operation is needed?
│
├─ Traversal → Go to Traversal Decision Tree (1.2)
│
├─ Construction → Go to Construction Decision Tree (1.3)
│
├─ Properties → Go to Properties Decision Tree (1.4)
│
├─ Path Problems → Go to Path Decision Tree (1.5)
│
└─ Other → Continue analysis
"""
```

### 1.2 Traversal Decision Tree

```python
"""
TRAVERSAL DECISION TREE

Question 1: What order is needed?
│
├─ Level by level → BFS (Queue)
│   │
│   ├─ Simple level order → Level Order Traversal
│   ├─ Zigzag → Level Order with direction toggle
│   ├─ Right side view → BFS, take last element of each level
│   └─ Level averages → BFS, calculate average per level
│
└─ Depth-first → Go to DFS Decision Tree

DFS Decision Tree:
│
├─ Preorder (Root → Left → Right) → DFS Preorder
│   │
│   └─ Examples: Tree Serialization, Tree Copying
│
├─ Inorder (Left → Root → Right) → DFS Inorder
│   │
│   └─ Examples: BST Validation, BST to Sorted Array
│
└─ Postorder (Left → Right → Root) → DFS Postorder
    │
    └─ Examples: Tree Deletion, Expression Evaluation
"""
```

**Detailed Examples**:

**Example 1: Level Order (BFS)**
```python
def identify_tree_bfs(problem):
    """
    Identify: "level order", "level by level", "breadth first"
    Pattern: BFS with Queue
    """
    keywords = ['level order', 'level by level', 'breadth first', 'bfs']
    if any(kw in problem.lower() for kw in keywords):
        return "BFS (Level Order)"
    return None

# Template
def level_order_template(root):
    if not root:
        return []
    
    queue = deque([root])
    result = []
    
    while queue:
        level_size = len(queue)
        level = []
        
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        
        result.append(level)
    
    return result
```

**Example 2: DFS Traversal**
```python
def identify_tree_dfs(problem):
    """
    Identify: "preorder", "inorder", "postorder", "depth first"
    Pattern: DFS (Recursive or Iterative)
    """
    if 'preorder' in problem.lower():
        return "DFS Preorder"
    elif 'inorder' in problem.lower():
        return "DFS Inorder"
    elif 'postorder' in problem.lower():
        return "DFS Postorder"
    elif 'depth first' in problem.lower() or 'dfs' in problem.lower():
        return "DFS (Determine order based on problem)"
    return None

# Universal DFS Template
def dfs_template(root, order='preorder'):
    result = []
    
    def dfs(node):
        if not node:
            return
        
        if order == 'preorder':
            result.append(node.val)
        
        dfs(node.left)
        
        if order == 'inorder':
            result.append(node.val)
        
        dfs(node.right)
        
        if order == 'postorder':
            result.append(node.val)
    
    dfs(root)
    return result
```

### 1.3 Construction Decision Tree

```python
"""
CONSTRUCTION DECISION TREE

Question 1: What information is given?
│
├─ Preorder + Inorder → Build from Preorder and Inorder
│   │
│   └─ Pattern: Divide and Conquer
│
├─ Postorder + Inorder → Build from Postorder and Inorder
│   │
│   └─ Pattern: Divide and Conquer
│
├─ Sorted Array → Build Balanced BST
│   │
│   └─ Pattern: Binary Search (middle as root)
│
├─ Level Order Array → Build from Level Order
│   │
│   └─ Pattern: BFS Construction
│
└─ Other → Analyze specific requirements
"""
```

**Example: Tree Construction Pattern**
```python
def identify_tree_construction(problem):
    """
    Identify tree construction problems
    """
    if 'preorder' in problem.lower() and 'inorder' in problem.lower():
        return "Build from Preorder and Inorder"
    elif 'postorder' in problem.lower() and 'inorder' in problem.lower():
        return "Build from Postorder and Inorder"
    elif 'sorted array' in problem.lower() and 'bst' in problem.lower():
        return "Build Balanced BST from Sorted Array"
    elif 'level order' in problem.lower() and 'construct' in problem.lower():
        return "Build from Level Order Array"
    return None

# Template: Divide and Conquer
def build_tree_template(data1, data2):
    def build(left, right):
        if left > right:
            return None
        
        # Find root (varies by problem)
        root_index = find_root(left, right, data1, data2)
        root = TreeNode(data1[root_index])
        
        # Recursively build subtrees
        root.left = build(left, root_index - 1)
        root.right = build(root_index + 1, right)
        
        return root
    
    return build(0, len(data1) - 1)
```

### 1.4 Properties Decision Tree

```python
"""
PROPERTIES DECISION TREE

Question 1: What property to check/calculate?
│
├─ Height/Depth → DFS (return height)
│   │
│   └─ Pattern: Return single value from recursion
│
├─ Balanced → DFS (return height and balanced flag)
│   │
│   └─ Pattern: Return multiple values
│
├─ Diameter → DFS (return height, track diameter)
│   │
│   └─ Pattern: Return value + update global
│
├─ Symmetric → DFS (compare left and right)
│   │
│   └─ Pattern: Two-tree comparison
│
├─ Same Tree → DFS (compare nodes)
│   │
│   └─ Pattern: Two-tree comparison
│
└─ Other properties → Analyze specific requirement
"""
```

**Example: Tree Properties Pattern**
```python
def identify_tree_property(problem):
    """
    Identify tree property problems
    """
    if 'height' in problem.lower() or 'depth' in problem.lower():
        return "Tree Height/Depth (DFS)"
    elif 'balanced' in problem.lower():
        return "Balanced Tree (DFS with Multiple Returns)"
    elif 'diameter' in problem.lower():
        return "Tree Diameter (DFS with Global Update)"
    elif 'symmetric' in problem.lower() or 'mirror' in problem.lower():
        return "Symmetric Tree (DFS Comparison)"
    elif 'same tree' in problem.lower() or 'identical' in problem.lower():
        return "Same Tree (DFS Comparison)"
    return None

# Template: Multiple Return Values
def tree_property_template(root):
    def dfs(node):
        if not node:
            return (base_value1, base_value2)
        
        left_result1, left_result2 = dfs(node.left)
        right_result1, right_result2 = dfs(node.right)
        
        # Calculate current values
        current1 = calculate1(left_result1, right_result1, node)
        current2 = calculate2(left_result2, right_result2, node)
        
        return (current1, current2)
    
    return dfs(root)
```

### 1.5 Path Decision Tree

```python
"""
PATH DECISION TREE

Question 1: What path property is needed?
│
├─ Path Sum → DFS (accumulate sum)
│   │
│   ├─ Has path sum → DFS with target sum
│   └─ All paths with sum → DFS with backtracking
│
├─ Maximum Path Sum → DFS (return gain, track max)
│   │
│   └─ Pattern: Return value + update global
│
├─ Path from root to leaf → DFS with path tracking
│   │
│   └─ Pattern: Backtracking on tree
│
└─ Lowest Common Ancestor → DFS (return node)
    │
    └─ Pattern: Return node or None
"""
```

**Example: Path Problems**
```python
def identify_tree_path(problem):
    """
    Identify tree path problems
    """
    if 'path sum' in problem.lower():
        if 'maximum' in problem.lower():
            return "Maximum Path Sum (DFS with Global)"
        else:
            return "Path Sum (DFS with Target)"
    elif 'lca' in problem.lower() or 'lowest common ancestor' in problem.lower():
        return "LCA (DFS Return Node)"
    elif 'path' in problem.lower() and 'root' in problem.lower():
        return "Root to Leaf Path (DFS with Backtracking)"
    return None
```

---

## 2. Graph Problem Decision Tree

### 2.1 Complete Graph Decision Tree

```python
"""
GRAPH PROBLEM DECISION TREE

Question 1: What operation is needed?
│
├─ Traversal → Go to Graph Traversal Decision Tree (2.2)
│
├─ Shortest Path → Go to Shortest Path Decision Tree (2.3)
│
├─ Connectivity → Go to Connectivity Decision Tree (2.4)
│
├─ Ordering → Go to Ordering Decision Tree (2.5)
│
└─ Other → Continue analysis
"""
```

### 2.2 Graph Traversal Decision Tree

```python
"""
GRAPH TRAVERSAL DECISION TREE

Question 1: What traversal is needed?
│
├─ All nodes reachable from start → DFS or BFS
│   │
│   ├─ DFS → Recursive or Iterative
│   └─ BFS → Queue-based
│
├─ Level-order traversal → BFS
│   │
│   └─ Examples: Level Order, Shortest Path (unweighted)
│
└─ Specific order → DFS with custom logic
    │
    └─ Examples: Topological Sort (DFS-based)
"""
```

**Example: Graph Traversal Identification**
```python
def identify_graph_traversal(problem):
    """
    Identify graph traversal problems
    """
    if 'shortest path' in problem.lower() and 'unweighted' in problem.lower():
        return "BFS (Shortest Path Unweighted)"
    elif 'level order' in problem.lower() or 'level by level' in problem.lower():
        return "BFS (Level Order)"
    elif 'all nodes' in problem.lower() or 'reachable' in problem.lower():
        return "DFS or BFS (Traversal)"
    elif 'topological' in problem.lower() or 'order' in problem.lower():
        return "DFS or BFS (Topological Sort)"
    return None

# Template: Graph BFS
def graph_bfs_template(graph, start):
    visited = set([start])
    queue = deque([start])
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    
    return result

# Template: Graph DFS
def graph_dfs_template(graph, start):
    visited = set()
    result = []
    
    def dfs(node):
        visited.add(node)
        result.append(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs(neighbor)
    
    dfs(start)
    return result
```

### 2.3 Shortest Path Decision Tree

```python
"""
SHORTEST PATH DECISION TREE

Question 1: Is graph weighted?
│
├─ NO (Unweighted) → BFS
│   │
│   └─ Examples: Shortest Path in Unweighted Graph
│
└─ YES (Weighted) → Continue to Question 2

Question 2: Are weights non-negative?
│
├─ YES → Dijkstra's Algorithm
│   │
│   └─ Examples: Network Delay Time, Cheapest Flights
│
└─ NO (Negative weights) → Continue to Question 3

Question 3: Are there negative cycles?
│
├─ NO → Bellman-Ford Algorithm
│   │
│   └─ Examples: Shortest Path with Negative Weights
│
└─ YES → Problem may be unsolvable (infinite negative cycles)

Question 4: Need all-pairs shortest paths?
│
├─ YES → Floyd-Warshall Algorithm
│   │
│   └─ Examples: All Pairs Shortest Path
│
└─ NO → Single-source algorithms (Dijkstra, Bellman-Ford)
"""
```

**Example: Shortest Path Identification**
```python
def identify_shortest_path(problem):
    """
    Identify shortest path problems
    """
    if 'unweighted' in problem.lower() or 'unit weight' in problem.lower():
        return "BFS (Unweighted)"
    elif 'negative weight' in problem.lower() or 'negative cycle' in problem.lower():
        return "Bellman-Ford"
    elif 'all pairs' in problem.lower() or 'all nodes' in problem.lower():
        return "Floyd-Warshall"
    elif 'weighted' in problem.lower() or 'cost' in problem.lower():
        return "Dijkstra (if non-negative weights)"
    return None
```

### 2.4 Connectivity Decision Tree

```python
"""
CONNECTIVITY DECISION TREE

Question 1: What connectivity problem?
│
├─ Connected Components → DFS or BFS
│   │
│   └─ Examples: Number of Islands, Connected Components
│
├─ Cycle Detection → DFS with color tracking
│   │
│   ├─ Directed Graph → DFS with WHITE/GRAY/BLACK
│   └─ Undirected Graph → DFS with parent tracking
│
├─ Strongly Connected Components → Kosaraju or Tarjan
│   │
│   └─ Examples: Strongly Connected Components
│
└─ Union-Find → Disjoint Set Union
    │
    └─ Examples: Number of Connected Components, Redundant Connection
"""
```

**Example: Connectivity Problems**
```python
def identify_connectivity(problem):
    """
    Identify connectivity problems
    """
    if 'connected components' in problem.lower():
        if 'strongly' in problem.lower():
            return "Kosaraju or Tarjan (SCC)"
        else:
            return "DFS/BFS (Connected Components)"
    elif 'cycle' in problem.lower():
        if 'directed' in problem.lower():
            return "DFS with Color Tracking"
        else:
            return "DFS with Parent Tracking"
    elif 'union' in problem.lower() or 'find' in problem.lower():
        return "Union-Find (DSU)"
    return None
```

### 2.5 Ordering Decision Tree

```python
"""
ORDERING DECISION TREE

Question 1: What ordering is needed?
│
├─ Topological Sort → Kahn's Algorithm or DFS
│   │
│   ├─ BFS-based → Kahn's Algorithm
│   └─ DFS-based → DFS with post-order
│
└─ Other ordering → Analyze specific requirement
"""
```

**Example: Topological Sort**
```python
def identify_topological_sort(problem):
    """
    Identify topological sort problems
    """
    keywords = ['topological', 'order', 'schedule', 'prerequisite', 'dependency']
    if any(kw in problem.lower() for kw in keywords):
        return "Topological Sort (Kahn's or DFS)"
    return None
```

---

## 3. Advanced Array Patterns

### 3.1 Advanced Array Decision Tree

```python
"""
ADVANCED ARRAY DECISION TREE

Question 1: Need range queries?
│
├─ YES → Continue to Range Query Decision Tree (3.2)
│
└─ NO → Continue to Question 2

Question 2: Need to find kth element?
│
├─ YES → QuickSelect or Heap
│   │
│   └─ Examples: Kth Largest Element, Top K Elements
│
└─ NO → Continue to Question 3

Question 3: Need to merge intervals?
│
├─ YES → Sort + Merge
│   │
│   └─ Examples: Merge Intervals, Non-overlapping Intervals
│
└─ NO → Other advanced patterns
"""
```

### 3.2 Range Query Decision Tree

```python
"""
RANGE QUERY DECISION TREE

Question 1: What type of range query?
│
├─ Sum Query → Prefix Sum
│   │
│   └─ Examples: Range Sum Query, Subarray Sum Equals K
│
├─ Min/Max Query → Segment Tree or Sparse Table
│   │
│   └─ Examples: Range Minimum Query
│
├─ Update + Query → Segment Tree or Fenwick Tree
│   │
│   └─ Examples: Range Sum Query - Mutable
│
└─ Other queries → Analyze specific requirement
"""
```

**Example: Range Query Identification**
```python
def identify_range_query(problem):
    """
    Identify range query problems
    """
    if 'range sum' in problem.lower() or 'subarray sum' in problem.lower():
        if 'mutable' in problem.lower() or 'update' in problem.lower():
            return "Segment Tree or Fenwick Tree"
        else:
            return "Prefix Sum"
    elif 'range min' in problem.lower() or 'range max' in problem.lower():
        return "Segment Tree or Sparse Table"
    return None
```

---

## 4. Multi-Pattern Problems

### 4.1 Multi-Pattern Decision Tree

```python
"""
MULTI-PATTERN DECISION TREE

Question 1: Does problem require multiple patterns?
│
├─ YES → Identify all required patterns
│   │
│   ├─ Pattern 1: Main algorithm
│   ├─ Pattern 2: Optimization/data structure
│   └─ Pattern 3: Additional support
│
└─ NO → Single pattern solution
"""
```

**Example: Multi-Pattern Problems**
```python
def identify_multi_pattern(problem):
    """
    Identify problems requiring multiple patterns
    """
    patterns = []
    
    # Check for multiple pattern indicators
    if 'sliding window' in problem.lower() and 'hash' in problem.lower():
        patterns.append("Sliding Window")
        patterns.append("Hash Table")
    
    if 'two pointers' in problem.lower() and 'sorted' in problem.lower():
        patterns.append("Two Pointers")
        if 'hash' in problem.lower():
            patterns.append("Hash Table")
    
    if 'tree' in problem.lower() and 'dp' in problem.lower():
        patterns.append("Tree DFS")
        patterns.append("Tree DP")
    
    return patterns if patterns else None
```

### 4.2 Pattern Combination Strategies

```python
"""
PATTERN COMBINATION STRATEGIES

Strategy 1: Main Pattern + Optimization
  Example: DP + Two Pointers (space optimization)
  
Strategy 2: Main Pattern + Data Structure
  Example: Sliding Window + Hash Table (counting)
  
Strategy 3: Sequential Patterns
  Example: Sort + Two Pointers
  
Strategy 4: Nested Patterns
  Example: Tree DFS + Backtracking
"""
```

---

## 5. Optimization Decision Tree

### 5.1 Optimization Decision Tree

```python
"""
OPTIMIZATION DECISION TREE

Question 1: What needs optimization?
│
├─ Time Complexity → Continue to Time Optimization (5.2)
│
├─ Space Complexity → Continue to Space Optimization (5.3)
│
└─ Both → Apply both optimizations
"""
```

### 5.2 Time Optimization Decision Tree

```python
"""
TIME OPTIMIZATION DECISION TREE

Question 1: Current complexity?
│
├─ O(n²) → Can reduce to O(n log n)?
│   │
│   ├─ YES → Sort + Two Pointers or Binary Search
│   └─ NO → Continue analysis
│
├─ O(n log n) → Can reduce to O(n)?
│   │
│   ├─ YES → Hash Table or Two Pointers
│   └─ NO → Already optimal
│
└─ O(n) → Already optimal for most cases
"""
```

### 5.3 Space Optimization Decision Tree

```python
"""
SPACE OPTIMIZATION DECISION TREE

Question 1: Current space complexity?
│
├─ O(n²) → Can reduce to O(n)?
│   │
│   ├─ YES → Use 1D DP instead of 2D
│   └─ NO → Continue analysis
│
├─ O(n) → Can reduce to O(1)?
│   │
│   ├─ YES → Use variables instead of array
│   └─ NO → Continue analysis
│
└─ O(1) → Already optimal
"""
```

---

## 6. Complete Flowchart Integration

### 6.1 Integrated Decision Flow

```python
"""
COMPLETE INTEGRATED DECISION FLOW

START
  │
  ├─ Is it a tree problem?
  │   │
  │   ├─ YES → Tree Decision Tree (Part 2, Section 1)
  │   └─ NO → Continue
  │
  ├─ Is it a graph problem?
  │   │
  │   ├─ YES → Graph Decision Tree (Part 2, Section 2)
  │   └─ NO → Continue
  │
  ├─ Is it an array/string problem?
  │   │
  │   ├─ YES → Array/String Decision Tree (Part 1)
  │   └─ NO → Continue
  │
  ├─ Is it a linked list problem?
  │   │
  │   ├─ YES → Linked List Decision Tree (Part 1, Section 4)
  │   └─ NO → Continue
  │
  └─ Go to Part 3 (DP, Backtracking, Greedy)
"""
```

### 6.2 Problem Type Classification

```python
def classify_problem_type(problem_description):
    """
    Classify problem into main category
    """
    desc_lower = problem_description.lower()
    
    # Tree problems
    if any(kw in desc_lower for kw in ['tree', 'binary tree', 'bst', 'node']):
        if 'graph' not in desc_lower:
            return "Tree"
    
    # Graph problems
    if any(kw in desc_lower for kw in ['graph', 'node', 'edge', 'adjacency']):
        return "Graph"
    
    # Array/String problems
    if any(kw in desc_lower for kw in ['array', 'string', 'subarray', 'substring']):
        return "Array/String"
    
    # Linked List problems
    if any(kw in desc_lower for kw in ['linked list', 'listnode', 'node.next']):
        return "Linked List"
    
    # DP problems
    if any(kw in desc_lower for kw in ['maximum', 'minimum', 'optimal', 'how many ways']):
        return "DP"
    
    # Backtracking problems
    if any(kw in desc_lower for kw in ['all', 'generate', 'permutation', 'combination']):
        return "Backtracking"
    
    # Greedy problems
    if any(kw in desc_lower for kw in ['greedy', 'schedule', 'activity', 'interval']):
        return "Greedy"
    
    return "Unknown"
```

---

## Summary: Part 2 Decision Tree

### Key Decision Points

1. **Tree Problems** → Traversal, Construction, Properties, Paths
2. **Graph Problems** → Traversal, Shortest Path, Connectivity, Ordering
3. **Advanced Arrays** → Range Queries, Kth Element, Intervals
4. **Multi-Pattern** → Combine patterns as needed
5. **Optimization** → Time and space optimization strategies

### Pattern Selection Rules

- **Tree traversal** → BFS for level-order, DFS for depth-first
- **Graph shortest path** → BFS (unweighted), Dijkstra (weighted)
- **Connectivity** → DFS/BFS or Union-Find
- **Range queries** → Prefix Sum (static), Segment Tree (mutable)

### Next Steps

**Part 3** will cover:
- Dynamic Programming Decision Tree
- Backtracking Decision Tree
- Greedy Decision Tree
- Complete Universal Decision Framework

---

**Master trees and graphs to solve 80% of tree/graph problems!**

