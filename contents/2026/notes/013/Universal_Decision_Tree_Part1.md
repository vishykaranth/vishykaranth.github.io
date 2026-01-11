# Universal Decision Tree: Solve Any DS/Algo Problem

## Part 1: Foundation - Arrays, Strings, and Basic Data Structures

---

## Table of Contents

1. [Decision Tree Fundamentals](#1-decision-tree-fundamentals)
2. [Array Problem Decision Tree](#2-array-problem-decision-tree)
3. [String Problem Decision Tree](#3-string-problem-decision-tree)
4. [Linked List Decision Tree](#4-linked-list-decision-tree)
5. [Stack and Queue Decision Tree](#5-stack-and-queue-decision-tree)
6. [Hash Table Decision Tree](#6-hash-table-decision-tree)
7. [Complete Flowchart](#7-complete-flowchart)

---

## 1. Decision Tree Fundamentals

### 1.1 What is a Decision Tree?

A **Decision Tree** is a systematic approach to identify the correct algorithm/pattern for any problem by asking a series of questions that narrow down the solution space.

**Key Principles**:
1. **Start Broad**: Identify problem category
2. **Narrow Down**: Ask specific questions
3. **Eliminate Options**: Rule out inappropriate patterns
4. **Select Pattern**: Choose the best approach
5. **Verify**: Ensure pattern fits constraints

### 1.2 Decision Tree Structure

```python
"""
Decision Tree Structure:

Level 1: Problem Category
  ├─ Arrays/Strings
  ├─ Linked Lists
  ├─ Trees
  ├─ Graphs
  ├─ Dynamic Programming
  └─ Backtracking/Greedy

Level 2: Problem Characteristics
  ├─ Subarray/Substring?
  ├─ Sorted?
  ├─ Need all solutions?
  └─ Optimization problem?

Level 3: Pattern Selection
  └─ Specific algorithm/pattern
"""
```

### 1.3 How to Use This Guide

**Step-by-Step Process**:
1. Read the problem statement
2. Identify keywords and characteristics
3. Follow the decision tree
4. Select the appropriate pattern
5. Apply the template
6. Verify solution

---

## 2. Array Problem Decision Tree

### 2.1 Complete Array Decision Tree

```python
"""
ARRAY PROBLEM DECISION TREE

Question 1: Is it a subarray/substring problem?
│
├─ YES → Go to Subarray Decision Tree (2.2)
│
└─ NO → Continue to Question 2

Question 2: Is the array sorted (or can be sorted)?
│
├─ YES → Go to Sorted Array Decision Tree (2.3)
│
└─ NO → Continue to Question 3

Question 3: Need to find pairs/triplets?
│
├─ YES → Hash Table (if unsorted) or Two Pointers (if sorted)
│
└─ NO → Continue to Question 4

Question 4: Need range queries?
│
├─ YES → Prefix Sum or Segment Tree
│
└─ NO → Continue to Question 5

Question 5: Need fast lookup?
│
├─ YES → Hash Table
│
└─ NO → Continue to Question 6

Question 6: Need to process in order?
│
├─ YES → Stack or Queue
│
└─ NO → Linear scan or other pattern
"""
```

### 2.2 Subarray Decision Tree

```python
"""
SUBARRAY/SUBSTRING DECISION TREE

Question 1: What is the window size?
│
├─ Fixed Size → Sliding Window (Fixed)
│   │
│   └─ Examples: Maximum Average Subarray, Sliding Window Maximum
│
└─ Variable Size → Continue to Question 2

Question 2: What property must the window have?
│
├─ Exact match (e.g., sum = k) → Sliding Window + Hash Table
│   │
│   └─ Examples: Subarray Sum Equals K, Minimum Window Substring
│
├─ At most K (e.g., at most k distinct) → Sliding Window (Expand/Shrink)
│   │
│   └─ Examples: Longest Substring with At Most K Distinct Characters
│
└─ Longest/Shortest with property → Sliding Window (Variable)
    │
    └─ Examples: Longest Substring Without Repeating Characters
"""
```

**Detailed Examples**:

**Example 1: Fixed Window**
```python
def identify_pattern_fixed_window(problem):
    """
    Identify: "subarray of size k", "window of length k"
    Pattern: Fixed Sliding Window
    """
    keywords = ['subarray of size', 'window of length', 'k consecutive']
    if any(kw in problem.lower() for kw in keywords):
        return "Fixed Sliding Window"
    return None

# Template
def fixed_window_template(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum
```

**Example 2: Variable Window - Longest**
```python
def identify_pattern_longest_substring(problem):
    """
    Identify: "longest substring", "maximum length"
    Pattern: Variable Sliding Window (Expand/Shrink)
    """
    keywords = ['longest', 'maximum length', 'longest substring']
    if any(kw in problem.lower() for kw in keywords):
        return "Variable Sliding Window (Longest)"
    return None

# Template
def longest_substring_template(s):
    char_map = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char_map[s[right]] = char_map.get(s[right], 0) + 1
        
        # Shrink window if condition violated
        while condition_violated(char_map):
            char_map[s[left]] -= 1
            if char_map[s[left]] == 0:
                del char_map[s[left]]
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

**Example 3: Variable Window - Shortest**
```python
def identify_pattern_shortest_substring(problem):
    """
    Identify: "minimum window", "shortest substring"
    Pattern: Variable Sliding Window (Expand/Shrink for minimum)
    """
    keywords = ['minimum window', 'shortest substring', 'smallest']
    if any(kw in problem.lower() for kw in keywords):
        return "Variable Sliding Window (Shortest)"
    return None

# Template
def shortest_substring_template(s, t):
    need = {}
    for char in t:
        need[char] = need.get(char, 0) + 1
    
    window = {}
    left = 0
    valid = 0
    min_len = float('inf')
    start = 0
    
    for right in range(len(s)):
        # Expand window
        char = s[right]
        if char in need:
            window[char] = window.get(char, 0) + 1
            if window[char] == need[char]:
                valid += 1
        
        # Shrink window when valid
        while valid == len(need):
            if right - left + 1 < min_len:
                min_len = right - left + 1
                start = left
            
            left_char = s[left]
            if left_char in need:
                if window[left_char] == need[left_char]:
                    valid -= 1
                window[left_char] -= 1
            left += 1
    
    return "" if min_len == float('inf') else s[start:start + min_len]
```

### 2.3 Sorted Array Decision Tree

```python
"""
SORTED ARRAY DECISION TREE

Question 1: Need to find specific element?
│
├─ YES → Binary Search
│   │
│   └─ Examples: Search in Rotated Array, Find Peak Element
│
└─ NO → Continue to Question 2

Question 2: Need to find pairs/triplets?
│
├─ YES → Two Pointers (Opposite Ends)
│   │
│   ├─ Two Sum → Two Pointers
│   ├─ Three Sum → Two Pointers (nested)
│   └─ Four Sum → Two Pointers (nested)
│
└─ NO → Continue to Question 3

Question 3: Need to merge/combine?
│
├─ YES → Two Pointers (Merge Pattern)
│   │
│   └─ Examples: Merge Sorted Arrays, Intersection of Arrays
│
└─ NO → Other sorted array operations
"""
```

**Detailed Examples**:

**Example 1: Binary Search**
```python
def identify_binary_search(problem):
    """
    Identify: "search", "find", "sorted array", "O(log n)"
    Pattern: Binary Search
    """
    keywords = ['search', 'find', 'sorted', 'log n', 'binary search']
    if any(kw in problem.lower() for kw in keywords):
        return "Binary Search"
    return None

# Template
def binary_search_template(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    
    return -1
```

**Example 2: Two Pointers on Sorted Array**
```python
def identify_two_pointers_sorted(problem):
    """
    Identify: "sorted array", "two sum", "three sum", "pairs"
    Pattern: Two Pointers (Opposite Ends)
    """
    keywords = ['sorted', 'two sum', 'three sum', 'pairs', 'triplets']
    if any(kw in problem.lower() for kw in keywords):
        return "Two Pointers (Sorted Array)"
    return None

# Template
def two_pointers_sorted_template(arr, target):
    left, right = 0, len(arr) - 1
    
    while left < right:
        current_sum = arr[left] + arr[right]
        
        if current_sum == target:
            return [left, right]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    
    return []
```

### 2.4 Unsorted Array Decision Tree

```python
"""
UNSORTED ARRAY DECISION TREE

Question 1: Need to find pairs/triplets?
│
├─ YES → Hash Table
│   │
│   └─ Examples: Two Sum, Three Sum (can use hash table)
│
└─ NO → Continue to Question 2

Question 2: Need to count frequencies?
│
├─ YES → Hash Table (Counter)
│   │
│   └─ Examples: Group Anagrams, Frequency Sort
│
└─ NO → Continue to Question 3

Question 3: Need to remove duplicates?
│
├─ YES → Hash Table (Set) or Two Pointers (if can sort)
│   │
│   └─ Examples: Remove Duplicates, Contains Duplicate
│
└─ NO → Other patterns
"""
```

---

## 3. String Problem Decision Tree

### 3.1 Complete String Decision Tree

```python
"""
STRING PROBLEM DECISION TREE

Question 1: Is it a substring/subsequence problem?
│
├─ YES → Go to Substring Decision Tree (3.2)
│
└─ NO → Continue to Question 2

Question 2: Is it a pattern matching problem?
│
├─ YES → Go to Pattern Matching Decision Tree (3.3)
│
└─ NO → Continue to Question 3

Question 3: Is it a palindrome problem?
│
├─ YES → Go to Palindrome Decision Tree (3.4)
│
└─ NO → Continue to Question 4

Question 4: Is it an anagram problem?
│
├─ YES → Hash Table (Counter) or Sorting
│   │
│   └─ Examples: Group Anagrams, Valid Anagram
│
└─ NO → Continue to Question 5

Question 5: Is it a parsing/validation problem?
│
├─ YES → Stack
│   │
│   └─ Examples: Valid Parentheses, Decode String
│
└─ NO → Other string operations
"""
```

### 3.2 Substring Decision Tree

```python
"""
SUBSTRING DECISION TREE

Question 1: What property must substring have?
│
├─ No repeating characters → Sliding Window
│   │
│   └─ Examples: Longest Substring Without Repeating Characters
│
├─ Contains all characters of another string → Sliding Window + Hash Table
│   │
│   └─ Examples: Minimum Window Substring
│
├─ At most K distinct characters → Sliding Window
│   │
│   └─ Examples: Longest Substring with At Most K Distinct Characters
│
└─ Other properties → Analyze specific requirement
"""
```

### 3.3 Pattern Matching Decision Tree

```python
"""
PATTERN MATCHING DECISION TREE

Question 1: What is the pattern complexity?
│
├─ Simple pattern (no wildcards) → KMP or Rabin-Karp
│   │
│   └─ Examples: strStr(), Repeated Substring Pattern
│
├─ Pattern with wildcards (., *) → Dynamic Programming
│   │
│   └─ Examples: Regular Expression Matching, Wildcard Matching
│
└─ Multiple patterns → Trie + Pattern Matching
    │
    └─ Examples: Word Search II, Add and Search Words
"""
```

**Example: Pattern Matching Identification**
```python
def identify_pattern_matching(problem):
    """
    Identify pattern matching problems
    """
    if 'wildcard' in problem.lower() or 'regular expression' in problem.lower():
        return "DP Pattern Matching"
    elif 'find pattern' in problem.lower() or 'strstr' in problem.lower():
        return "KMP or Rabin-Karp"
    elif 'multiple patterns' in problem.lower() or 'word search' in problem.lower():
        return "Trie + Pattern Matching"
    return None
```

### 3.4 Palindrome Decision Tree

```python
"""
PALINDROME DECISION TREE

Question 1: Check if palindrome?
│
├─ YES → Two Pointers (from ends)
│   │
│   └─ Examples: Valid Palindrome, Palindrome Linked List
│
└─ NO → Continue to Question 2

Question 2: Find longest palindromic substring?
│
├─ YES → Manacher's Algorithm or DP
│   │
│   └─ Examples: Longest Palindromic Substring
│
└─ NO → Continue to Question 3

Question 3: Count palindromic substrings?
│
├─ YES → DP or Expand Around Centers
│   │
│   └─ Examples: Palindromic Substrings
│
└─ NO → Other palindrome operations
"""
```

---

## 4. Linked List Decision Tree

### 4.1 Complete Linked List Decision Tree

```python
"""
LINKED LIST DECISION TREE

Question 1: Need to detect cycle?
│
├─ YES → Fast/Slow Pointers (Floyd's Algorithm)
│   │
│   └─ Examples: Linked List Cycle, Find Cycle Start
│
└─ NO → Continue to Question 2

Question 2: Need to find middle?
│
├─ YES → Fast/Slow Pointers
│   │
│   └─ Examples: Middle of Linked List
│
└─ NO → Continue to Question 3

Question 3: Need to reverse?
│
├─ YES → Iterative or Recursive Reversal
│   │
│   └─ Examples: Reverse Linked List, Reverse Nodes in k-Group
│
└─ NO → Continue to Question 4

Question 4: Need to merge?
│
├─ YES → Two Pointers (Merge Pattern)
│   │
│   └─ Examples: Merge Two Sorted Lists, Merge k Sorted Lists
│
└─ NO → Continue to Question 5

Question 5: Need to remove nodes?
│
├─ YES → Two Pointers (Fast/Slow) or Dummy Node
│   │
│   └─ Examples: Remove Nth Node From End, Remove Duplicates
│
└─ NO → Other linked list operations
"""
```

**Detailed Examples**:

**Example 1: Cycle Detection**
```python
def identify_linked_list_cycle(problem):
    """
    Identify: "cycle", "loop", "detect"
    Pattern: Fast/Slow Pointers
    """
    keywords = ['cycle', 'loop', 'detect', 'circular']
    if any(kw in problem.lower() for kw in keywords):
        return "Fast/Slow Pointers (Cycle Detection)"
    return None

# Template
def detect_cycle(head):
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False
```

**Example 2: Reverse Linked List**
```python
def identify_reverse_linked_list(problem):
    """
    Identify: "reverse", "flip"
    Pattern: Iterative or Recursive Reversal
    """
    keywords = ['reverse', 'flip', 'opposite order']
    if any(kw in problem.lower() for kw in keywords):
        return "Reverse Linked List"
    return None

# Template
def reverse_linked_list(head):
    prev = None
    current = head
    
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    return prev
```

---

## 5. Stack and Queue Decision Tree

### 5.1 Stack Decision Tree

```python
"""
STACK DECISION TREE

Question 1: Is it a matching pairs problem?
│
├─ YES → Stack
│   │
│   └─ Examples: Valid Parentheses, Valid Parentheses String
│
└─ NO → Continue to Question 2

Question 2: Is it a next greater/smaller element problem?
│
├─ YES → Monotonic Stack
│   │
│   └─ Examples: Next Greater Element, Daily Temperatures
│
└─ NO → Continue to Question 3

Question 3: Is it a nested structure problem?
│
├─ YES → Stack
│   │
│   └─ Examples: Decode String, Basic Calculator
│
└─ NO → Continue to Question 4

Question 4: Is it a histogram/rectangle problem?
│
├─ YES → Monotonic Stack
│   │
│   └─ Examples: Largest Rectangle in Histogram
│
└─ NO → Other stack applications
"""
```

**Example: Stack Pattern Identification**
```python
def identify_stack_pattern(problem):
    """
    Identify stack problems
    """
    if any(kw in problem.lower() for kw in ['parentheses', 'bracket', 'matching']):
        return "Stack (Matching Pairs)"
    elif any(kw in problem.lower() for kw in ['next greater', 'next smaller', 'monotonic']):
        return "Monotonic Stack"
    elif any(kw in problem.lower() for kw in ['nested', 'decode', 'calculator']):
        return "Stack (Nested Structures)"
    elif 'histogram' in problem.lower() or 'rectangle' in problem.lower():
        return "Monotonic Stack (Histogram)"
    return None
```

### 5.2 Queue Decision Tree

```python
"""
QUEUE DECISION TREE

Question 1: Is it a level-order/BFS problem?
│
├─ YES → Queue (BFS)
│   │
│   └─ Examples: Level Order Traversal, BFS
│
└─ NO → Continue to Question 2

Question 2: Is it a sliding window maximum problem?
│
├─ YES → Deque (Monotonic Queue)
│   │
│   └─ Examples: Sliding Window Maximum
│
└─ NO → Other queue applications
"""
```

---

## 6. Hash Table Decision Tree

### 6.1 Complete Hash Table Decision Tree

```python
"""
HASH TABLE DECISION TREE

Question 1: Need fast lookup?
│
├─ YES → Hash Table
│   │
│   └─ Examples: Two Sum, Contains Duplicate
│
└─ NO → Continue to Question 2

Question 2: Need to count frequencies?
│
├─ YES → Hash Table (Counter)
│   │
│   └─ Examples: Group Anagrams, Top K Frequent Elements
│
└─ NO → Continue to Question 3

Question 3: Need to group elements?
│
├─ YES → Hash Table (Dictionary of Lists)
│   │
│   └─ Examples: Group Anagrams, Group Shifted Strings
│
└─ NO → Continue to Question 4

Question 4: Need to track seen elements?
│
├─ YES → Hash Table (Set)
│   │
│   └─ Examples: Contains Duplicate, Longest Consecutive Sequence
│
└─ NO → Other hash table applications
"""
```

**Example: Hash Table Pattern Identification**
```python
def identify_hash_table_pattern(problem):
    """
    Identify hash table problems
    """
    if any(kw in problem.lower() for kw in ['two sum', 'complement', 'lookup']):
        return "Hash Table (Lookup)"
    elif any(kw in problem.lower() for kw in ['frequency', 'count', 'occurrence']):
        return "Hash Table (Counter)"
    elif any(kw in problem.lower() for kw in ['group', 'anagram', 'categorize']):
        return "Hash Table (Grouping)"
    elif any(kw in problem.lower() for kw in ['duplicate', 'seen', 'unique']):
        return "Hash Table (Set)"
    return None
```

---

## 7. Complete Flowchart

### 7.1 Visual Decision Flow

```python
"""
COMPLETE ARRAY/STRING/LINKED LIST DECISION FLOW

START
  │
  ├─ Is it a subarray/substring problem?
  │   │
  │   ├─ YES → Fixed window size?
  │   │   │
  │   │   ├─ YES → Fixed Sliding Window
  │   │   └─ NO → Variable Sliding Window
  │   │
  │   └─ NO → Continue
  │
  ├─ Is array/string sorted?
  │   │
  │   ├─ YES → Need pairs/triplets?
  │   │   │
  │   │   ├─ YES → Two Pointers (Opposite Ends)
  │   │   └─ NO → Binary Search
  │   │
  │   └─ NO → Continue
  │
  ├─ Is it a linked list problem?
  │   │
  │   ├─ YES → Cycle detection?
  │   │   │
  │   │   ├─ YES → Fast/Slow Pointers
  │   │   └─ NO → Reverse/Merge/Remove → Appropriate pattern
  │   │
  │   └─ NO → Continue
  │
  ├─ Need matching pairs?
  │   │
  │   ├─ YES → Stack
  │   └─ NO → Continue
  │
  ├─ Need fast lookup?
  │   │
  │   ├─ YES → Hash Table
  │   └─ NO → Continue
  │
  └─ Other patterns (see Part 2 and 3)
"""
```

### 7.2 Quick Reference Matrix

| Problem Characteristic | Pattern | Example |
|------------------------|---------|---------|
| Subarray of size k | Fixed Sliding Window | Maximum Average Subarray |
| Longest substring | Variable Sliding Window | Longest Substring Without Repeating |
| Sorted array + pairs | Two Pointers | Two Sum (sorted) |
| Unsorted + pairs | Hash Table | Two Sum |
| Cycle detection | Fast/Slow Pointers | Linked List Cycle |
| Matching pairs | Stack | Valid Parentheses |
| Next greater | Monotonic Stack | Next Greater Element |
| Frequency count | Hash Table (Counter) | Top K Frequent |
| Group elements | Hash Table (Dict) | Group Anagrams |
| Range queries | Prefix Sum | Subarray Sum Equals K |

---

## Summary: Part 1 Decision Tree

### Key Decision Points

1. **Subarray/Substring** → Sliding Window
2. **Sorted Array** → Two Pointers or Binary Search
3. **Linked List** → Fast/Slow Pointers or Reversal
4. **Matching Pairs** → Stack
5. **Fast Lookup** → Hash Table
6. **Frequency/Grouping** → Hash Table

### Pattern Selection Rules

- **Subarray problems** → Always consider Sliding Window first
- **Sorted arrays** → Two Pointers or Binary Search
- **Linked lists** → Fast/Slow Pointers for cycles/middle
- **Parentheses/brackets** → Stack
- **Lookup problems** → Hash Table

### Next Steps

**Part 2** will cover:
- Tree Problem Decision Tree
- Graph Problem Decision Tree
- Advanced Array Patterns
- Multi-pattern Problems

---

**Master this foundation to identify patterns for 60% of competitive programming problems!**

