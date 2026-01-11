# Data Structures & Algorithms for Competitive Programming

## Part 8: String Algorithms

---

## Table of Contents

1. [KMP Algorithm](#1-kmp-algorithm)
2. [Z-Algorithm](#2-z-algorithm)
3. [Rabin-Karp Algorithm](#3-rabin-karp-algorithm)
4. [Manacher's Algorithm](#4-manachers-algorithm)
5. [Suffix Array](#5-suffix-array)
6. [Practice Problems](#6-practice-problems)

---

## 1. KMP Algorithm

### 1.1 KMP for Pattern Matching

```python
def kmp_search(text, pattern):
    """
    Find all occurrences of pattern in text using KMP
    Time: O(n + m), Space: O(m)
    """
    def build_lps(pattern):
        """
        Build Longest Proper Prefix which is also Suffix
        """
        lps = [0] * len(pattern)
        length = 0
        i = 1
        
        while i < len(pattern):
            if pattern[i] == pattern[length]:
                length += 1
                lps[i] = length
                i += 1
            else:
                if length != 0:
                    length = lps[length - 1]
                else:
                    lps[i] = 0
                    i += 1
        
        return lps
    
    if not pattern:
        return []
    
    lps = build_lps(pattern)
    result = []
    i = j = 0  # i for text, j for pattern
    
    while i < len(text):
        if text[i] == pattern[j]:
            i += 1
            j += 1
        
        if j == len(pattern):
            result.append(i - j)
            j = lps[j - 1]
        elif i < len(text) and text[i] != pattern[j]:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
    
    return result

# Example
text = "ABABDABACDABABCABCAB"
pattern = "ABABCABAB"
print(kmp_search(text, pattern))  # Output: [10]
```

### 1.2 Repeated Substring Pattern

```python
def repeated_substring_pattern(s):
    """
    Check if string can be constructed by repeating substring
    Time: O(n), Space: O(n)
    """
    def build_lps(pattern):
        lps = [0] * len(pattern)
        length = 0
        i = 1
        
        while i < len(pattern):
            if pattern[i] == pattern[length]:
                length += 1
                lps[i] = length
                i += 1
            else:
                if length != 0:
                    length = lps[length - 1]
                else:
                    i += 1
        
        return lps
    
    lps = build_lps(s)
    n = len(s)
    # If lps[n-1] > 0 and n % (n - lps[n-1]) == 0, pattern repeats
    return lps[n - 1] > 0 and n % (n - lps[n - 1]) == 0
```

---

## 2. Z-Algorithm

### 2.1 Z-Array Construction

```python
def build_z_array(s):
    """
    Build Z-array: Z[i] = longest substring starting at i that is also prefix
    Time: O(n), Space: O(n)
    """
    n = len(s)
    z = [0] * n
    left = right = 0
    
    for i in range(1, n):
        if i <= right:
            z[i] = min(right - i + 1, z[i - left])
        
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        
        if i + z[i] - 1 > right:
            left = i
            right = i + z[i] - 1
    
    return z

def z_search(text, pattern):
    """
    Find all occurrences of pattern in text using Z-algorithm
    Time: O(n + m), Space: O(n + m)
    """
    concat = pattern + '$' + text
    z = build_z_array(concat)
    result = []
    
    for i in range(len(pattern) + 1, len(concat)):
        if z[i] == len(pattern):
            result.append(i - len(pattern) - 1)
    
    return result
```

---

## 3. Rabin-Karp Algorithm

### 3.1 Rolling Hash

```python
def rabin_karp(text, pattern):
    """
    Pattern matching using rolling hash
    Time: O(n + m) average, O(nm) worst, Space: O(1)
    """
    n, m = len(text), len(pattern)
    if m > n:
        return []
    
    base = 256
    mod = 10**9 + 7
    
    # Calculate hash of pattern and first window
    pattern_hash = 0
    window_hash = 0
    h = 1
    
    for i in range(m):
        pattern_hash = (pattern_hash * base + ord(pattern[i])) % mod
        window_hash = (window_hash * base + ord(text[i])) % mod
        if i < m - 1:
            h = (h * base) % mod
    
    result = []
    
    # Slide window
    for i in range(n - m + 1):
        if pattern_hash == window_hash:
            # Verify (to avoid hash collisions)
            if text[i:i+m] == pattern:
                result.append(i)
        
        if i < n - m:
            window_hash = ((window_hash - ord(text[i]) * h) * base + ord(text[i + m])) % mod
            window_hash = (window_hash + mod) % mod
    
    return result
```

---

## 4. Manacher's Algorithm

### 4.1 Longest Palindromic Substring

```python
def manacher(s):
    """
    Find longest palindromic substring using Manacher's algorithm
    Time: O(n), Space: O(n)
    """
    # Transform string: "abc" -> "^#a#b#c#$"
    transformed = '^#' + '#'.join(s) + '#$'
    n = len(transformed)
    p = [0] * n
    center = right = 0
    
    for i in range(1, n - 1):
        # Mirror index
        mirror = 2 * center - i
        
        if i < right:
            p[i] = min(right - i, p[mirror])
        
        # Expand around center
        while transformed[i + p[i] + 1] == transformed[i - p[i] - 1]:
            p[i] += 1
        
        # Update center and right boundary
        if i + p[i] > right:
            center = i
            right = i + p[i]
    
    # Find maximum
    max_len = max(p)
    center_index = p.index(max_len)
    start = (center_index - max_len) // 2
    
    return s[start:start + max_len]
```

---

## 5. Suffix Array

### 5.1 Suffix Array Construction

```python
def build_suffix_array(s):
    """
    Build suffix array (indices of sorted suffixes)
    Time: O(n log^2 n), Space: O(n)
    """
    n = len(s)
    suffixes = []
    
    for i in range(n):
        suffixes.append((s[i:], i))
    
    suffixes.sort()
    return [index for _, index in suffixes]

def build_suffix_array_optimized(s):
    """
    Optimized suffix array construction
    Time: O(n log n), Space: O(n)
    """
    n = len(s)
    # Add sentinel
    s += '$'
    n += 1
    
    # Initial sorting by first character
    p = [0] * n
    c = [0] * n
    
    # Sort by first character
    arr = sorted([(s[i], i) for i in range(n)])
    for i in range(n):
        p[i] = arr[i][1]
    
    c[p[0]] = 0
    classes = 1
    for i in range(1, n):
        if arr[i][0] != arr[i-1][0]:
            classes += 1
        c[p[i]] = classes - 1
    
    # Iterate for k = 0, 1, 2, ...
    k = 0
    while (1 << k) < n:
        # Sort by second half
        p_new = [0] * n
        for i in range(n):
            p_new[i] = (p[i] - (1 << k) + n) % n
        
        # Count sort
        count = [0] * classes
        for i in range(n):
            count[c[p_new[i]]] += 1
        pos = [0] * classes
        for i in range(1, classes):
            pos[i] = pos[i-1] + count[i-1]
        
        p = [0] * n
        for i in range(n):
            p[pos[c[p_new[i]]]] = p_new[i]
            pos[c[p_new[i]]] += 1
        
        # Update classes
        c_new = [0] * n
        c_new[p[0]] = 0
        classes = 1
        for i in range(1, n):
            mid1 = (p[i] + (1 << k)) % n
            mid2 = (p[i-1] + (1 << k)) % n
            if c[p[i]] != c[p[i-1]] or c[mid1] != c[mid2]:
                classes += 1
            c_new[p[i]] = classes - 1
        c = c_new
        k += 1
    
    return p[1:]  # Remove sentinel
```

---

## 6. Practice Problems

### Problem Set 1: Pattern Matching

1. **Implement strStr()**
```python
def str_str(haystack, needle):
    """Find first occurrence of needle in haystack"""
    if not needle:
        return 0
    result = kmp_search(haystack, needle)
    return result[0] if result else -1
```

2. **Repeated String Match**
```python
def repeated_string_match(a, b):
    """
    Minimum times to repeat a so b is substring
    Time: O(n + m), Space: O(m)
    """
    if not b:
        return 0
    
    # Build LPS for pattern b
    def build_lps(pattern):
        lps = [0] * len(pattern)
        length = 0
        i = 1
        while i < len(pattern):
            if pattern[i] == pattern[length]:
                length += 1
                lps[i] = length
                i += 1
            else:
                if length != 0:
                    length = lps[length - 1]
                else:
                    i += 1
        return lps
    
    lps = build_lps(b)
    i = j = 0
    count = 1
    repeated = a
    
    while j < len(b):
        if i >= len(repeated):
            repeated += a
            count += 1
        
        if repeated[i] == b[j]:
            i += 1
            j += 1
        else:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
        
        # Prevent infinite loop
        if count > len(b) // len(a) + 2:
            return -1
    
    return count
```

---

## Summary of Part 8

### String Algorithms

1. **KMP**: Pattern matching in O(n + m)
2. **Z-Algorithm**: Pattern matching, string analysis
3. **Rabin-Karp**: Rolling hash for pattern matching
4. **Manacher**: Longest palindromic substring in O(n)
5. **Suffix Array**: Advanced string operations

### Next Steps

**Part 9** will cover:
- Math and Number Theory
- Combinatorics
- Prime Numbers
- GCD, LCM, Modular Arithmetic

---

**Practice string algorithm problems!**

