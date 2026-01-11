# Data Structures & Algorithms for Competitive Programming

## Part 9: Math and Number Theory

---

## Table of Contents

1. [Prime Numbers](#1-prime-numbers)
2. [GCD and LCM](#2-gcd-and-lcm)
3. [Modular Arithmetic](#3-modular-arithmetic)
4. [Combinatorics](#4-combinatorics)
5. [Bit Manipulation](#5-bit-manipulation)
6. [Practice Problems](#6-practice-problems)

---

## 1. Prime Numbers

### 1.1 Prime Checking

```python
def is_prime(n):
    """
    Check if number is prime
    Time: O(sqrt(n)), Space: O(1)
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    for i in range(3, int(n**0.5) + 1, 2):
        if n % i == 0:
            return False
    
    return True

def sieve_of_eratosthenes(n):
    """
    Generate all primes up to n
    Time: O(n log log n), Space: O(n)
    """
    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False
    
    for i in range(2, int(n**0.5) + 1):
        if is_prime[i]:
            for j in range(i * i, n + 1, i):
                is_prime[j] = False
    
    return [i for i in range(n + 1) if is_prime[i]]
```

### 1.2 Prime Factorization

```python
def prime_factors(n):
    """
    Prime factorization of n
    Time: O(sqrt(n)), Space: O(log n)
    """
    factors = []
    
    # Check for 2
    while n % 2 == 0:
        factors.append(2)
        n //= 2
    
    # Check for odd factors
    for i in range(3, int(n**0.5) + 1, 2):
        while n % i == 0:
            factors.append(i)
            n //= i
    
    if n > 2:
        factors.append(n)
    
    return factors

def count_divisors(n):
    """Count number of divisors"""
    factors = prime_factors(n)
    from collections import Counter
    count = Counter(factors)
    
    result = 1
    for freq in count.values():
        result *= (freq + 1)
    
    return result
```

---

## 2. GCD and LCM

### 2.1 Euclidean Algorithm

```python
def gcd(a, b):
    """
    Greatest Common Divisor using Euclidean algorithm
    Time: O(log min(a, b)), Space: O(1)
    """
    while b:
        a, b = b, a % b
    return a

def gcd_recursive(a, b):
    """Recursive version"""
    if b == 0:
        return a
    return gcd_recursive(b, a % b)

def lcm(a, b):
    """
    Least Common Multiple
    Time: O(log min(a, b)), Space: O(1)
    """
    return a * b // gcd(a, b)

def extended_gcd(a, b):
    """
    Extended Euclidean algorithm: ax + by = gcd(a, b)
    Returns: (gcd, x, y)
    Time: O(log min(a, b)), Space: O(log min(a, b))
    """
    if b == 0:
        return (a, 1, 0)
    
    gcd, x1, y1 = extended_gcd(b, a % b)
    x = y1
    y = x1 - (a // b) * y1
    
    return (gcd, x, y)
```

### 2.2 Applications

**Modular Inverse**:
```python
def mod_inverse(a, m):
    """
    Find modular inverse: a^(-1) mod m
    Time: O(log m), Space: O(log m)
    """
    gcd, x, y = extended_gcd(a, m)
    if gcd != 1:
        return None  # Inverse doesn't exist
    return (x % m + m) % m
```

---

## 3. Modular Arithmetic

### 3.1 Basic Operations

```python
MOD = 10**9 + 7

def mod_add(a, b, mod=MOD):
    """Modular addition"""
    return (a + b) % mod

def mod_sub(a, b, mod=MOD):
    """Modular subtraction"""
    return (a - b + mod) % mod

def mod_mul(a, b, mod=MOD):
    """Modular multiplication"""
    return (a * b) % mod

def mod_power(base, exp, mod=MOD):
    """
    Modular exponentiation: base^exp mod mod
    Time: O(log exp), Space: O(1)
    """
    result = 1
    base %= mod
    
    while exp > 0:
        if exp % 2 == 1:
            result = (result * base) % mod
        base = (base * base) % mod
        exp //= 2
    
    return result

def mod_divide(a, b, mod=MOD):
    """Modular division"""
    return (a * mod_inverse(b, mod)) % mod
```

### 3.2 Applications

**Large Factorial**:
```python
def factorial_mod(n, mod=MOD):
    """
    Calculate n! mod mod
    Time: O(n), Space: O(1)
    """
    result = 1
    for i in range(2, n + 1):
        result = (result * i) % mod
    return result
```

---

## 4. Combinatorics

### 4.1 Combinations and Permutations

```python
def nCr(n, r, mod=MOD):
    """
    Calculate C(n, r) = n! / (r! * (n-r)!)
    Time: O(n), Space: O(1)
    """
    if r > n or r < 0:
        return 0
    if r == 0 or r == n:
        return 1
    
    # Use: C(n, r) = C(n, n-r)
    if r > n - r:
        r = n - r
    
    numerator = 1
    denominator = 1
    
    for i in range(r):
        numerator = (numerator * (n - i)) % mod
        denominator = (denominator * (i + 1)) % mod
    
    return (numerator * mod_inverse(denominator, mod)) % mod

def nPr(n, r, mod=MOD):
    """
    Calculate P(n, r) = n! / (n-r)!
    Time: O(r), Space: O(1)
    """
    if r > n:
        return 0
    
    result = 1
    for i in range(r):
        result = (result * (n - i)) % mod
    
    return result

# Precompute factorials
def precompute_factorials(n, mod=MOD):
    """Precompute factorials up to n"""
    fact = [1] * (n + 1)
    for i in range(1, n + 1):
        fact[i] = (fact[i - 1] * i) % mod
    return fact

def nCr_precomputed(n, r, fact, mod=MOD):
    """Using precomputed factorials"""
    if r > n or r < 0:
        return 0
    return (fact[n] * mod_inverse(fact[r] * fact[n - r] % mod, mod)) % mod
```

### 4.2 Catalan Numbers

```python
def catalan_number(n, mod=MOD):
    """
    Calculate nth Catalan number
    C(n) = C(2n, n) / (n + 1)
    Time: O(n), Space: O(1)
    """
    return nCr(2 * n, n, mod) * mod_inverse(n + 1, mod) % mod

def catalan_numbers(n, mod=MOD):
    """Generate first n Catalan numbers"""
    catalan = [0] * (n + 1)
    catalan[0] = 1
    
    for i in range(1, n + 1):
        catalan[i] = 0
        for j in range(i):
            catalan[i] = (catalan[i] + catalan[j] * catalan[i - j - 1]) % mod
    
    return catalan
```

---

## 5. Bit Manipulation

### 5.1 Basic Operations

```python
# Set bit
def set_bit(n, i):
    """Set ith bit"""
    return n | (1 << i)

# Clear bit
def clear_bit(n, i):
    """Clear ith bit"""
    return n & ~(1 << i)

# Toggle bit
def toggle_bit(n, i):
    """Toggle ith bit"""
    return n ^ (1 << i)

# Check bit
def check_bit(n, i):
    """Check if ith bit is set"""
    return (n >> i) & 1

# Count set bits
def count_set_bits(n):
    """Count number of set bits (Brian Kernighan's algorithm)"""
    count = 0
    while n:
        n &= n - 1
        count += 1
    return count

# Power of 2
def is_power_of_2(n):
    """Check if n is power of 2"""
    return n > 0 and (n & (n - 1)) == 0

# Get lowest set bit
def lowest_set_bit(n):
    """Get position of lowest set bit"""
    return n & -n
```

### 5.2 Advanced Bit Manipulation

**Single Number**:
```python
def single_number(nums):
    """
    Find number that appears once (others appear twice)
    Time: O(n), Space: O(1)
    """
    result = 0
    for num in nums:
        result ^= num
    return result

def single_number_ii(nums):
    """
    Find number that appears once (others appear three times)
    Time: O(n), Space: O(1)
    """
    ones = twos = 0
    for num in nums:
        ones = (ones ^ num) & ~twos
        twos = (twos ^ num) & ~ones
    return ones
```

**Subset Generation**:
```python
def subsets_bitmask(nums):
    """
    Generate all subsets using bit manipulation
    Time: O(2^n * n), Space: O(2^n)
    """
    n = len(nums)
    result = []
    
    for mask in range(1 << n):
        subset = []
        for i in range(n):
            if mask & (1 << i):
                subset.append(nums[i])
        result.append(subset)
    
    return result
```

---

## 6. Practice Problems

### Problem Set 1: Math

1. **Pow(x, n)**
```python
def my_pow(x, n):
    """
    Calculate x^n
    Time: O(log n), Space: O(1)
    """
    if n < 0:
        x = 1 / x
        n = -n
    
    result = 1
    while n > 0:
        if n % 2 == 1:
            result *= x
        x *= x
        n //= 2
    
    return result
```

2. **Sqrt(x)**
```python
def my_sqrt(x):
    """
    Integer square root using binary search
    Time: O(log x), Space: O(1)
    """
    if x < 2:
        return x
    
    left, right = 1, x // 2
    
    while left <= right:
        mid = (left + right) // 2
        square = mid * mid
        
        if square == x:
            return mid
        elif square < x:
            left = mid + 1
        else:
            right = mid - 1
    
    return right
```

---

## Summary of Part 9

### Math Topics

1. **Prime Numbers**: Sieve, factorization
2. **GCD/LCM**: Euclidean algorithm
3. **Modular Arithmetic**: Operations, inverse
4. **Combinatorics**: nCr, nPr, Catalan
5. **Bit Manipulation**: Operations, tricks

### Next Steps

**Part 10** will cover:
- Problem-Solving Strategies
- Common Patterns
- Competitive Programming Tips
- Practice Roadmap

---

**Master these math concepts for competitive programming!**

