# Data Structures & Algorithms for Competitive Programming

## Part 4: Graphs

---

## Table of Contents

1. [Graph Representation](#1-graph-representation)
2. [Graph Traversals](#2-graph-traversals)
3. [Shortest Path Algorithms](#3-shortest-path-algorithms)
4. [Minimum Spanning Tree](#4-minimum-spanning-tree)
5. [Topological Sort](#5-topological-sort)
6. [Union-Find (Disjoint Set)](#6-union-find-disjoint-set)
7. [Practice Problems](#7-practice-problems)

---

## 1. Graph Representation

### 1.1 Adjacency List

```python
# Undirected graph
graph = {
    0: [1, 2],
    1: [0, 3],
    2: [0, 3, 4],
    3: [1, 2],
    4: [2]
}

# Directed graph
digraph = {
    0: [1, 2],
    1: [3],
    2: [3, 4],
    3: [],
    4: []
}

# Weighted graph
weighted_graph = {
    0: [(1, 5), (2, 3)],
    1: [(0, 5), (3, 2)],
    2: [(0, 3), (3, 1), (4, 4)],
    3: [(1, 2), (2, 1)],
    4: [(2, 4)]
}
```

### 1.2 Adjacency Matrix

```python
# Unweighted graph
adj_matrix = [
    [0, 1, 1, 0, 0],
    [1, 0, 0, 1, 0],
    [1, 0, 0, 1, 1],
    [0, 1, 1, 0, 0],
    [0, 0, 1, 0, 0]
]

# Weighted graph (use float('inf') for no edge)
weighted_matrix = [
    [0, 5, 3, float('inf'), float('inf')],
    [5, 0, float('inf'), 2, float('inf')],
    [3, float('inf'), 0, 1, 4],
    [float('inf'), 2, 1, 0, float('inf')],
    [float('inf'), float('inf'), 4, float('inf'), 0]
]
```

### 1.3 Edge List

```python
# List of edges
edges = [
    (0, 1),
    (0, 2),
    (1, 3),
    (2, 3),
    (2, 4)
]

# Weighted edges
weighted_edges = [
    (0, 1, 5),
    (0, 2, 3),
    (1, 3, 2),
    (2, 3, 1),
    (2, 4, 4)
]
```

---

## 2. Graph Traversals

### 2.1 Depth-First Search (DFS)

```python
def dfs(graph, start):
    """
    DFS traversal
    Time: O(V + E), Space: O(V)
    """
    visited = set()
    result = []
    
    def dfs_helper(node):
        visited.add(node)
        result.append(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                dfs_helper(neighbor)
    
    dfs_helper(start)
    return result

def dfs_iterative(graph, start):
    """Iterative DFS"""
    visited = set()
    stack = [start]
    result = []
    
    while stack:
        node = stack.pop()
        if node not in visited:
            visited.add(node)
            result.append(node)
            # Add neighbors in reverse to maintain order
            for neighbor in reversed(graph.get(node, [])):
                if neighbor not in visited:
                    stack.append(neighbor)
    
    return result
```

### 2.2 Breadth-First Search (BFS)

```python
from collections import deque

def bfs(graph, start):
    """
    BFS traversal
    Time: O(V + E), Space: O(V)
    """
    visited = set()
    queue = deque([start])
    visited.add(start)
    result = []
    
    while queue:
        node = queue.popleft()
        result.append(node)
        
        for neighbor in graph.get(node, []):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    
    return result

def bfs_level_order(graph, start):
    """BFS with level information"""
    visited = set()
    queue = deque([start])
    visited.add(start)
    levels = []
    
    while queue:
        level_size = len(queue)
        level = []
        
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node)
            
            for neighbor in graph.get(node, []):
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)
        
        levels.append(level)
    
    return levels
```

### 2.3 Connected Components

```python
def connected_components(graph):
    """
    Find all connected components in undirected graph
    Time: O(V + E), Space: O(V)
    """
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

def num_islands(grid):
    """
    Count number of islands (connected components of '1's)
    Time: O(m * n), Space: O(m * n)
    """
    if not grid:
        return 0
    
    rows, cols = len(grid), len(grid[0])
    visited = set()
    islands = 0
    
    def dfs(r, c):
        if (r < 0 or r >= rows or c < 0 or c >= cols or
            (r, c) in visited or grid[r][c] == '0'):
            return
        
        visited.add((r, c))
        # Check all 4 directions
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
    
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1' and (r, c) not in visited:
                dfs(r, c)
                islands += 1
    
    return islands
```

---

## 3. Shortest Path Algorithms

### 3.1 BFS for Unweighted Graphs

```python
def shortest_path_bfs(graph, start, end):
    """
    Shortest path in unweighted graph using BFS
    Time: O(V + E), Space: O(V)
    """
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
    
    return []  # No path found
```

### 3.2 Dijkstra's Algorithm

```python
import heapq

def dijkstra(graph, start):
    """
    Shortest path from start to all nodes (weighted graph, non-negative)
    Time: O((V + E) log V), Space: O(V)
    """
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    parent = {start: None}
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
                parent[neighbor] = current
                heapq.heappush(heap, (new_dist, neighbor))
    
    return distances, parent

def shortest_path_dijkstra(graph, start, end):
    """Get shortest path from start to end"""
    distances, parent = dijkstra(graph, start)
    
    if distances[end] == float('inf'):
        return []
    
    # Reconstruct path
    path = []
    current = end
    while current is not None:
        path.append(current)
        current = parent[current]
    
    return path[::-1]
```

### 3.3 Bellman-Ford Algorithm

```python
def bellman_ford(edges, num_nodes, start):
    """
    Shortest path with negative weights (detects negative cycles)
    Time: O(V * E), Space: O(V)
    """
    distances = [float('inf')] * num_nodes
    distances[start] = 0
    
    # Relax edges V-1 times
    for _ in range(num_nodes - 1):
        for u, v, w in edges:
            if distances[u] != float('inf') and distances[u] + w < distances[v]:
                distances[v] = distances[u] + w
    
    # Check for negative cycles
    for u, v, w in edges:
        if distances[u] != float('inf') and distances[u] + w < distances[v]:
            return None  # Negative cycle detected
    
    return distances
```

### 3.4 Floyd-Warshall Algorithm

```python
def floyd_warshall(graph, num_nodes):
    """
    All-pairs shortest paths
    Time: O(V^3), Space: O(V^2)
    """
    # Initialize distance matrix
    dist = [[float('inf')] * num_nodes for _ in range(num_nodes)]
    
    # Distance from node to itself is 0
    for i in range(num_nodes):
        dist[i][i] = 0
    
    # Initialize with direct edges
    for u in graph:
        for v, w in graph[u]:
            dist[u][v] = w
    
    # Floyd-Warshall algorithm
    for k in range(num_nodes):
        for i in range(num_nodes):
            for j in range(num_nodes):
                if dist[i][k] != float('inf') and dist[k][j] != float('inf'):
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
    
    return dist
```

---

## 4. Minimum Spanning Tree

### 4.1 Kruskal's Algorithm

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Path compression
        return self.parent[x]
    
    def union(self, x, y):
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False
        
        # Union by rank
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1
        
        return True

def kruskal(edges, num_nodes):
    """
    Minimum Spanning Tree using Kruskal's algorithm
    Time: O(E log E), Space: O(V)
    """
    # Sort edges by weight
    edges.sort(key=lambda x: x[2])
    
    uf = UnionFind(num_nodes)
    mst = []
    total_weight = 0
    
    for u, v, weight in edges:
        if uf.union(u, v):
            mst.append((u, v, weight))
            total_weight += weight
            if len(mst) == num_nodes - 1:
                break
    
    return mst, total_weight
```

### 4.2 Prim's Algorithm

```python
import heapq

def prim(graph, start, num_nodes):
    """
    Minimum Spanning Tree using Prim's algorithm
    Time: O((V + E) log V), Space: O(V)
    """
    visited = set()
    mst = []
    total_weight = 0
    heap = [(0, start, None)]  # (weight, node, parent)
    
    while heap and len(visited) < num_nodes:
        weight, node, parent = heapq.heappop(heap)
        
        if node in visited:
            continue
        
        visited.add(node)
        if parent is not None:
            mst.append((parent, node, weight))
            total_weight += weight
        
        for neighbor, edge_weight in graph.get(node, []):
            if neighbor not in visited:
                heapq.heappush(heap, (edge_weight, neighbor, node))
    
    return mst, total_weight
```

---

## 5. Topological Sort

### 5.1 Kahn's Algorithm (BFS-based)

```python
from collections import deque

def topological_sort_kahn(graph, num_nodes):
    """
    Topological sort using Kahn's algorithm
    Time: O(V + E), Space: O(V)
    """
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

### 5.2 DFS-based Topological Sort

```python
def topological_sort_dfs(graph, num_nodes):
    """
    Topological sort using DFS
    Time: O(V + E), Space: O(V)
    """
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
    
    return result[::-1]  # Reverse for topological order
```

---

## 6. Union-Find (Disjoint Set)

### 6.1 Union-Find Implementation

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        """Find with path compression"""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """Union by rank"""
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False
        
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1
        
        self.components -= 1
        return True
    
    def connected(self, x, y):
        """Check if two elements are in same set"""
        return self.find(x) == self.find(y)
```

### 6.2 Applications

**Number of Connected Components**:
```python
def num_connected_components(n, edges):
    """
    Count connected components using Union-Find
    Time: O(E * α(V)), Space: O(V)
    """
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.components
```

**Redundant Connection**:
```python
def find_redundant_connection(edges):
    """
    Find edge that creates cycle (last edge that connects already connected nodes)
    Time: O(E * α(V)), Space: O(V)
    """
    n = len(edges)
    uf = UnionFind(n + 1)
    
    for u, v in edges:
        if not uf.union(u, v):
            return [u, v]
    
    return []
```

---

## 7. Practice Problems

### Problem Set 1: Graph Traversals

1. **Clone Graph**
```python
def clone_graph(node):
    """
    Deep copy undirected graph
    Time: O(V + E), Space: O(V)
    """
    if not node:
        return None
    
    clone_map = {}
    
    def dfs(original):
        if original in clone_map:
            return clone_map[original]
        
        clone = Node(original.val)
        clone_map[original] = clone
        
        for neighbor in original.neighbors:
            clone.neighbors.append(dfs(neighbor))
        
        return clone
    
    return dfs(node)
```

2. **Course Schedule**
```python
def can_finish(num_courses, prerequisites):
    """
    Check if can finish all courses (no cycle in DAG)
    Time: O(V + E), Space: O(V)
    """
    graph = {i: [] for i in range(num_courses)}
    in_degree = [0] * num_courses
    
    for course, prereq in prerequisites:
        graph[prereq].append(course)
        in_degree[course] += 1
    
    queue = deque([i for i in range(num_courses) if in_degree[i] == 0])
    completed = 0
    
    while queue:
        course = queue.popleft()
        completed += 1
        
        for next_course in graph[course]:
            in_degree[next_course] -= 1
            if in_degree[next_course] == 0:
                queue.append(next_course)
    
    return completed == num_courses
```

### Problem Set 2: Shortest Path

1. **Network Delay Time**
```python
def network_delay_time(times, n, k):
    """
    Time for signal to reach all nodes from node k
    Time: O((V + E) log V), Space: O(V)
    """
    graph = {i: [] for i in range(1, n + 1)}
    for u, v, w in times:
        graph[u].append((v, w))
    
    distances = {i: float('inf') for i in range(1, n + 1)}
    distances[k] = 0
    heap = [(0, k)]
    visited = set()
    
    while heap:
        time, node = heapq.heappop(heap)
        if node in visited:
            continue
        visited.add(node)
        
        for neighbor, weight in graph[node]:
            new_time = time + weight
            if new_time < distances[neighbor]:
                distances[neighbor] = new_time
                heapq.heappush(heap, (new_time, neighbor))
    
    max_time = max(distances.values())
    return max_time if max_time != float('inf') else -1
```

### Problem Set 3: MST and Union-Find

1. **Connecting Cities with Minimum Cost**
```python
def minimum_cost(n, connections):
    """
    Minimum cost to connect all cities
    Time: O(E log E), Space: O(V)
    """
    uf = UnionFind(n)
    connections.sort(key=lambda x: x[2])
    total_cost = 0
    
    for city1, city2, cost in connections:
        if uf.union(city1 - 1, city2 - 1):
            total_cost += cost
            if uf.components == 1:
                break
    
    return total_cost if uf.components == 1 else -1
```

---

## Summary of Part 4

### Key Algorithms

1. **Graph Traversals**: DFS, BFS
2. **Shortest Path**: BFS, Dijkstra, Bellman-Ford, Floyd-Warshall
3. **MST**: Kruskal, Prim
4. **Topological Sort**: Kahn's, DFS-based
5. **Union-Find**: Disjoint set operations

### Common Patterns

- **DFS/BFS**: Traversal, connected components
- **Shortest Path**: Weighted vs unweighted
- **Cycle Detection**: Topological sort, Union-Find
- **MST**: Connect all nodes with minimum cost

### Next Steps

**Part 5** will cover:
- Dynamic Programming (1D, 2D, Advanced)
- Memoization
- DP Patterns

---

**Practice graph problems before moving to Part 5!**

