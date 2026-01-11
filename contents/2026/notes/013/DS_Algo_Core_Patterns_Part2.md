# Core DS/Algo Patterns: Solve Almost Any Problem

## Part 2: Tree and Graph Patterns

---

## Table of Contents

1. [Tree Traversal Patterns](#1-tree-traversal-patterns)
2. [Tree Construction Patterns](#2-tree-construction-patterns)
3. [Tree Property Patterns](#3-tree-property-patterns)
4. [Graph Traversal Patterns](#4-graph-traversal-patterns)
5. [Graph Algorithm Patterns](#5-graph-algorithm-patterns)
6. [Master Tree/Graph Recognition](#6-master-treegraph-recognition)

---

## 1. Tree Traversal Patterns

### 1.1 Core DFS Pattern

**Universal DFS Template**:
```python
def dfs_template(root):
    """
    Universal DFS pattern for trees
    Works for: Preorder, Inorder, Postorder
    """
    result = []
    
    def dfs(node):
        # Base case
        if not node:
            return
        
        # Preorder: Process before children
        # result.append(node.val)
        
        # Traverse left
        dfs(node.left)
        
        # Inorder: Process between children
        # result.append(node.val)
        
        # Traverse right
        dfs(node.right)
        
        # Postorder: Process after children
        # result.append(node.val)
    
    dfs(root)
    return result
```

### 1.2 Key Variations

**Variation 1: Preorder (Root → Left → Right)**
```python
def preorder_traversal(root):
    """Preorder: Process root before children"""
    result = []
    
    def dfs(node):
        if node:
            result.append(node.val)  # Process root
            dfs(node.left)
            dfs(node.right)
    
    dfs(root)
    return result

# Use cases: Tree copying, prefix expressions, tree serialization
```

**Variation 2: Inorder (Left → Root → Right)**
```python
def inorder_traversal(root):
    """Inorder: Process root between children"""
    result = []
    
    def dfs(node):
        if node:
            dfs(node.left)
            result.append(node.val)  # Process root
            dfs(node.right)
    
    dfs(root)
    return result

# Use cases: BST validation, BST to sorted array
```

**Variation 3: Postorder (Left → Right → Root)**
```python
def postorder_traversal(root):
    """Postorder: Process root after children"""
    result = []
    
    def dfs(node):
        if node:
            dfs(node.left)
            dfs(node.right)
            result.append(node.val)  # Process root
    
    dfs(root)
    return result

# Use cases: Tree deletion, expression evaluation, tree height
```

**Variation 4: Level-Order (BFS)**
```python
from collections import deque

def level_order_traversal(root):
    """Level-order: Process level by level"""
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

# Use cases: Level-order problems, right side view, level averages
```

### 1.3 Universal Tree DFS Pattern

**Single Template for All Problems**:
```python
def universal_tree_dfs(root):
    """
    Universal pattern: Return value from children, process, return to parent
    """
    def dfs(node):
        # Base case
        if not node:
            return base_value  # 0, [], None, etc.
        
        # Get results from children
        left_result = dfs(node.left)
        right_result = dfs(node.right)
        
        # Process current node with children's results
        current_result = process(node, left_result, right_result)
        
        # Update global result if needed
        update_global_result(current_result)
        
        # Return value to parent
        return current_result
    
    return dfs(root)
```

**Example: Maximum Path Sum**
```python
def max_path_sum(root):
    """Maximum path sum (path can start/end anywhere)"""
    max_sum = [float('-inf')]
    
    def dfs(node):
        if not node:
            return 0
        
        # Get max gain from children (can be negative)
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

## 2. Tree Construction Patterns

### 2.1 Core Pattern

**Divide and Conquer Template**:
```python
def build_tree_template(data):
    """
    Universal tree construction pattern
    """
    def build(left, right):
        # Base case
        if left > right:
            return None
        
        # Find root (varies by problem)
        root_index = find_root(left, right)
        root = TreeNode(data[root_index])
        
        # Recursively build left and right subtrees
        root.left = build(left, root_index - 1)
        root.right = build(root_index + 1, right)
        
        return root
    
    return build(0, len(data) - 1)
```

### 2.2 Key Applications

**Application 1: From Preorder and Inorder**
```python
def build_tree_pre_inorder(preorder, inorder):
    """Build tree from preorder and inorder"""
    if not preorder or not inorder:
        return None
    
    # Root is first in preorder
    root_val = preorder[0]
    root = TreeNode(root_val)
    
    # Find root in inorder
    root_index = inorder.index(root_val)
    
    # Left subtree: inorder[:root_index]
    # Right subtree: inorder[root_index+1:]
    root.left = build_tree_pre_inorder(
        preorder[1:root_index+1],
        inorder[:root_index]
    )
    root.right = build_tree_pre_inorder(
        preorder[root_index+1:],
        inorder[root_index+1:]
    )
    
    return root
```

**Application 2: From Sorted Array (BST)**
```python
def sorted_array_to_bst(nums):
    """Build balanced BST from sorted array"""
    def build(left, right):
        if left > right:
            return None
        
        mid = (left + right) // 2
        root = TreeNode(nums[mid])
        
        root.left = build(left, mid - 1)
        root.right = build(mid + 1, right)
        
        return root
    
    return build(0, len(nums) - 1)
```

---

## 3. Tree Property Patterns

### 3.1 Core Pattern: Return Multiple Values

**Pattern Template**:
```python
def tree_property_template(root):
    """
    Pattern: Return multiple values from recursion
    Useful for: Height, diameter, balanced, etc.
    """
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

### 3.2 Key Applications

**Application 1: Check Balanced Tree**
```python
def is_balanced(root):
    """Check if tree is height-balanced"""
    def dfs(node):
        if not node:
            return (0, True)  # (height, is_balanced)
        
        left_height, left_balanced = dfs(node.left)
        right_height, right_balanced = dfs(node.right)
        
        balanced = (left_balanced and right_balanced and
                   abs(left_height - right_height) <= 1)
        height = 1 + max(left_height, right_height)
        
        return (height, balanced)
    
    _, balanced = dfs(root)
    return balanced
```

**Application 2: Diameter of Tree**
```python
def diameter_of_tree(root):
    """Longest path between any two nodes"""
    max_diameter = [0]
    
    def dfs(node):
        if not node:
            return 0
        
        left_height = dfs(node.left)
        right_height = dfs(node.right)
        
        # Diameter passing through this node
        max_diameter[0] = max(max_diameter[0], left_height + right_height)
        
        return 1 + max(left_height, right_height)
    
    dfs(root)
    return max_diameter[0]
```

**Application 3: Lowest Common Ancestor**
```python
def lowest_common_ancestor(root, p, q):
    """Find LCA of two nodes"""
    if not root or root == p or root == q:
        return root
    
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    
    if left and right:
        return root
    return left or right
```

---

## 4. Graph Traversal Patterns

### 4.1 Core DFS Pattern

**Universal Graph DFS Template**:
```python
def graph_dfs_template(graph, start):
    """
    Universal DFS pattern for graphs
    """
    visited = set()
    result = []
    
    def dfs(node):
        # Mark as visited
        visited.add(node)
        result.append(node)
        
        # Explore neighbors
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs(neighbor)
    
    dfs(start)
    return result
```

### 4.2 Key Variations

**Variation 1: DFS for Connected Components**
```python
def connected_components(graph):
    """Find all connected components"""
    visited = set()
    components = []
    
    def dfs(node, component):
        visited.add(node)
        component.append(node)
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs(neighbor, component)
    
    for node in graph:
        if node not in visited:
            component = []
            dfs(node, component)
            components.append(component)
    
    return components
```

**Variation 2: DFS for Cycle Detection**
```python
def has_cycle_directed(graph):
    """Detect cycle in directed graph"""
    WHITE, GRAY, BLACK = 0, 1, 2
    color = {}
    
    def dfs(node):
        color[node] = GRAY
        
        for neighbor in graph.get(node, []):
            if neighbor not in color:
                if dfs(neighbor):
                    return True
            elif color[neighbor] == GRAY:
                return True  # Back edge found
        
        color[node] = BLACK
        return False
    
    for node in graph:
        if node not in color:
            if dfs(node):
                return True
    
    return False
```

**Variation 3: DFS for Path Finding**
```python
def find_path(graph, start, end):
    """Find path from start to end"""
    visited = set()
    path = []
    
    def dfs(node):
        visited.add(node)
        path.append(node)
        
        if node == end:
            return True
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                if dfs(neighbor):
                    return True
        
        path.pop()  # Backtrack
        return False
    
    if dfs(start):
        return path
    return []
```

### 4.3 Core BFS Pattern

**Universal Graph BFS Template**:
```python
from collections import deque

def graph_bfs_template(graph, start):
    """
    Universal BFS pattern for graphs
    """
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
```

**Variation: BFS for Shortest Path**
```python
def shortest_path_bfs(graph, start, end):
    """Shortest path in unweighted graph"""
    if start == end:
        return [start]
    
    queue = deque([start])
    visited = {start}
    parent = {start: None}
    
    while queue:
        node = queue.popleft()
        
        if node == end:
            # Reconstruct path
            path = []
            current = end
            while current is not None:
                path.append(current)
                current = parent[current]
            return path[::-1]
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                parent[neighbor] = node
                queue.append(neighbor)
    
    return []  # No path
```

---

## 5. Graph Algorithm Patterns

### 5.1 Topological Sort Pattern

**Kahn's Algorithm (BFS-based)**:
```python
def topological_sort_kahn(graph, num_nodes):
    """Topological sort using Kahn's algorithm"""
    # Calculate in-degrees
    in_degree = [0] * num_nodes
    for node in graph:
        for neighbor in graph[node]:
            in_degree[neighbor] += 1
    
    # Start with nodes having in-degree 0
    queue = deque([i for i in range(num_nodes) if in_degree[i] == 0])
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node)
        
        for neighbor in graph.get(node, []):
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    
    # Check for cycle
    if len(result) != num_nodes:
        return []  # Cycle exists
    
    return result
```

**DFS-based Topological Sort**:
```python
def topological_sort_dfs(graph, num_nodes):
    """Topological sort using DFS"""
    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * num_nodes
    result = []
    has_cycle = [False]
    
    def dfs(node):
        if has_cycle[0]:
            return
        
        color[node] = GRAY
        
        for neighbor in graph.get(node, []):
            if color[neighbor] == WHITE:
                dfs(neighbor)
            elif color[neighbor] == GRAY:
                has_cycle[0] = True
                return
        
        color[node] = BLACK
        result.append(node)
    
    for node in range(num_nodes):
        if color[node] == WHITE:
            dfs(node)
    
    if has_cycle[0]:
        return []
    
    return result[::-1]
```

### 5.2 Shortest Path Patterns

**Dijkstra's Algorithm**:
```python
import heapq

def dijkstra(graph, start):
    """Shortest path from start to all nodes"""
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    heap = [(0, start)]
    visited = set()
    
    while heap:
        current_dist, current = heapq.heappop(heap)
        
        if current in visited:
            continue
        
        visited.add(current)
        
        for neighbor, weight in graph.get(current, []):
            if neighbor in visited:
                continue
            
            new_dist = current_dist + weight
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                heapq.heappush(heap, (new_dist, neighbor))
    
    return distances
```

### 5.3 Union-Find Pattern

**Disjoint Set Union**:
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False
        
        if self.rank[root_x] < self.rank[root_y]:
            root_x, root_y = root_y, root_x
        
        self.parent[root_y] = root_x
        if self.rank[root_x] == self.rank[root_y]:
            self.rank[root_x] += 1
        
        return True
```

---

## 6. Master Tree/Graph Recognition

### 6.1 Decision Tree

```python
"""
Tree/Graph Problem Analysis:

1. Is it a tree problem?
   YES → 
      a. Need traversal? → DFS/BFS
      b. Need construction? → Divide & Conquer
      c. Need properties? → Return multiple values
      d. Need path? → DFS with backtracking
   
2. Is it a graph problem?
   YES →
      a. Need shortest path? → BFS (unweighted) or Dijkstra (weighted)
      b. Need connectivity? → DFS/BFS
      c. Need ordering? → Topological Sort
      d. Need cycles? → DFS with color tracking
      e. Need MST? → Kruskal/Prim
"""

def identify_tree_graph_pattern(problem):
    """
    Pattern identification for tree/graph problems
    """
    if "tree" in problem.lower() or "binary tree" in problem.lower():
        if "traverse" in problem.lower() or "path" in problem.lower():
            return "DFS/BFS"
        elif "construct" in problem.lower() or "build" in problem.lower():
            return "Divide & Conquer"
        elif "balanced" in problem.lower() or "height" in problem.lower():
            return "Tree Property (Multiple Returns)"
    
    if "graph" in problem.lower() or "node" in problem.lower():
        if "shortest" in problem.lower() or "path" in problem.lower():
            return "BFS/Dijkstra"
        elif "connected" in problem.lower():
            return "DFS/BFS"
        elif "order" in problem.lower() or "schedule" in problem.lower():
            return "Topological Sort"
    
    return "Unknown"
```

### 6.2 Universal Templates

**Universal Tree Template**:
```python
def solve_tree_problem(root):
    """
    Universal template for most tree problems
    """
    def dfs(node):
        # Base case
        if not node:
            return base_value
        
        # Get results from children
        left = dfs(node.left)
        right = dfs(node.right)
        
        # Process current node
        result = process(node, left, right)
        
        # Update global if needed
        update_global(result)
        
        return result
    
    return dfs(root)
```

**Universal Graph Template**:
```python
def solve_graph_problem(graph, start):
    """
    Universal template for most graph problems
    """
    visited = set()
    result = []
    
    def dfs(node):
        visited.add(node)
        # Process node
        process(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs(neighbor)
    
    # Or use BFS
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

---

## Summary: Tree and Graph Patterns

### Core Patterns

1. **Tree DFS**: Universal template with return values
2. **Tree BFS**: Level-order traversal
3. **Tree Construction**: Divide and conquer
4. **Tree Properties**: Return multiple values
5. **Graph DFS**: Connected components, cycles
6. **Graph BFS**: Shortest path, level-order
7. **Topological Sort**: Ordering, dependencies
8. **Union-Find**: Connectivity, cycles

### Pattern Selection Guide

```
Tree Problem Type → Pattern:
- Traversal → DFS/BFS
- Construction → Divide & Conquer
- Properties → Multiple Returns
- Path → DFS with Backtracking

Graph Problem Type → Pattern:
- Shortest Path → BFS/Dijkstra
- Connectivity → DFS/BFS
- Ordering → Topological Sort
- Cycles → DFS with Colors
- MST → Kruskal/Prim
```

### Next Steps

**Part 3** will cover:
- Dynamic Programming Patterns
- DP State Transitions
- DP Optimization

---

**Master these tree/graph patterns to solve 80% of tree and graph problems!**

