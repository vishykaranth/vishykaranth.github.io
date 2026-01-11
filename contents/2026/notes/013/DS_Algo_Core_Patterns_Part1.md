# Core DS/Algo Patterns: Solve Almost Any Problem

## Part 1: Fundamental Patterns and Techniques

---

## Table of Contents

1. [Two Pointers Pattern](#1-two-pointers-pattern)
2. [Sliding Window Pattern](#2-sliding-window-pattern)
3. [Fast and Slow Pointers](#3-fast-and-slow-pointers)
4. [Prefix Sum Pattern](#4-prefix-sum-pattern)
5. [Hash Table Pattern](#5-hash-table-pattern)
6. [Stack Pattern](#6-stack-pattern)
7. [Master Pattern Recognition](#7-master-pattern-recognition)

---

## 1. Two Pointers Pattern

### 1.1 Core Logic

**When to Use**: Sorted arrays, pairs, palindromes, removing duplicates

**Pattern Template**:
```python
def two_pointers_template(arr):
    left, right = 0, len(arr) - 1
    
    while left < right:
        # Calculate current value
        current = calculate(arr[left], arr[right])
        
        if current == target:
            # Found solution
            return [left, right]
        elif current < target:
            # Need larger value
            left += 1
        else:
            # Need smaller value
            right -= 1
    
    return []  # No solution
```

### 1.2 Key Variations

**Variation 1: Opposite Ends (Sorted Array)**
```python
def two_sum_sorted(arr, target):
    """Two sum in sorted array"""
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

# Problems: Two Sum (sorted), 3Sum, 4Sum, Container With Water
```

**Variation 2: Same Direction (Fast and Slow)**
```python
def remove_duplicates(arr):
    """Remove duplicates in-place"""
    if not arr:
        return 0
    
    slow = 0
    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    
    return slow + 1

# Problems: Remove Duplicates, Move Zeros, Remove Element
```

**Variation 3: Three Pointers**
```python
def three_sum(arr, target):
    """Find triplets that sum to target"""
    arr.sort()
    result = []
    
    for i in range(len(arr) - 2):
        if i > 0 and arr[i] == arr[i - 1]:
            continue
        
        left, right = i + 1, len(arr) - 1
        while left < right:
            current_sum = arr[i] + arr[left] + arr[right]
            if current_sum == target:
                result.append([arr[i], arr[left], arr[right]])
                # Skip duplicates
                while left < right and arr[left] == arr[left + 1]:
                    left += 1
                while left < right and arr[right] == arr[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif current_sum < target:
                left += 1
            else:
                right -= 1
    
    return result
```

### 1.3 Pattern Recognition

**Use Two Pointers When**:
- Array is sorted (or can be sorted)
- Need to find pairs/triplets
- Need to compare elements from different positions
- Problem involves "opposite" or "symmetric" operations

**Common Problems**:
- Two Sum, 3Sum, 4Sum
- Container With Water
- Trapping Rain Water
- Valid Palindrome
- Remove Duplicates

---

## 2. Sliding Window Pattern

### 2.1 Core Logic

**When to Use**: Subarray/substring problems, "window" of elements, consecutive elements

**Pattern Template**:
```python
def sliding_window_template(s):
    left = 0
    window = {}  # or set, list, depending on problem
    
    for right in range(len(s)):
        # Expand window: add s[right]
        window[s[right]] = window.get(s[right], 0) + 1
        
        # Shrink window if condition met
        while condition_to_shrink(window):
            window[s[left]] -= 1
            if window[s[left]] == 0:
                del window[s[left]]
            left += 1
        
        # Update result
        result = max(result, right - left + 1)
    
    return result
```

### 2.2 Key Variations

**Variation 1: Fixed Window Size**
```python
def fixed_window(arr, k):
    """Maximum sum of subarray of size k"""
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# Problems: Maximum Average Subarray, Sliding Window Maximum
```

**Variation 2: Variable Window (Longest)**
```python
def longest_substring_no_repeat(s):
    """Longest substring without repeating characters"""
    char_map = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        # If character seen and in current window
        if s[right] in char_map and char_map[s[right]] >= left:
            left = char_map[s[right]] + 1
        
        char_map[s[right]] = right
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Problems: Longest Substring, Minimum Window Substring
```

**Variation 3: Variable Window (Shortest)**
```python
def min_window_substring(s, t):
    """Minimum window containing all characters of t"""
    need = {}
    for char in t:
        need[char] = need.get(char, 0) + 1
    
    left = 0
    valid = 0  # Number of unique characters satisfied
    window = {}
    min_len = float('inf')
    start = 0
    
    for right in range(len(s)):
        char = s[right]
        if char in need:
            window[char] = window.get(char, 0) + 1
            if window[char] == need[char]:
                valid += 1
        
        # Shrink window
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

**Variation 4: At Most K Pattern**
```python
def longest_substring_at_most_k(s, k):
    """Longest substring with at most k distinct characters"""
    char_count = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char_count[s[right]] = char_count.get(s[right], 0) + 1
        
        # Shrink if more than k distinct
        while len(char_count) > k:
            char_count[s[left]] -= 1
            if char_count[s[left]] == 0:
                del char_count[s[left]]
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

### 2.3 Pattern Recognition

**Use Sliding Window When**:
- Problem involves subarray/substring
- Need to find "longest" or "shortest" subarray
- Need to maintain some property in window
- Problem mentions "consecutive" or "contiguous"

**Common Problems**:
- Longest Substring Without Repeating Characters
- Minimum Window Substring
- Maximum Average Subarray
- Longest Repeating Character Replacement
- Permutation in String

---

## 3. Fast and Slow Pointers

### 3.1 Core Logic

**When to Use**: Linked lists, cycle detection, finding middle, palindrome checking

**Pattern Template**:
```python
def fast_slow_template(head):
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next      # Move 1 step
        fast = fast.next.next # Move 2 steps
        # Process or check condition
    
    return slow  # or result based on problem
```

### 3.2 Key Applications

**Application 1: Cycle Detection**
```python
def has_cycle(head):
    """Detect cycle in linked list"""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    
    return False

def find_cycle_start(head):
    """Find node where cycle begins"""
    slow = fast = head
    
    # Find meeting point
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    else:
        return None
    
    # Find cycle start
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

**Application 2: Find Middle**
```python
def find_middle(head):
    """Find middle node of linked list"""
    slow = fast = head
    
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    return slow
```

**Application 3: Palindrome Check**
```python
def is_palindrome_linked_list(head):
    """Check if linked list is palindrome"""
    # Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    
    # Reverse second half
    prev = None
    current = slow
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    # Compare
    first, second = head, prev
    while second:
        if first.val != second.val:
            return False
        first = first.next
        second = second.next
    
    return True
```

### 3.3 Pattern Recognition

**Use Fast/Slow When**:
- Linked list problems
- Need to find middle element
- Cycle detection
- Need to process in pairs

---

## 4. Prefix Sum Pattern

### 4.1 Core Logic

**When to Use**: Range sum queries, subarray sum problems, cumulative operations

**Pattern Template**:
```python
def prefix_sum_template(arr):
    # Build prefix sum
    prefix = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        prefix[i + 1] = prefix[i] + arr[i]
    
    # Query range [left, right]
    def range_sum(left, right):
        return prefix[right + 1] - prefix[left]
    
    return prefix
```

### 4.2 Key Applications

**Application 1: Subarray Sum Equals K**
```python
def subarray_sum_equals_k(arr, k):
    """Count subarrays with sum equal to k"""
    prefix_sum = 0
    count = 0
    sum_count = {0: 1}  # prefix_sum 0 appears once
    
    for num in arr:
        prefix_sum += num
        # If prefix_sum - k exists, we found a subarray
        if prefix_sum - k in sum_count:
            count += sum_count[prefix_sum - k]
        sum_count[prefix_sum] = sum_count.get(prefix_sum, 0) + 1
    
    return count
```

**Application 2: 2D Prefix Sum**
```python
def prefix_sum_2d(matrix):
    """2D prefix sum for range queries"""
    rows, cols = len(matrix), len(matrix[0])
    prefix = [[0] * (cols + 1) for _ in range(rows + 1)]
    
    for i in range(rows):
        for j in range(cols):
            prefix[i + 1][j + 1] = (
                matrix[i][j] +
                prefix[i][j + 1] +
                prefix[i + 1][j] -
                prefix[i][j]
            )
    
    def range_sum(row1, col1, row2, col2):
        return (
            prefix[row2 + 1][col2 + 1] -
            prefix[row1][col2 + 1] -
            prefix[row2 + 1][col1] +
            prefix[row1][col1]
        )
    
    return prefix
```

### 4.3 Pattern Recognition

**Use Prefix Sum When**:
- Range sum queries
- Subarray sum problems
- Cumulative operations
- Need O(1) range queries

---

## 5. Hash Table Pattern

### 5.1 Core Logic

**When to Use**: Fast lookups, counting, grouping, avoiding duplicates

**Pattern Template**:
```python
def hash_table_template(arr):
    seen = {}  # or set, Counter, defaultdict
    
    for item in arr:
        # Check if seen
        if item in seen:
            # Process
            pass
        else:
            # First occurrence
            seen[item] = value
    
    return result
```

### 5.2 Key Applications

**Application 1: Two Sum**
```python
def two_sum(arr, target):
    """Find two numbers that sum to target"""
    seen = {}
    for i, num in enumerate(arr):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

**Application 2: Grouping**
```python
def group_anagrams(strs):
    """Group strings that are anagrams"""
    groups = defaultdict(list)
    
    for s in strs:
        key = ''.join(sorted(s))  # or tuple(sorted(s))
        groups[key].append(s)
    
    return list(groups.values())
```

**Application 3: Frequency Counting**
```python
from collections import Counter

def frequency_analysis(arr):
    """Analyze frequency of elements"""
    counter = Counter(arr)
    
    # Most common
    most_common = counter.most_common(3)
    
    # Filter by frequency
    frequent = [item for item, count in counter.items() if count > threshold]
    
    return most_common, frequent
```

### 5.3 Pattern Recognition

**Use Hash Table When**:
- Need O(1) lookup
- Counting frequencies
- Grouping elements
- Avoiding duplicates
- Two sum / complement problems

---

## 6. Stack Pattern

### 6.1 Core Logic

**When to Use**: Matching pairs, nested structures, next greater/smaller, monotonic problems

**Pattern Template**:
```python
def stack_template(arr):
    stack = []
    result = []
    
    for i, num in enumerate(arr):
        # Process with stack
        while stack and condition(stack[-1], num):
            # Pop and process
            element = stack.pop()
            process(element, num)
        
        stack.append(num)
    
    return result
```

### 6.2 Key Applications

**Application 1: Next Greater Element**
```python
def next_greater_element(nums):
    """Find next greater element for each element"""
    stack = []
    result = [-1] * len(nums)
    
    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            result[stack.pop()] = num
        stack.append(i)
    
    return result
```

**Application 2: Valid Parentheses**
```python
def is_valid_parentheses(s):
    """Check if parentheses are valid"""
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

**Application 3: Monotonic Stack**
```python
def largest_rectangle_histogram(heights):
    """Largest rectangle in histogram"""
    stack = []
    max_area = 0
    
    for i, height in enumerate(heights):
        while stack and heights[stack[-1]] > height:
            h = heights[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, h * width)
        stack.append(i)
    
    # Process remaining
    while stack:
        h = heights[stack.pop()]
        width = len(heights) if not stack else len(heights) - stack[-1] - 1
        max_area = max(max_area, h * width)
    
    return max_area
```

### 6.3 Pattern Recognition

**Use Stack When**:
- Matching pairs (parentheses, brackets)
- Next greater/smaller element
- Monotonic problems
- Nested structures
- Reversing order

---

## 7. Master Pattern Recognition

### 7.1 Decision Tree

```python
"""
Problem Analysis Decision Tree:

1. Is it a subarray/substring problem?
   YES → Sliding Window
   
2. Is the array sorted (or can be sorted)?
   YES → Two Pointers (opposite ends)
   
3. Is it a linked list problem?
   YES → Fast/Slow Pointers
   
4. Need range queries?
   YES → Prefix Sum
   
5. Need fast lookup?
   YES → Hash Table
   
6. Need matching pairs or nested structures?
   YES → Stack
   
7. Need to find optimal solution?
   YES → DP or Greedy
   
8. Need to try all possibilities?
   YES → Backtracking
   
9. Is it a tree/graph problem?
   YES → DFS/BFS
"""

def identify_pattern(problem_description):
    """
    Pattern identification logic
    """
    keywords = {
        'sliding_window': ['subarray', 'substring', 'window', 'consecutive', 'contiguous'],
        'two_pointers': ['sorted', 'pair', 'sum', 'palindrome'],
        'hash_table': ['frequency', 'count', 'group', 'duplicate', 'lookup'],
        'stack': ['parentheses', 'bracket', 'nested', 'next greater', 'monotonic'],
        'prefix_sum': ['range sum', 'subarray sum', 'cumulative'],
        'fast_slow': ['linked list', 'cycle', 'middle', 'palindrome']
    }
    
    # Analyze problem description and return pattern
    pass
```

### 7.2 Universal Problem-Solving Template

```python
def solve_any_problem(input_data):
    """
    Universal problem-solving template
    """
    # Step 1: Understand constraints
    n = len(input_data)
    # Expected complexity: O(n), O(n log n), O(n^2)?
    
    # Step 2: Identify pattern
    # - Look for keywords
    # - Analyze problem type
    # - Consider data structure
    
    # Step 3: Choose approach
    # - Two Pointers?
    # - Sliding Window?
    # - Hash Table?
    # - Stack?
    # - DP?
    # - Backtracking?
    
    # Step 4: Implement
    result = []
    # ... implementation
    
    # Step 5: Handle edge cases
    if not input_data:
        return []
    
    return result
```

### 7.3 Pattern Combinations

**Combination 1: Hash Table + Two Pointers**
```python
def three_sum_with_hash(arr, target):
    """3Sum using hash table"""
    arr.sort()
    result = []
    
    for i in range(len(arr) - 2):
        if i > 0 and arr[i] == arr[i - 1]:
            continue
        
        seen = set()
        for j in range(i + 1, len(arr)):
            complement = target - arr[i] - arr[j]
            if complement in seen:
                result.append([arr[i], complement, arr[j]])
            seen.add(arr[j])
    
    return result
```

**Combination 2: Stack + Hash Table**
```python
def next_greater_element_ii(nums):
    """Next greater element in circular array"""
    n = len(nums)
    result = [-1] * n
    stack = []
    
    # Process circular array (2 * n)
    for i in range(2 * n):
        while stack and nums[stack[-1]] < nums[i % n]:
            result[stack.pop()] = nums[i % n]
        if i < n:
            stack.append(i)
    
    return result
```

---

## Summary: Core Patterns

### Pattern Quick Reference

1. **Two Pointers**: Sorted arrays, pairs, palindromes
2. **Sliding Window**: Subarray/substring, consecutive elements
3. **Fast/Slow**: Linked lists, cycles, middle element
4. **Prefix Sum**: Range queries, subarray sums
5. **Hash Table**: Fast lookup, counting, grouping
6. **Stack**: Matching pairs, monotonic, nested structures

### When to Use Each Pattern

```
Problem Type → Pattern:
- Subarray/Substring → Sliding Window
- Sorted Array + Pairs → Two Pointers
- Linked List → Fast/Slow Pointers
- Range Queries → Prefix Sum
- Fast Lookup → Hash Table
- Matching/Nested → Stack
```

### Next Steps

**Part 2** will cover:
- Tree Patterns (DFS, BFS, Tree DP)
- Graph Patterns (Traversals, Shortest Path)
- Recursion Patterns

---

**Master these 6 core patterns to solve 60% of competitive programming problems!**

