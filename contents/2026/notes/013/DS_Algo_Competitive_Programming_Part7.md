# Data Structures & Algorithms for Competitive Programming

## Part 7: Advanced Data Structures

---

## Table of Contents

1. [Trie (Prefix Tree)](#1-trie-prefix-tree)
2. [Segment Tree](#2-segment-tree)
3. [Fenwick Tree (Binary Indexed Tree)](#3-fenwick-tree-binary-indexed-tree)
4. [Disjoint Set Union (DSU)](#4-disjoint-set-union-dsu)
5. [Practice Problems](#5-practice-problems)

---

## 1. Trie (Prefix Tree)

### 1.1 Trie Implementation

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        """
        Insert word into trie
        Time: O(m) where m is word length, Space: O(m)
        """
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end = True
    
    def search(self, word):
        """
        Search for exact word
        Time: O(m), Space: O(1)
        """
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end
    
    def starts_with(self, prefix):
        """
        Check if prefix exists
        Time: O(m), Space: O(1)
        """
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
    
    def delete(self, word):
        """
        Delete word from trie
        Time: O(m), Space: O(1)
        """
        def _delete(node, word, index):
            if index == len(word):
                if not node.is_end:
                    return False
                node.is_end = False
                return len(node.children) == 0
            
            char = word[index]
            if char not in node.children:
                return False
            
            should_delete = _delete(node.children[char], word, index + 1)
            
            if should_delete:
                del node.children[char]
                return len(node.children) == 0 and not node.is_end
            
            return False
        
        _delete(self.root, word, 0)
```

### 1.2 Trie Applications

**Word Search II**:
```python
def find_words(board, words):
    """
    Find all words from dictionary in 2D board
    Time: O(m * n * 4^L), Space: O(ALPHABET_SIZE * N * M)
    """
    trie = Trie()
    for word in words:
        trie.insert(word)
    
    rows, cols = len(board), len(board[0])
    result = set()
    
    def backtrack(row, col, node, path):
        char = board[row][col]
        if char not in node.children:
            return
        
        node = node.children[char]
        path += char
        
        if node.is_end:
            result.add(path)
        
        # Mark as visited
        temp = board[row][col]
        board[row][col] = '#'
        
        # Try all directions
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            new_row, new_col = row + dr, col + dc
            if 0 <= new_row < rows and 0 <= new_col < cols:
                backtrack(new_row, new_col, node, path)
        
        board[row][col] = temp
    
    for i in range(rows):
        for j in range(cols):
            backtrack(i, j, trie.root, "")
    
    return list(result)
```

**Maximum XOR**:
```python
class TrieNode:
    def __init__(self):
        self.children = {}  # 0 or 1

def find_maximum_xor(nums):
    """
    Find maximum XOR of two numbers in array
    Time: O(n * 32), Space: O(n * 32)
    """
    root = TrieNode()
    
    # Build trie
    for num in nums:
        node = root
        for i in range(31, -1, -1):
            bit = (num >> i) & 1
            if bit not in node.children:
                node.children[bit] = TrieNode()
            node = node.children[bit]
    
    max_xor = 0
    
    for num in nums:
        node = root
        current_xor = 0
        for i in range(31, -1, -1):
            bit = (num >> i) & 1
            # Try opposite bit for maximum XOR
            opposite = 1 - bit
            if opposite in node.children:
                current_xor |= (1 << i)
                node = node.children[opposite]
            else:
                node = node.children[bit]
        max_xor = max(max_xor, current_xor)
    
    return max_xor
```

---

## 2. Segment Tree

### 2.1 Segment Tree Implementation

```python
class SegmentTree:
    def __init__(self, arr):
        """
        Build segment tree for range sum queries
        Time: O(n), Space: O(n)
        """
        self.n = len(arr)
        self.size = 1
        while self.size < self.n:
            self.size *= 2
        self.tree = [0] * (2 * self.size)
        
        # Build tree
        for i in range(self.n):
            self.tree[self.size + i] = arr[i]
        for i in range(self.size - 1, 0, -1):
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]
    
    def update(self, index, value):
        """
        Update value at index
        Time: O(log n), Space: O(1)
        """
        index += self.size
        self.tree[index] = value
        index //= 2
        
        while index:
            self.tree[index] = self.tree[2 * index] + self.tree[2 * index + 1]
            index //= 2
    
    def query(self, left, right):
        """
        Range sum query [left, right]
        Time: O(log n), Space: O(1)
        """
        left += self.size
        right += self.size
        result = 0
        
        while left <= right:
            if left % 2 == 1:
                result += self.tree[left]
                left += 1
            if right % 2 == 0:
                result += self.tree[right]
                right -= 1
            left //= 2
            right //= 2
        
        return result
```

### 2.2 Segment Tree for Range Minimum

```python
class SegmentTreeMin:
    def __init__(self, arr):
        self.n = len(arr)
        self.size = 1
        while self.size < self.n:
            self.size *= 2
        self.tree = [float('inf')] * (2 * self.size)
        
        for i in range(self.n):
            self.tree[self.size + i] = arr[i]
        for i in range(self.size - 1, 0, -1):
            self.tree[i] = min(self.tree[2 * i], self.tree[2 * i + 1])
    
    def update(self, index, value):
        index += self.size
        self.tree[index] = value
        index //= 2
        
        while index:
            self.tree[index] = min(self.tree[2 * index], self.tree[2 * index + 1])
            index //= 2
    
    def query(self, left, right):
        left += self.size
        right += self.size
        result = float('inf')
        
        while left <= right:
            if left % 2 == 1:
                result = min(result, self.tree[left])
                left += 1
            if right % 2 == 0:
                result = min(result, self.tree[right])
                right -= 1
            left //= 2
            right //= 2
        
        return result
```

---

## 3. Fenwick Tree (Binary Indexed Tree)

### 3.1 Fenwick Tree Implementation

```python
class FenwickTree:
    def __init__(self, n):
        """
        Initialize Fenwick Tree
        Time: O(n), Space: O(n)
        """
        self.n = n
        self.tree = [0] * (n + 1)
    
    def update(self, index, delta):
        """
        Update value at index (add delta)
        Time: O(log n), Space: O(1)
        """
        index += 1
        while index <= self.n:
            self.tree[index] += delta
            index += index & -index  # Add last set bit
    
    def query(self, index):
        """
        Prefix sum from 0 to index
        Time: O(log n), Space: O(1)
        """
        index += 1
        result = 0
        while index > 0:
            result += self.tree[index]
            index -= index & -index  # Remove last set bit
        return result
    
    def range_query(self, left, right):
        """
        Range sum query [left, right]
        Time: O(log n), Space: O(1)
        """
        return self.query(right) - self.query(left - 1)
```

### 3.2 2D Fenwick Tree

```python
class FenwickTree2D:
    def __init__(self, rows, cols):
        self.rows = rows
        self.cols = cols
        self.tree = [[0] * (cols + 1) for _ in range(rows + 1)]
    
    def update(self, row, col, delta):
        i = row + 1
        while i <= self.rows:
            j = col + 1
            while j <= self.cols:
                self.tree[i][j] += delta
                j += j & -j
            i += i & -i
    
    def query(self, row, col):
        result = 0
        i = row + 1
        while i > 0:
            j = col + 1
            while j > 0:
                result += self.tree[i][j]
                j -= j & -j
            i -= i & -i
        return result
    
    def range_query(self, row1, col1, row2, col2):
        return (self.query(row2, col2) -
                self.query(row1 - 1, col2) -
                self.query(row2, col1 - 1) +
                self.query(row1 - 1, col1 - 1))
```

---

## 4. Disjoint Set Union (DSU)

### 4.1 DSU Implementation

```python
class DSU:
    def __init__(self, n):
        """
        Initialize DSU with n elements
        Time: O(n), Space: O(n)
        """
        self.parent = list(range(n))
        self.rank = [0] * n
        self.size = [1] * n
    
    def find(self, x):
        """
        Find root with path compression
        Time: O(α(n)) amortized, Space: O(1)
        """
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]
    
    def union(self, x, y):
        """
        Union by rank
        Time: O(α(n)) amortized, Space: O(1)
        """
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False
        
        if self.rank[root_x] < self.rank[root_y]:
            root_x, root_y = root_y, root_x
        
        self.parent[root_y] = root_x
        self.size[root_x] += self.size[root_y]
        
        if self.rank[root_x] == self.rank[root_y]:
            self.rank[root_x] += 1
        
        return True
    
    def connected(self, x, y):
        """Check if x and y are in same set"""
        return self.find(x) == self.find(y)
    
    def get_size(self, x):
        """Get size of set containing x"""
        return self.size[self.find(x)]
```

### 4.2 Applications

**Number of Islands II**:
```python
def num_islands_ii(m, n, positions):
    """
    Count islands after each land addition
    Time: O(k * α(mn)), Space: O(mn)
    """
    dsu = DSU(m * n)
    grid = [[0] * n for _ in range(m)]
    result = []
    count = 0
    
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    
    for row, col in positions:
        if grid[row][col] == 1:
            result.append(count)
            continue
        
        grid[row][col] = 1
        count += 1
        index = row * n + col
        
        for dr, dc in directions:
            new_row, new_col = row + dr, col + dc
            if (0 <= new_row < m and 0 <= new_col < n and
                grid[new_row][new_col] == 1):
                neighbor_index = new_row * n + new_col
                if dsu.union(index, neighbor_index):
                    count -= 1
        
        result.append(count)
    
    return result
```

---

## 5. Practice Problems

### Problem Set 1: Trie

1. **Implement Trie (Prefix Tree)**
```python
# See Trie implementation above
```

2. **Add and Search Words**
```python
class WordDictionary:
    def __init__(self):
        self.trie = Trie()
    
    def add_word(self, word):
        self.trie.insert(word)
    
    def search(self, word):
        """
        Search with '.' wildcard
        Time: O(26^m) worst case, Space: O(m)
        """
        def dfs(node, index):
            if index == len(word):
                return node.is_end
            
            char = word[index]
            if char == '.':
                for child in node.children.values():
                    if dfs(child, index + 1):
                        return True
                return False
            else:
                if char not in node.children:
                    return False
                return dfs(node.children[char], index + 1)
        
        return dfs(self.trie.root, 0)
```

### Problem Set 2: Segment Tree

1. **Range Sum Query - Mutable**
```python
# Use SegmentTree implementation above
```

2. **Count of Smaller Numbers After Self**
```python
def count_smaller(nums):
    """
    Count smaller numbers to the right
    Time: O(n log n), Space: O(n)
    """
    # Coordinate compression
    sorted_nums = sorted(set(nums))
    rank = {num: i + 1 for i, num in enumerate(sorted_nums)}
    
    n = len(sorted_nums)
    fenwick = FenwickTree(n)
    result = []
    
    for num in reversed(nums):
        r = rank[num]
        result.append(fenwick.query(r - 1))
        fenwick.update(r, 1)
    
    return result[::-1]
```

---

## Summary of Part 7

### Advanced Data Structures

1. **Trie**: Prefix matching, word search
2. **Segment Tree**: Range queries and updates
3. **Fenwick Tree**: Efficient prefix sums
4. **DSU**: Union-Find operations

### When to Use

- **Trie**: String prefix problems, word search
- **Segment Tree**: Range queries with updates
- **Fenwick Tree**: Prefix sum problems
- **DSU**: Connected components, cycle detection

### Next Steps

**Part 8** will cover:
- String Algorithms (KMP, Z-algorithm, Rabin-Karp)
- Pattern Matching

---

**Practice advanced data structure problems!**

