# Data Structures & Algorithms for Competitive Programming

## Part 3: Trees and Binary Trees

---

## Table of Contents

1. [Tree Fundamentals](#1-tree-fundamentals)
2. [Binary Tree Traversals](#2-binary-tree-traversals)
3. [Binary Search Tree](#3-binary-search-tree)
4. [Tree Construction](#4-tree-construction)
5. [Tree Properties](#5-tree-properties)
6. [Practice Problems](#6-practice-problems)

---

## 1. Tree Fundamentals

### 1.1 Tree Node Definition

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# N-ary Tree Node
class Node:
    def __init__(self, val=None, children=None):
        self.val = val
        self.children = children if children is not None else []
```

### 1.2 Basic Tree Properties

```python
def tree_height(root):
    """
    Calculate height of tree
    Time: O(n), Space: O(h) where h is height
    """
    if not root:
        return 0
    return 1 + max(tree_height(root.left), tree_height(root.right))

def tree_size(root):
    """
    Count number of nodes
    Time: O(n), Space: O(h)
    """
    if not root:
        return 0
    return 1 + tree_size(root.left) + tree_size(root.right)

def is_balanced(root):
    """
    Check if tree is height-balanced
    Time: O(n), Space: O(h)
    """
    def check_balance(node):
        if not node:
            return 0, True
        
        left_height, left_balanced = check_balance(node.left)
        right_height, right_balanced = check_balance(node.right)
        
        balanced = (left_balanced and right_balanced and 
                   abs(left_height - right_height) <= 1)
        height = 1 + max(left_height, right_height)
        
        return height, balanced
    
    _, balanced = check_balance(root)
    return balanced
```

---

## 2. Binary Tree Traversals

### 2.1 Depth-First Traversals

**Preorder (Root → Left → Right)**:
```python
def preorder_traversal(root):
    """
    Preorder: Root, Left, Right
    Time: O(n), Space: O(h)
    """
    result = []
    
    def dfs(node):
        if node:
            result.append(node.val)
            dfs(node.left)
            dfs(node.right)
    
    dfs(root)
    return result

def preorder_iterative(root):
    """Iterative version"""
    if not root:
        return []
    
    stack = [root]
    result = []
    
    while stack:
        node = stack.pop()
        result.append(node.val)
        # Push right first, then left (LIFO)
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
    
    return result
```

**Inorder (Left → Root → Right)**:
```python
def inorder_traversal(root):
    """
    Inorder: Left, Root, Right
    Time: O(n), Space: O(h)
    """
    result = []
    
    def dfs(node):
        if node:
            dfs(node.left)
            result.append(node.val)
            dfs(node.right)
    
    dfs(root)
    return result

def inorder_iterative(root):
    """Iterative version"""
    stack = []
    result = []
    current = root
    
    while stack or current:
        # Go to leftmost node
        while current:
            stack.append(current)
            current = current.left
        
        # Process node
        current = stack.pop()
        result.append(current.val)
        
        # Go to right
        current = current.right
    
    return result
```

**Postorder (Left → Right → Root)**:
```python
def postorder_traversal(root):
    """
    Postorder: Left, Right, Root
    Time: O(n), Space: O(h)
    """
    result = []
    
    def dfs(node):
        if node:
            dfs(node.left)
            dfs(node.right)
            result.append(node.val)
    
    dfs(root)
    return result

def postorder_iterative(root):
    """Iterative version using two stacks"""
    if not root:
        return []
    
    stack1 = [root]
    stack2 = []
    
    while stack1:
        node = stack1.pop()
        stack2.append(node)
        if node.left:
            stack1.append(node.left)
        if node.right:
            stack1.append(node.right)
    
    return [node.val for node in reversed(stack2)]
```

### 2.2 Level-Order Traversal (BFS)

```python
from collections import deque

def level_order(root):
    """
    Level-order traversal (BFS)
    Time: O(n), Space: O(n)
    """
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

def level_order_zigzag(root):
    """
    Zigzag level-order traversal
    Time: O(n), Space: O(n)
    """
    if not root:
        return []
    
    queue = deque([root])
    result = []
    left_to_right = True
    
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
        
        if not left_to_right:
            level.reverse()
        result.append(level)
        left_to_right = not left_to_right
    
    return result
```

### 2.3 Morris Traversal (O(1) Space)

```python
def inorder_morris(root):
    """
    Inorder traversal with O(1) space using Morris algorithm
    Time: O(n), Space: O(1)
    """
    result = []
    current = root
    
    while current:
        if not current.left:
            result.append(current.val)
            current = current.right
        else:
            # Find rightmost node in left subtree
            predecessor = current.left
            while predecessor.right and predecessor.right != current:
                predecessor = predecessor.right
            
            if not predecessor.right:
                # Create temporary link
                predecessor.right = current
                current = current.left
            else:
                # Remove temporary link
                predecessor.right = None
                result.append(current.val)
                current = current.right
    
    return result
```

---

## 3. Binary Search Tree

### 3.1 BST Operations

```python
class BST:
    def __init__(self):
        self.root = None
    
    def insert(self, val):
        """Insert value into BST"""
        self.root = self._insert(self.root, val)
    
    def _insert(self, node, val):
        if not node:
            return TreeNode(val)
        
        if val < node.val:
            node.left = self._insert(node.left, val)
        elif val > node.val:
            node.right = self._insert(node.right, val)
        
        return node
    
    def search(self, val):
        """Search for value in BST"""
        return self._search(self.root, val)
    
    def _search(self, node, val):
        if not node or node.val == val:
            return node
        
        if val < node.val:
            return self._search(node.left, val)
        else:
            return self._search(node.right, val)
    
    def delete(self, val):
        """Delete value from BST"""
        self.root = self._delete(self.root, val)
    
    def _delete(self, node, val):
        if not node:
            return None
        
        if val < node.val:
            node.left = self._delete(node.left, val)
        elif val > node.val:
            node.right = self._delete(node.right, val)
        else:
            # Node to delete found
            if not node.left:
                return node.right
            elif not node.right:
                return node.left
            else:
                # Node has two children
                # Find inorder successor (smallest in right subtree)
                successor = self._min_value_node(node.right)
                node.val = successor.val
                node.right = self._delete(node.right, successor.val)
        
        return node
    
    def _min_value_node(self, node):
        """Find node with minimum value"""
        current = node
        while current.left:
            current = current.left
        return current
```

### 3.2 BST Validation

```python
def is_valid_bst(root):
    """
    Validate if tree is valid BST
    Time: O(n), Space: O(h)
    """
    def validate(node, min_val, max_val):
        if not node:
            return True
        
        if node.val <= min_val or node.val >= max_val:
            return False
        
        return (validate(node.left, min_val, node.val) and
                validate(node.right, node.val, max_val))
    
    return validate(root, float('-inf'), float('inf'))

def is_valid_bst_inorder(root):
    """
    Validate using inorder traversal (BST should be sorted)
    Time: O(n), Space: O(h)
    """
    prev = None
    
    def inorder(node):
        nonlocal prev
        if not node:
            return True
        
        if not inorder(node.left):
            return False
        
        if prev is not None and node.val <= prev:
            return False
        
        prev = node.val
        return inorder(node.right)
    
    return inorder(root)
```

### 3.3 BST to Sorted Array

```python
def bst_to_sorted_array(root):
    """
    Convert BST to sorted array
    Time: O(n), Space: O(n)
    """
    result = []
    
    def inorder(node):
        if node:
            inorder(node.left)
            result.append(node.val)
            inorder(node.right)
    
    inorder(root)
    return result
```

---

## 4. Tree Construction

### 4.1 Construct from Traversals

**From Preorder and Inorder**:
```python
def build_tree_pre_inorder(preorder, inorder):
    """
    Build tree from preorder and inorder traversals
    Time: O(n), Space: O(n)
    """
    if not preorder or not inorder:
        return None
    
    # First element of preorder is root
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

# Optimized version with hash map
def build_tree_pre_inorder_optimized(preorder, inorder):
    """Time: O(n), Space: O(n)"""
    inorder_map = {val: i for i, val in enumerate(inorder)}
    pre_index = [0]
    
    def build(left, right):
        if left > right:
            return None
        
        root_val = preorder[pre_index[0]]
        pre_index[0] += 1
        root = TreeNode(root_val)
        
        root_index = inorder_map[root_val]
        root.left = build(left, root_index - 1)
        root.right = build(root_index + 1, right)
        
        return root
    
    return build(0, len(inorder) - 1)
```

**From Postorder and Inorder**:
```python
def build_tree_post_inorder(postorder, inorder):
    """
    Build tree from postorder and inorder traversals
    Time: O(n), Space: O(n)
    """
    if not postorder or not inorder:
        return None
    
    # Last element of postorder is root
    root_val = postorder[-1]
    root = TreeNode(root_val)
    
    root_index = inorder.index(root_val)
    
    root.left = build_tree_post_inorder(
        postorder[:root_index],
        inorder[:root_index]
    )
    root.right = build_tree_post_inorder(
        postorder[root_index:-1],
        inorder[root_index+1:]
    )
    
    return root
```

**From Preorder and Postorder**:
```python
def build_tree_pre_postorder(preorder, postorder):
    """
    Build tree from preorder and postorder (not unique, but can construct)
    Time: O(n), Space: O(n)
    """
    if not preorder:
        return None
    
    root_val = preorder[0]
    root = TreeNode(root_val)
    
    if len(preorder) == 1:
        return root
    
    # Find left subtree root in postorder
    left_root = preorder[1]
    left_root_index = postorder.index(left_root)
    
    root.left = build_tree_pre_postorder(
        preorder[1:left_root_index+2],
        postorder[:left_root_index+1]
    )
    root.right = build_tree_pre_postorder(
        preorder[left_root_index+2:],
        postorder[left_root_index+1:-1]
    )
    
    return root
```

### 4.2 Construct from Array

**From Level-Order Array**:
```python
def build_tree_from_array(arr):
    """
    Build complete binary tree from level-order array
    Time: O(n), Space: O(n)
    """
    if not arr:
        return None
    
    root = TreeNode(arr[0])
    queue = deque([root])
    i = 1
    
    while queue and i < len(arr):
        node = queue.popleft()
        
        if i < len(arr) and arr[i] is not None:
            node.left = TreeNode(arr[i])
            queue.append(node.left)
        i += 1
        
        if i < len(arr) and arr[i] is not None:
            node.right = TreeNode(arr[i])
            queue.append(node.right)
        i += 1
    
    return root
```

---

## 5. Tree Properties

### 5.1 Maximum Depth

```python
def max_depth(root):
    """
    Maximum depth of binary tree
    Time: O(n), Space: O(h)
    """
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

### 5.2 Minimum Depth

```python
def min_depth(root):
    """
    Minimum depth of binary tree
    Time: O(n), Space: O(h)
    """
    if not root:
        return 0
    
    if not root.left:
        return 1 + min_depth(root.right)
    if not root.right:
        return 1 + min_depth(root.left)
    
    return 1 + min(min_depth(root.left), min_depth(root.right))
```

### 5.3 Diameter of Tree

```python
def diameter_of_binary_tree(root):
    """
    Longest path between any two nodes
    Time: O(n), Space: O(h)
    """
    max_diameter = [0]
    
    def height(node):
        if not node:
            return 0
        
        left_height = height(node.left)
        right_height = height(node.right)
        
        # Diameter passing through this node
        max_diameter[0] = max(max_diameter[0], left_height + right_height)
        
        return 1 + max(left_height, right_height)
    
    height(root)
    return max_diameter[0]
```

### 5.4 Path Sum

```python
def has_path_sum(root, target_sum):
    """
    Check if path from root to leaf sums to target
    Time: O(n), Space: O(h)
    """
    if not root:
        return False
    
    if not root.left and not root.right:
        return root.val == target_sum
    
    return (has_path_sum(root.left, target_sum - root.val) or
            has_path_sum(root.right, target_sum - root.val))

def path_sum_ii(root, target_sum):
    """
    Find all paths from root to leaf that sum to target
    Time: O(n^2), Space: O(h)
    """
    result = []
    
    def dfs(node, current_path, remaining_sum):
        if not node:
            return
        
        current_path.append(node.val)
        
        if not node.left and not node.right and remaining_sum == node.val:
            result.append(current_path[:])
        else:
            dfs(node.left, current_path, remaining_sum - node.val)
            dfs(node.right, current_path, remaining_sum - node.val)
        
        current_path.pop()
    
    dfs(root, [], target_sum)
    return result
```

### 5.5 Lowest Common Ancestor

```python
def lowest_common_ancestor(root, p, q):
    """
    Find lowest common ancestor of two nodes
    Time: O(n), Space: O(h)
    """
    if not root or root == p or root == q:
        return root
    
    left = lowest_common_ancestor(root.left, p, q)
    right = lowest_common_ancestor(root.right, p, q)
    
    if left and right:
        return root
    return left or right

# For BST (optimized)
def lca_bst(root, p, q):
    """
    LCA in BST (can use BST property)
    Time: O(h), Space: O(1)
    """
    while root:
        if p.val < root.val and q.val < root.val:
            root = root.left
        elif p.val > root.val and q.val > root.val:
            root = root.right
        else:
            return root
    return None
```

### 5.6 Symmetric Tree

```python
def is_symmetric(root):
    """
    Check if tree is symmetric (mirror of itself)
    Time: O(n), Space: O(h)
    """
    def is_mirror(left, right):
        if not left and not right:
            return True
        if not left or not right:
            return False
        return (left.val == right.val and
                is_mirror(left.left, right.right) and
                is_mirror(left.right, right.left))
    
    if not root:
        return True
    return is_mirror(root.left, root.right)
```

### 5.7 Same Tree

```python
def is_same_tree(p, q):
    """
    Check if two trees are identical
    Time: O(n), Space: O(h)
    """
    if not p and not q:
        return True
    if not p or not q:
        return False
    return (p.val == q.val and
            is_same_tree(p.left, q.left) and
            is_same_tree(p.right, q.right))
```

### 5.8 Invert Tree

```python
def invert_tree(root):
    """
    Invert binary tree (mirror)
    Time: O(n), Space: O(h)
    """
    if not root:
        return None
    
    # Swap children
    root.left, root.right = root.right, root.left
    
    # Recursively invert subtrees
    invert_tree(root.left)
    invert_tree(root.right)
    
    return root
```

---

## 6. Practice Problems

### Problem Set 1: Tree Traversals

1. **Binary Tree Right Side View**
```python
def right_side_view(root):
    """
    Return values of nodes visible from right side
    Time: O(n), Space: O(h)
    """
    result = []
    
    def dfs(node, depth):
        if not node:
            return
        
        if depth == len(result):
            result.append(node.val)
        
        dfs(node.right, depth + 1)
        dfs(node.left, depth + 1)
    
    dfs(root, 0)
    return result
```

2. **Binary Tree Level Order Traversal II**
```python
def level_order_bottom(root):
    """
    Level order from bottom to top
    Time: O(n), Space: O(n)
    """
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
    
    return result[::-1]
```

### Problem Set 2: Tree Construction

1. **Serialize and Deserialize Binary Tree**
```python
def serialize(root):
    """
    Serialize tree to string
    Time: O(n), Space: O(n)
    """
    if not root:
        return "None,"
    
    return (str(root.val) + "," +
            serialize(root.left) +
            serialize(root.right))

def deserialize(data):
    """
    Deserialize string to tree
    Time: O(n), Space: O(n)
    """
    values = data.split(',')
    index = [0]
    
    def build():
        if values[index[0]] == 'None':
            index[0] += 1
            return None
        
        node = TreeNode(int(values[index[0]]))
        index[0] += 1
        node.left = build()
        node.right = build()
        
        return node
    
    return build()
```

2. **Construct BST from Preorder**
```python
def bst_from_preorder(preorder):
    """
    Build BST from preorder traversal
    Time: O(n), Space: O(n)
    """
    index = [0]
    
    def build(min_val, max_val):
        if index[0] >= len(preorder):
            return None
        
        val = preorder[index[0]]
        if val < min_val or val > max_val:
            return None
        
        node = TreeNode(val)
        index[0] += 1
        
        node.left = build(min_val, val)
        node.right = build(val, max_val)
        
        return node
    
    return build(float('-inf'), float('inf'))
```

### Problem Set 3: Tree Properties

1. **Maximum Path Sum**
```python
def max_path_sum(root):
    """
    Maximum path sum (path can start and end anywhere)
    Time: O(n), Space: O(h)
    """
    max_sum = [float('-inf')]
    
    def max_gain(node):
        if not node:
            return 0
        
        # Max gain from left and right (can be negative)
        left_gain = max(max_gain(node.left), 0)
        right_gain = max(max_gain(node.right), 0)
        
        # Path through this node
        current_path = node.val + left_gain + right_gain
        max_sum[0] = max(max_sum[0], current_path)
        
        # Return max gain for parent
        return node.val + max(left_gain, right_gain)
    
    max_gain(root)
    return max_sum[0]
```

2. **Count Complete Tree Nodes**
```python
def count_nodes(root):
    """
    Count nodes in complete binary tree
    Time: O(log^2 n), Space: O(1)
    """
    if not root:
        return 0
    
    def left_depth(node):
        depth = 0
        while node:
            depth += 1
            node = node.left
        return depth
    
    def right_depth(node):
        depth = 0
        while node:
            depth += 1
            node = node.right
        return depth
    
    left_d = left_depth(root)
    right_d = right_depth(root)
    
    if left_d == right_d:
        # Perfect binary tree
        return 2**left_d - 1
    else:
        return 1 + count_nodes(root.left) + count_nodes(root.right)
```

---

## Summary of Part 3

### Key Concepts

1. **Tree Traversals**: Preorder, Inorder, Postorder, Level-order
2. **BST Operations**: Insert, Search, Delete, Validation
3. **Tree Construction**: From traversals, from arrays
4. **Tree Properties**: Depth, Diameter, Path Sum, LCA
5. **Advanced**: Morris traversal, Serialization

### Common Patterns

- **DFS Traversals**: Recursive and iterative
- **BFS (Level-order)**: Queue-based
- **Tree Construction**: Divide and conquer
- **Tree Properties**: Recursive calculations

### Next Steps

**Part 4** will cover:
- Graphs (Representation, Traversals)
- Shortest Path Algorithms
- Minimum Spanning Tree
- Topological Sort

---

**Practice tree problems on LeetCode before moving to Part 4!**

