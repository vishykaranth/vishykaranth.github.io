# Data Structures & Algorithms for Competitive Programming

## Part 1: Fundamentals, Time/Space Complexity, and Basic Data Structures

---

## Table of Contents

1. [Introduction to Competitive Programming](#1-introduction-to-competitive-programming)
2. [Time and Space Complexity](#2-time-and-space-complexity)
3. [Python Basics for Competitive Programming](#3-python-basics-for-competitive-programming)
4. [Arrays and Lists](#4-arrays-and-lists)
5. [Two Pointers Technique](#5-two-pointers-technique)
6. [Sliding Window Technique](#6-sliding-window-technique)
7. [Prefix Sums](#7-prefix-sums)
8. [Practice Problems](#8-practice-problems)

---

## 1. Introduction to Competitive Programming

### 1.1 What is Competitive Programming?

Competitive programming involves solving algorithmic problems under time constraints, typically on platforms like:
- **LeetCode**: Interview preparation, daily challenges
- **Codeforces**: Competitive contests, rating system
- **AtCoder**: Japanese platform, educational
- **HackerRank**: Skill assessment, challenges
- **CodeChef**: Monthly contests, long challenges

### 1.2 Problem-Solving Approach

**Step-by-Step Process**:
1. **Understand the Problem**: Read carefully, identify constraints
2. **Identify Patterns**: Recognize common DS/Algo patterns
3. **Design Algorithm**: Think about approach before coding
4. **Analyze Complexity**: Ensure solution fits constraints
5. **Code**: Implement cleanly and efficiently
6. **Test**: Test with edge cases
7. **Optimize**: Improve if needed

### 1.3 Common Constraints

```
Typical Constraints:
- n ≤ 10^6: O(n) or O(n log n) solutions
- n ≤ 10^5: O(n log n) solutions
- n ≤ 10^4: O(n^2) might work
- n ≤ 10^3: O(n^2) or O(n^2 log n)
- n ≤ 100: O(n^3) might work
- n ≤ 20: O(2^n) or backtracking
```

---

## 2. Time and Space Complexity

### 2.1 Big O Notation

**Definition**: Describes the upper bound of algorithm complexity.

**Common Complexities**:
```python
# O(1) - Constant time
def constant_time(arr):
    return arr[0]  # Always takes same time

# O(log n) - Logarithmic time
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1

# O(n) - Linear time
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1

# O(n log n) - Linearithmic time
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

# O(n^2) - Quadratic time
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):
        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
    return arr

# O(2^n) - Exponential time
def fibonacci_naive(n):
    if n <= 1:
        return n
    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2)
```

### 2.2 Space Complexity

**Definition**: Amount of memory used by algorithm.

```python
# O(1) - Constant space
def constant_space(arr):
    total = 0
    for num in arr:
        total += num
    return total

# O(n) - Linear space
def linear_space(arr):
    result = []
    for num in arr:
        result.append(num * 2)
    return result

# O(n) - Recursive call stack
def recursive_sum(arr, index=0):
    if index == len(arr):
        return 0
    return arr[index] + recursive_sum(arr, index + 1)
```

### 2.3 Complexity Analysis Rules

1. **Drop constants**: O(2n) → O(n)
2. **Drop lower order terms**: O(n^2 + n) → O(n^2)
3. **Different terms for inputs**: O(a + b) or O(a * b)
4. **Nested loops multiply**: O(n * m)
5. **Recursive calls**: Usually O(branches^depth)

---

## 3. Python Basics for Competitive Programming

### 3.1 Fast I/O

```python
import sys

# Fast input
def input():
    return sys.stdin.readline().rstrip()

# For reading integers
n = int(input())
arr = list(map(int, input().split()))

# For reading multiple lines
n = int(input())
lines = [input() for _ in range(n)]
```

### 3.2 Useful Built-in Functions

```python
# Collections
from collections import defaultdict, deque, Counter

# Math
import math
math.gcd(a, b)
math.lcm(a, b)
math.factorial(n)
math.comb(n, k)  # Python 3.8+

# Heap
import heapq
heapq.heapify(arr)
heapq.heappush(heap, item)
heapq.heappop(heap)

# Bisect (Binary search)
import bisect
bisect.bisect_left(arr, x)  # Leftmost insertion point
bisect.bisect_right(arr, x)  # Rightmost insertion point
bisect.insort_left(arr, x)  # Insert maintaining sorted order
```

### 3.3 Common Patterns

```python
# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]

# Dictionary comprehension
square_dict = {x: x**2 for x in range(10)}

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union = set1 | set2
intersection = set1 & set2
difference = set1 - set2

# String operations
s = "hello"
s[::-1]  # Reverse
s.upper(), s.lower()
s.startswith("he"), s.endswith("lo")
s.split(), s.join(["a", "b"])
```

---

## 4. Arrays and Lists

### 4.1 Basic Operations

```python
# Creating arrays
arr = [1, 2, 3, 4, 5]
arr = [0] * n  # Array of zeros
arr = [[0] * m for _ in range(n)]  # 2D array

# Accessing
arr[0]  # First element
arr[-1]  # Last element
arr[1:4]  # Slicing [1, 2, 3]

# Modifying
arr.append(6)  # O(1)
arr.insert(0, 0)  # O(n)
arr.pop()  # O(1)
arr.pop(0)  # O(n)
arr.remove(3)  # O(n)
```

### 4.2 Common Array Problems

**Problem 1: Maximum Subarray (Kadane's Algorithm)**
```python
def max_subarray(arr):
    """
    Find maximum sum of contiguous subarray
    Time: O(n), Space: O(1)
    """
    max_sum = current_sum = arr[0]
    
    for num in arr[1:]:
        current_sum = max(num, current_sum + num)
        max_sum = max(max_sum, current_sum)
    
    return max_sum

# Example
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
print(max_subarray(arr))  # Output: 6 (subarray [4, -1, 2, 1])
```

**Problem 2: Two Sum**
```python
def two_sum(arr, target):
    """
    Find two numbers that add up to target
    Return indices
    Time: O(n), Space: O(n)
    """
    seen = {}
    for i, num in enumerate(arr):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

# Example
arr = [2, 7, 11, 15]
target = 9
print(two_sum(arr, target))  # Output: [0, 1]
```

**Problem 3: Product of Array Except Self**
```python
def product_except_self(arr):
    """
    Return array where result[i] = product of all elements except arr[i]
    Time: O(n), Space: O(1) excluding output array
    """
    n = len(arr)
    result = [1] * n
    
    # Left pass: result[i] = product of all elements to the left
    for i in range(1, n):
        result[i] = result[i - 1] * arr[i - 1]
    
    # Right pass: multiply by product of all elements to the right
    right_product = 1
    for i in range(n - 1, -1, -1):
        result[i] *= right_product
        right_product *= arr[i]
    
    return result

# Example
arr = [1, 2, 3, 4]
print(product_except_self(arr))  # Output: [24, 12, 8, 6]
```

**Problem 4: Rotate Array**
```python
def rotate_array(arr, k):
    """
    Rotate array to the right by k steps
    Time: O(n), Space: O(1)
    """
    n = len(arr)
    k %= n  # Handle k > n
    
    def reverse(start, end):
        while start < end:
            arr[start], arr[end] = arr[end], arr[start]
            start += 1
            end -= 1
    
    # Reverse entire array
    reverse(0, n - 1)
    # Reverse first k elements
    reverse(0, k - 1)
    # Reverse remaining elements
    reverse(k, n - 1)
    
    return arr

# Example
arr = [1, 2, 3, 4, 5, 6, 7]
k = 3
print(rotate_array(arr, k))  # Output: [5, 6, 7, 1, 2, 3, 4]
```

---

## 5. Two Pointers Technique

### 5.1 Concept

Use two pointers to traverse array, often reducing O(n^2) to O(n).

### 5.2 Common Patterns

**Pattern 1: Opposite Ends**
```python
def two_sum_sorted(arr, target):
    """
    Two sum in sorted array
    Time: O(n), Space: O(1)
    """
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

# Example
arr = [2, 7, 11, 15]
target = 9
print(two_sum_sorted(arr, target))  # Output: [0, 1]
```

**Pattern 2: Same Direction (Fast and Slow)**
```python
def remove_duplicates(arr):
    """
    Remove duplicates in-place, return new length
    Time: O(n), Space: O(1)
    """
    if not arr:
        return 0
    
    slow = 0
    for fast in range(1, len(arr)):
        if arr[fast] != arr[slow]:
            slow += 1
            arr[slow] = arr[fast]
    
    return slow + 1

# Example
arr = [1, 1, 2, 2, 3, 4, 4, 5]
new_length = remove_duplicates(arr)
print(arr[:new_length])  # Output: [1, 2, 3, 4, 5]
```

**Pattern 3: Container With Most Water**
```python
def max_area(height):
    """
    Find two lines that form container with most water
    Time: O(n), Space: O(1)
    """
    left, right = 0, len(height) - 1
    max_water = 0
    
    while left < right:
        width = right - left
        current_water = min(height[left], height[right]) * width
        max_water = max(max_water, current_water)
        
        # Move pointer with smaller height
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    
    return max_water

# Example
height = [1, 8, 6, 2, 5, 4, 8, 3, 7]
print(max_area(height))  # Output: 49
```

**Pattern 4: Trapping Rain Water**
```python
def trap_rain_water(height):
    """
    Calculate trapped rainwater
    Time: O(n), Space: O(1)
    """
    if not height:
        return 0
    
    left, right = 0, len(height) - 1
    left_max, right_max = 0, 0
    water = 0
    
    while left < right:
        if height[left] < height[right]:
            if height[left] >= left_max:
                left_max = height[left]
            else:
                water += left_max - height[left]
            left += 1
        else:
            if height[right] >= right_max:
                right_max = height[right]
            else:
                water += right_max - height[right]
            right -= 1
    
    return water

# Example
height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]
print(trap_rain_water(height))  # Output: 6
```

---

## 6. Sliding Window Technique

### 6.1 Concept

Maintain a window that slides through array, useful for subarray/substring problems.

### 6.2 Fixed Window Size

```python
def max_sum_subarray(arr, k):
    """
    Maximum sum of subarray of size k
    Time: O(n), Space: O(1)
    """
    if len(arr) < k:
        return 0
    
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# Example
arr = [1, 4, 2, 10, 23, 3, 1, 0, 20]
k = 4
print(max_sum_subarray(arr, k))  # Output: 39
```

### 6.3 Variable Window Size

```python
def longest_substring_no_repeat(s):
    """
    Longest substring without repeating characters
    Time: O(n), Space: O(min(n, m)) where m is charset size
    """
    char_map = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        # If character seen and within current window
        if s[right] in char_map and char_map[s[right]] >= left:
            left = char_map[s[right]] + 1
        
        char_map[s[right]] = right
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Example
s = "abcabcbb"
print(longest_substring_no_repeat(s))  # Output: 3 ("abc")

def min_window_substring(s, t):
    """
    Minimum window substring containing all characters of t
    Time: O(|s| + |t|), Space: O(|s| + |t|)
    """
    if not s or not t:
        return ""
    
    # Count characters in t
    need = {}
    for char in t:
        need[char] = need.get(char, 0) + 1
    
    left = 0
    valid = 0  # Number of unique characters satisfied
    window = {}
    start = 0
    min_len = float('inf')
    
    for right in range(len(s)):
        char = s[right]
        
        # Expand window
        if char in need:
            window[char] = window.get(char, 0) + 1
            if window[char] == need[char]:
                valid += 1
        
        # Shrink window
        while valid == len(need):
            # Update minimum window
            if right - left + 1 < min_len:
                min_len = right - left + 1
                start = left
            
            # Remove left character
            left_char = s[left]
            if left_char in need:
                if window[left_char] == need[left_char]:
                    valid -= 1
                window[left_char] -= 1
            
            left += 1
    
    return "" if min_len == float('inf') else s[start:start + min_len]

# Example
s = "ADOBECODEBANC"
t = "ABC"
print(min_window_substring(s, t))  # Output: "BANC"
```

### 6.4 At Most K Pattern

```python
def longest_substring_at_most_k(s, k):
    """
    Longest substring with at most k distinct characters
    Time: O(n), Space: O(k)
    """
    char_count = {}
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        char_count[s[right]] = char_count.get(s[right], 0) + 1
        
        # Shrink window if more than k distinct characters
        while len(char_count) > k:
            char_count[s[left]] -= 1
            if char_count[s[left]] == 0:
                del char_count[s[left]]
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Example
s = "eceba"
k = 2
print(longest_substring_at_most_k(s, k))  # Output: 3 ("ece")
```

---

## 7. Prefix Sums

### 7.1 Concept

Precompute cumulative sums to answer range sum queries in O(1).

### 7.2 1D Prefix Sum

```python
def prefix_sum(arr):
    """
    Create prefix sum array
    prefix[i] = sum of arr[0] to arr[i]
    Time: O(n), Space: O(n)
    """
    prefix = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        prefix[i + 1] = prefix[i] + arr[i]
    return prefix

def range_sum(prefix, left, right):
    """
    Get sum of arr[left] to arr[right] inclusive
    Time: O(1)
    """
    return prefix[right + 1] - prefix[left]

# Example
arr = [1, 2, 3, 4, 5]
prefix = prefix_sum(arr)
print(range_sum(prefix, 1, 3))  # Output: 9 (2 + 3 + 4)
```

### 7.3 2D Prefix Sum

```python
def prefix_sum_2d(matrix):
    """
    Create 2D prefix sum matrix
    Time: O(n * m), Space: O(n * m)
    """
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
    
    return prefix

def range_sum_2d(prefix, row1, col1, row2, col2):
    """
    Get sum of submatrix from (row1, col1) to (row2, col2)
    Time: O(1)
    """
    return (
        prefix[row2 + 1][col2 + 1] -
        prefix[row1][col2 + 1] -
        prefix[row2 + 1][col1] +
        prefix[row1][col1]
    )

# Example
matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
]
prefix = prefix_sum_2d(matrix)
print(range_sum_2d(prefix, 1, 1, 2, 2))  # Output: 28 (5+6+8+9)
```

### 7.4 Applications

**Problem: Subarray Sum Equals K**
```python
def subarray_sum_equals_k(arr, k):
    """
    Count subarrays with sum equal to k
    Time: O(n), Space: O(n)
    """
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

# Example
arr = [1, 1, 1]
k = 2
print(subarray_sum_equals_k(arr, k))  # Output: 2
```

---

## 8. Practice Problems

### Problem Set 1: Arrays

1. **Best Time to Buy and Sell Stock**
```python
def max_profit(prices):
    """
    Maximum profit from buying and selling stock once
    Time: O(n), Space: O(1)
    """
    min_price = float('inf')
    max_profit = 0
    
    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)
    
    return max_profit
```

2. **Merge Sorted Arrays**
```python
def merge_sorted_arrays(nums1, m, nums2, n):
    """
    Merge nums2 into nums1 in-place
    Time: O(m + n), Space: O(1)
    """
    i, j, k = m - 1, n - 1, m + n - 1
    
    while i >= 0 and j >= 0:
        if nums1[i] > nums2[j]:
            nums1[k] = nums1[i]
            i -= 1
        else:
            nums1[k] = nums2[j]
            j -= 1
        k -= 1
    
    # Copy remaining elements from nums2
    while j >= 0:
        nums1[k] = nums2[j]
        j -= 1
        k -= 1
```

3. **Find All Duplicates**
```python
def find_duplicates(arr):
    """
    Find all duplicates in array (1 ≤ arr[i] ≤ n)
    Time: O(n), Space: O(1) excluding output
    """
    result = []
    for num in arr:
        index = abs(num) - 1
        if arr[index] < 0:
            result.append(abs(num))
        else:
            arr[index] = -arr[index]
    return result
```

### Problem Set 2: Two Pointers

1. **3Sum**
```python
def three_sum(arr):
    """
    Find all unique triplets that sum to zero
    Time: O(n^2), Space: O(1) excluding output
    """
    arr.sort()
    result = []
    
    for i in range(len(arr) - 2):
        # Skip duplicates
        if i > 0 and arr[i] == arr[i - 1]:
            continue
        
        left, right = i + 1, len(arr) - 1
        while left < right:
            current_sum = arr[i] + arr[left] + arr[right]
            if current_sum == 0:
                result.append([arr[i], arr[left], arr[right]])
                # Skip duplicates
                while left < right and arr[left] == arr[left + 1]:
                    left += 1
                while left < right and arr[right] == arr[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif current_sum < 0:
                left += 1
            else:
                right -= 1
    
    return result
```

2. **Valid Palindrome**
```python
def is_palindrome(s):
    """
    Check if string is palindrome (ignoring non-alphanumeric)
    Time: O(n), Space: O(1)
    """
    left, right = 0, len(s) - 1
    
    while left < right:
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        
        if s[left].lower() != s[right].lower():
            return False
        
        left += 1
        right -= 1
    
    return True
```

### Problem Set 3: Sliding Window

1. **Longest Repeating Character Replacement**
```python
def character_replacement(s, k):
    """
    Longest substring with same character after k replacements
    Time: O(n), Space: O(1)
    """
    char_count = {}
    left = 0
    max_count = 0
    max_length = 0
    
    for right in range(len(s)):
        char_count[s[right]] = char_count.get(s[right], 0) + 1
        max_count = max(max_count, char_count[s[right]])
        
        # If window needs more than k replacements, shrink
        if (right - left + 1) - max_count > k:
            char_count[s[left]] -= 1
            left += 1
        
        max_length = max(max_length, right - left + 1)
    
    return max_length
```

2. **Permutation in String**
```python
def check_inclusion(s1, s2):
    """
    Check if s2 contains permutation of s1
    Time: O(n), Space: O(1)
    """
    if len(s1) > len(s2):
        return False
    
    s1_count = {}
    for char in s1:
        s1_count[char] = s1_count.get(char, 0) + 1
    
    window_count = {}
    left = 0
    
    for right in range(len(s2)):
        window_count[s2[right]] = window_count.get(s2[right], 0) + 1
        
        # Maintain window of size len(s1)
        if right - left + 1 > len(s1):
            window_count[s2[left]] -= 1
            if window_count[s2[left]] == 0:
                del window_count[s2[left]]
            left += 1
        
        # Check if window matches s1
        if window_count == s1_count:
            return True
    
    return False
```

---

## Summary of Part 1

### Key Concepts

1. **Time/Space Complexity**: Big O notation, common complexities
2. **Arrays**: Basic operations, common problems
3. **Two Pointers**: Opposite ends, same direction patterns
4. **Sliding Window**: Fixed and variable window sizes
5. **Prefix Sums**: 1D and 2D prefix sums for range queries

### Common Patterns

- **Kadane's Algorithm**: Maximum subarray
- **Two Pointers**: Opposite ends, fast/slow
- **Sliding Window**: Fixed/variable size, at most k
- **Prefix Sum**: Range sum queries

### Next Steps

**Part 2** will cover:
- Linked Lists
- Stacks and Queues
- Hash Tables and Sets
- String Algorithms

---

**Practice these concepts on LeetCode Easy/Medium problems before moving to Part 2!**

