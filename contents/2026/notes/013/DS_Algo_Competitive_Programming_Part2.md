# Data Structures & Algorithms for Competitive Programming

## Part 2: Linked Lists, Stacks, Queues, and Hash Tables

---

## Table of Contents

1. [Linked Lists](#1-linked-lists)
2. [Stacks](#2-stacks)
3. [Queues](#3-queues)
4. [Hash Tables and Sets](#4-hash-tables-and-sets)
5. [Practice Problems](#5-practice-problems)

---

## 1. Linked Lists

### 1.1 Linked List Node Definition

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

# Helper function to create linked list from array
def create_linked_list(arr):
    if not arr:
        return None
    head = ListNode(arr[0])
    current = head
    for val in arr[1:]:
        current.next = ListNode(val)
        current = current.next
    return head

# Helper function to convert linked list to array
def linked_list_to_array(head):
    result = []
    current = head
    while current:
        result.append(current.val)
        current = current.next
    return result
```

### 1.2 Basic Operations

```python
def traverse(head):
    """Traverse linked list"""
    current = head
    while current:
        print(current.val)
        current = current.next

def search(head, target):
    """Search for value in linked list"""
    current = head
    while current:
        if current.val == target:
            return True
        current = current.next
    return False

def insert_at_head(head, val):
    """Insert at head - O(1)"""
    new_node = ListNode(val)
    new_node.next = head
    return new_node

def insert_after(node, val):
    """Insert after given node - O(1)"""
    new_node = ListNode(val)
    new_node.next = node.next
    node.next = new_node

def delete_node(node):
    """Delete node (given node, not head) - O(1)"""
    if node.next:
        node.val = node.next.val
        node.next = node.next.next
    # If last node, cannot delete without head reference
```

### 1.3 Common Linked List Problems

**Problem 1: Reverse Linked List**
```python
def reverse_list(head):
    """
    Reverse linked list iteratively
    Time: O(n), Space: O(1)
    """
    prev = None
    current = head
    
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    return prev

def reverse_list_recursive(head):
    """
    Reverse linked list recursively
    Time: O(n), Space: O(n) for call stack
    """
    if not head or not head.next:
        return head
    
    new_head = reverse_list_recursive(head.next)
    head.next.next = head
    head.next = None
    
    return new_head
```

**Problem 2: Merge Two Sorted Lists**
```python
def merge_two_lists(list1, list2):
    """
    Merge two sorted linked lists
    Time: O(n + m), Space: O(1)
    """
    dummy = ListNode(0)
    current = dummy
    
    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next
        current = current.next
    
    # Append remaining nodes
    current.next = list1 if list1 else list2
    
    return dummy.next
```

**Problem 3: Detect Cycle (Floyd's Algorithm)**
```python
def has_cycle(head):
    """
    Detect cycle in linked list using Floyd's cycle detection
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return False
    
    slow = head
    fast = head.next
    
    while fast and fast.next:
        if slow == fast:
            return True
        slow = slow.next
        fast = fast.next.next
    
    return False

def detect_cycle_start(head):
    """
    Find the node where cycle begins
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return None
    
    # Find meeting point
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    else:
        return None  # No cycle
    
    # Find cycle start
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next
    
    return slow
```

**Problem 4: Remove Nth Node From End**
```python
def remove_nth_from_end(head, n):
    """
    Remove nth node from end of list
    Time: O(n), Space: O(1)
    """
    dummy = ListNode(0)
    dummy.next = head
    first = second = dummy
    
    # Move first pointer n+1 steps ahead
    for _ in range(n + 1):
        first = first.next
    
    # Move both pointers until first reaches end
    while first:
        first = first.next
        second = second.next
    
    # Remove node
    second.next = second.next.next
    
    return dummy.next
```

**Problem 5: Add Two Numbers**
```python
def add_two_numbers(l1, l2):
    """
    Add two numbers represented as linked lists
    Time: O(max(n, m)), Space: O(max(n, m))
    """
    dummy = ListNode(0)
    current = dummy
    carry = 0
    
    while l1 or l2 or carry:
        val1 = l1.val if l1 else 0
        val2 = l2.val if l2 else 0
        
        total = val1 + val2 + carry
        carry = total // 10
        current.next = ListNode(total % 10)
        
        current = current.next
        l1 = l1.next if l1 else None
        l2 = l2.next if l2 else None
    
    return dummy.next
```

**Problem 6: Reverse Nodes in k-Group**
```python
def reverse_k_group(head, k):
    """
    Reverse nodes in groups of k
    Time: O(n), Space: O(1)
    """
    def reverse_range(start, end):
        """Reverse linked list from start to end (exclusive)"""
        prev = end
        current = start
        while current != end:
            next_node = current.next
            current.next = prev
            prev = current
            current = next_node
        return prev
    
    # Check if we have k nodes
    count = 0
    current = head
    while current and count < k:
        current = current.next
        count += 1
    
    if count == k:
        # Reverse first k nodes
        current = reverse_k_group(current, k)  # Recurse for rest
        head = reverse_range(head, current)
    
    return head
```

**Problem 7: Copy List with Random Pointer**
```python
class Node:
    def __init__(self, val=0, next=None, random=None):
        self.val = val
        self.next = next
        self.random = random

def copy_random_list(head):
    """
    Deep copy linked list with random pointers
    Time: O(n), Space: O(n)
    """
    if not head:
        return None
    
    # Create mapping of old nodes to new nodes
    node_map = {}
    current = head
    
    # First pass: create all nodes
    while current:
        node_map[current] = Node(current.val)
        current = current.next
    
    # Second pass: set next and random pointers
    current = head
    while current:
        if current.next:
            node_map[current].next = node_map[current.next]
        if current.random:
            node_map[current].random = node_map[current.random]
        current = current.next
    
    return node_map[head]
```

---

## 2. Stacks

### 2.1 Stack Implementation

```python
class Stack:
    def __init__(self):
        self.items = []
    
    def push(self, item):
        """Add item to top - O(1)"""
        self.items.append(item)
    
    def pop(self):
        """Remove and return top item - O(1)"""
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items.pop()
    
    def peek(self):
        """Return top item without removing - O(1)"""
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.items[-1]
    
    def is_empty(self):
        """Check if stack is empty - O(1)"""
        return len(self.items) == 0
    
    def size(self):
        """Return size - O(1)"""
        return len(self.items)

# Python list can be used as stack
stack = []
stack.append(1)  # push
stack.pop()      # pop
stack[-1]        # peek
```

### 2.2 Common Stack Problems

**Problem 1: Valid Parentheses**
```python
def is_valid(s):
    """
    Check if parentheses are valid
    Time: O(n), Space: O(n)
    """
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in mapping:
            # Closing bracket
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            # Opening bracket
            stack.append(char)
    
    return len(stack) == 0
```

**Problem 2: Daily Temperatures**
```python
def daily_temperatures(temperatures):
    """
    For each day, find number of days until warmer temperature
    Time: O(n), Space: O(n)
    """
    stack = []
    result = [0] * len(temperatures)
    
    for i, temp in enumerate(temperatures):
        while stack and temperatures[stack[-1]] < temp:
            prev_index = stack.pop()
            result[prev_index] = i - prev_index
        stack.append(i)
    
    return result

# Example
temps = [73, 74, 75, 71, 69, 72, 76, 73]
print(daily_temperatures(temps))  # Output: [1, 1, 4, 2, 1, 1, 0, 0]
```

**Problem 3: Largest Rectangle in Histogram**
```python
def largest_rectangle_area(heights):
    """
    Find largest rectangle area in histogram
    Time: O(n), Space: O(n)
    """
    stack = []
    max_area = 0
    
    for i, height in enumerate(heights):
        while stack and heights[stack[-1]] > height:
            h = heights[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, h * width)
        stack.append(i)
    
    # Process remaining bars
    while stack:
        h = heights[stack.pop()]
        width = len(heights) if not stack else len(heights) - stack[-1] - 1
        max_area = max(max_area, h * width)
    
    return max_area
```

**Problem 4: Next Greater Element**
```python
def next_greater_element(nums1, nums2):
    """
    Find next greater element for each element in nums1
    Time: O(n + m), Space: O(n + m)
    """
    stack = []
    next_greater = {}
    
    # Build next greater map for nums2
    for num in nums2:
        while stack and stack[-1] < num:
            next_greater[stack.pop()] = num
        stack.append(num)
    
    # Find next greater for nums1
    result = []
    for num in nums1:
        result.append(next_greater.get(num, -1))
    
    return result
```

**Problem 5: Evaluate Reverse Polish Notation**
```python
def eval_rpn(tokens):
    """
    Evaluate expression in Reverse Polish Notation
    Time: O(n), Space: O(n)
    """
    stack = []
    operators = {'+', '-', '*', '/'}
    
    for token in tokens:
        if token in operators:
            b = stack.pop()
            a = stack.pop()
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            else:  # division
                stack.append(int(a / b))  # Truncate towards zero
        else:
            stack.append(int(token))
    
    return stack[0]

# Example
tokens = ["2", "1", "+", "3", "*"]
print(eval_rpn(tokens))  # Output: 9 ((2+1)*3)
```

**Problem 6: Decode String**
```python
def decode_string(s):
    """
    Decode string like "3[a2[c]]" -> "accaccacc"
    Time: O(n), Space: O(n)
    """
    stack = []
    current_string = ""
    current_num = 0
    
    for char in s:
        if char.isdigit():
            current_num = current_num * 10 + int(char)
        elif char == '[':
            stack.append((current_string, current_num))
            current_string = ""
            current_num = 0
        elif char == ']':
            prev_string, num = stack.pop()
            current_string = prev_string + current_string * num
        else:
            current_string += char
    
    return current_string

# Example
s = "3[a2[c]]"
print(decode_string(s))  # Output: "accaccacc"
```

---

## 3. Queues

### 3.1 Queue Implementation

```python
from collections import deque

class Queue:
    def __init__(self):
        self.items = deque()
    
    def enqueue(self, item):
        """Add item to rear - O(1)"""
        self.items.append(item)
    
    def dequeue(self):
        """Remove and return front item - O(1)"""
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items.popleft()
    
    def front(self):
        """Return front item without removing - O(1)"""
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self.items[0]
    
    def is_empty(self):
        """Check if queue is empty - O(1)"""
        return len(self.items) == 0
    
    def size(self):
        """Return size - O(1)"""
        return len(self.items)

# Using deque directly
queue = deque()
queue.append(1)    # enqueue
queue.popleft()     # dequeue
queue[0]            # front
```

### 3.2 Priority Queue (Heap)

```python
import heapq

# Min heap (default)
heap = []
heapq.heappush(heap, 3)
heapq.heappush(heap, 1)
heapq.heappush(heap, 2)
print(heapq.heappop(heap))  # Output: 1 (smallest)

# Max heap (negate values)
max_heap = []
heapq.heappush(max_heap, -3)
heapq.heappush(max_heap, -1)
heapq.heappush(max_heap, -2)
print(-heapq.heappop(max_heap))  # Output: 3 (largest)

# Heapify existing list
arr = [3, 1, 4, 1, 5, 9, 2, 6]
heapq.heapify(arr)  # O(n)
```

### 3.3 Common Queue Problems

**Problem 1: Sliding Window Maximum**
```python
from collections import deque

def max_sliding_window(nums, k):
    """
    Find maximum in each sliding window of size k
    Time: O(n), Space: O(k)
    """
    dq = deque()  # Store indices
    result = []
    
    for i in range(len(nums)):
        # Remove indices outside window
        while dq and dq[0] <= i - k:
            dq.popleft()
        
        # Remove indices with smaller values
        while dq and nums[dq[-1]] < nums[i]:
            dq.pop()
        
        dq.append(i)
        
        # Add to result when window is complete
        if i >= k - 1:
            result.append(nums[dq[0]])
    
    return result

# Example
nums = [1, 3, -1, -3, 5, 3, 6, 7]
k = 3
print(max_sliding_window(nums, k))  # Output: [3, 3, 5, 5, 6, 7]
```

**Problem 2: Design Circular Queue**
```python
class MyCircularQueue:
    def __init__(self, k):
        """
        Initialize circular queue of size k
        """
        self.queue = [0] * k
        self.head = 0
        self.count = 0
        self.capacity = k
    
    def enQueue(self, value):
        """
        Insert element into circular queue
        Time: O(1)
        """
        if self.isFull():
            return False
        self.queue[(self.head + self.count) % self.capacity] = value
        self.count += 1
        return True
    
    def deQueue(self):
        """
        Delete element from circular queue
        Time: O(1)
        """
        if self.isEmpty():
            return False
        self.head = (self.head + 1) % self.capacity
        self.count -= 1
        return True
    
    def Front(self):
        """
        Get front item
        Time: O(1)
        """
        if self.isEmpty():
            return -1
        return self.queue[self.head]
    
    def Rear(self):
        """
        Get rear item
        Time: O(1)
        """
        if self.isEmpty():
            return -1
        return self.queue[(self.head + self.count - 1) % self.capacity]
    
    def isEmpty(self):
        """Check if queue is empty"""
        return self.count == 0
    
    def isFull(self):
        """Check if queue is full"""
        return self.count == self.capacity
```

**Problem 3: Task Scheduler**
```python
import heapq
from collections import Counter

def least_interval(tasks, n):
    """
    Find minimum time to complete all tasks with cooldown
    Time: O(n * m) where m is unique tasks, Space: O(1)
    """
    task_counts = Counter(tasks)
    max_heap = [-count for count in task_counts.values()]
    heapq.heapify(max_heap)
    
    time = 0
    queue = []  # (count, available_time)
    
    while max_heap or queue:
        time += 1
        
        if max_heap:
            count = heapq.heappop(max_heap)
            count += 1  # Decrease count
            if count < 0:
                queue.append((count, time + n))
        
        if queue and queue[0][1] == time:
            heapq.heappush(max_heap, queue.pop(0)[0])
    
    return time
```

---

## 4. Hash Tables and Sets

### 4.1 Dictionary (Hash Table) Operations

```python
# Basic operations
d = {}
d['key'] = 'value'  # O(1) average
value = d.get('key', default)  # O(1) average
del d['key']  # O(1) average
'key' in d  # O(1) average

# Defaultdict (no KeyError)
from collections import defaultdict
dd = defaultdict(int)
dd['key'] += 1  # Automatically initializes to 0

# Counter
from collections import Counter
counter = Counter([1, 2, 2, 3, 3, 3])
print(counter[2])  # Output: 2
print(counter.most_common(2))  # Output: [(3, 3), (2, 2)]
```

### 4.2 Set Operations

```python
# Basic operations
s = set()
s.add(1)  # O(1) average
s.remove(1)  # O(1) average
1 in s  # O(1) average

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union = set1 | set2  # {1, 2, 3, 4, 5}
intersection = set1 & set2  # {3}
difference = set1 - set2  # {1, 2}
symmetric_diff = set1 ^ set2  # {1, 2, 4, 5}
```

### 4.3 Common Hash Table Problems

**Problem 1: Group Anagrams**
```python
from collections import defaultdict

def group_anagrams(strs):
    """
    Group strings that are anagrams
    Time: O(n * k log k) where k is max string length, Space: O(n * k)
    """
    groups = defaultdict(list)
    
    for s in strs:
        # Sort string to get key
        key = ''.join(sorted(s))
        groups[key].append(s)
    
    return list(groups.values())

# Optimized version (count characters)
def group_anagrams_optimized(strs):
    """
    Time: O(n * k), Space: O(n * k)
    """
    groups = defaultdict(list)
    
    for s in strs:
        count = [0] * 26
        for char in s:
            count[ord(char) - ord('a')] += 1
        key = tuple(count)
        groups[key].append(s)
    
    return list(groups.values())
```

**Problem 2: Longest Consecutive Sequence**
```python
def longest_consecutive(nums):
    """
    Find length of longest consecutive sequence
    Time: O(n), Space: O(n)
    """
    num_set = set(nums)
    max_length = 0
    
    for num in num_set:
        # Only start from beginning of sequence
        if num - 1 not in num_set:
            current_num = num
            current_length = 1
            
            while current_num + 1 in num_set:
                current_num += 1
                current_length += 1
            
            max_length = max(max_length, current_length)
    
    return max_length
```

**Problem 3: Two Sum (Hash Table)**
```python
def two_sum(nums, target):
    """
    Find two numbers that add up to target
    Time: O(n), Space: O(n)
    """
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

**Problem 4: Contains Duplicate**
```python
def contains_duplicate(nums):
    """
    Check if array contains duplicates
    Time: O(n), Space: O(n)
    """
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False

def contains_nearby_duplicate(nums, k):
    """
    Check if there are duplicates within k distance
    Time: O(n), Space: O(k)
    """
    seen = {}
    for i, num in enumerate(nums):
        if num in seen and i - seen[num] <= k:
            return True
        seen[num] = i
    return False
```

**Problem 5: Design HashMap**
```python
class MyHashMap:
    def __init__(self):
        """
        Initialize hash map
        """
        self.size = 1000
        self.buckets = [[] for _ in range(self.size)]
    
    def _hash(self, key):
        """Hash function"""
        return key % self.size
    
    def put(self, key, value):
        """
        Insert or update key-value pair
        Time: O(1) average, O(n) worst case
        """
        bucket = self.buckets[self._hash(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)
                return
        bucket.append((key, value))
    
    def get(self, key):
        """
        Get value for key
        Time: O(1) average, O(n) worst case
        """
        bucket = self.buckets[self._hash(key)]
        for k, v in bucket:
            if k == key:
                return v
        return -1
    
    def remove(self, key):
        """
        Remove key-value pair
        Time: O(1) average, O(n) worst case
        """
        bucket = self.buckets[self._hash(key)]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket.pop(i)
                return
```

**Problem 6: LRU Cache**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        """
        Initialize LRU cache with capacity
        """
        self.capacity = capacity
        self.cache = OrderedDict()
    
    def get(self, key):
        """
        Get value for key, move to end (most recently used)
        Time: O(1)
        """
        if key not in self.cache:
            return -1
        # Move to end
        self.cache.move_to_end(key)
        return self.cache[key]
    
    def put(self, key, value):
        """
        Insert or update key-value pair
        Time: O(1)
        """
        if key in self.cache:
            # Update existing
            self.cache.move_to_end(key)
        else:
            # Check capacity
            if len(self.cache) >= self.capacity:
                # Remove least recently used (first item)
                self.cache.popitem(last=False)
        self.cache[key] = value
```

---

## 5. Practice Problems

### Problem Set 1: Linked Lists

1. **Swap Nodes in Pairs**
```python
def swap_pairs(head):
    """
    Swap every two adjacent nodes
    Time: O(n), Space: O(1)
    """
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy
    
    while prev.next and prev.next.next:
        first = prev.next
        second = prev.next.next
        
        # Swap
        prev.next = second
        first.next = second.next
        second.next = first
        
        prev = first
    
    return dummy.next
```

2. **Reorder List**
```python
def reorder_list(head):
    """
    Reorder: L0 → L1 → ... → Ln-1 → Ln
    To: L0 → Ln → L1 → Ln-1 → ...
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return
    
    # Find middle
    slow = fast = head
    while fast.next and fast.next.next:
        slow = slow.next
        fast = fast.next.next
    
    # Reverse second half
    prev = None
    current = slow.next
    slow.next = None
    
    while current:
        next_node = current.next
        current.next = prev
        prev = current
        current = next_node
    
    # Merge two halves
    first, second = head, prev
    while second:
        temp1, temp2 = first.next, second.next
        first.next = second
        second.next = temp1
        first, second = temp1, temp2
```

### Problem Set 2: Stacks

1. **Basic Calculator**
```python
def calculate(s):
    """
    Evaluate expression with +, -, (, )
    Time: O(n), Space: O(n)
    """
    stack = []
    num = 0
    sign = 1
    result = 0
    
    for char in s:
        if char.isdigit():
            num = num * 10 + int(char)
        elif char == '+':
            result += sign * num
            num = 0
            sign = 1
        elif char == '-':
            result += sign * num
            num = 0
            sign = -1
        elif char == '(':
            stack.append(result)
            stack.append(sign)
            result = 0
            sign = 1
        elif char == ')':
            result += sign * num
            num = 0
            result *= stack.pop()  # sign
            result += stack.pop()  # previous result
    
    return result + sign * num
```

2. **Remove K Digits**
```python
def remove_k_digits(num, k):
    """
    Remove k digits to get smallest number
    Time: O(n), Space: O(n)
    """
    stack = []
    
    for digit in num:
        while k > 0 and stack and stack[-1] > digit:
            stack.pop()
            k -= 1
        stack.append(digit)
    
    # Remove remaining k digits
    while k > 0:
        stack.pop()
        k -= 1
    
    # Remove leading zeros
    result = ''.join(stack).lstrip('0')
    
    return result if result else '0'
```

### Problem Set 3: Hash Tables

1. **Subarray Sum Equals K**
```python
def subarray_sum(nums, k):
    """
    Count subarrays with sum equal to k
    Time: O(n), Space: O(n)
    """
    prefix_sum = 0
    count = 0
    sum_count = {0: 1}
    
    for num in nums:
        prefix_sum += num
        if prefix_sum - k in sum_count:
            count += sum_count[prefix_sum - k]
        sum_count[prefix_sum] = sum_count.get(prefix_sum, 0) + 1
    
    return count
```

2. **Word Pattern**
```python
def word_pattern(pattern, s):
    """
    Check if string follows pattern
    Time: O(n), Space: O(n)
    """
    words = s.split()
    if len(pattern) != len(words):
        return False
    
    char_to_word = {}
    word_to_char = {}
    
    for char, word in zip(pattern, words):
        if char in char_to_word:
            if char_to_word[char] != word:
                return False
        else:
            if word in word_to_char:
                return False
            char_to_word[char] = word
            word_to_char[word] = char
    
    return True
```

---

## Summary of Part 2

### Key Data Structures

1. **Linked Lists**: Single, double, circular variations
2. **Stacks**: LIFO, useful for parentheses, monotonic stack
3. **Queues**: FIFO, deque, priority queue (heap)
4. **Hash Tables**: Dictionary, defaultdict, Counter
5. **Sets**: Fast membership testing, set operations

### Common Patterns

- **Two Pointers in Linked Lists**: Fast/slow, reverse
- **Monotonic Stack**: Next greater/smaller element
- **Sliding Window with Deque**: Maximum in window
- **Hash Table for Lookup**: O(1) average access
- **Prefix Sum with Hash**: Subarray sum problems

### Next Steps

**Part 3** will cover:
- Trees and Binary Trees
- Binary Search Trees
- Tree Traversals
- Tree Construction Problems

---

**Practice these concepts before moving to Part 3!**

