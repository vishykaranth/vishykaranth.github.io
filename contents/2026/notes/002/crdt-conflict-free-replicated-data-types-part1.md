# CRDT (Conflict-free Replicated Data Types) - Part 1: Fundamentals & State-Based CRDTs

## Table of Contents
1. [Introduction to CRDTs](#introduction-to-crdts)
2. [Why CRDTs?](#why-crdts)
3. [Core Concepts](#core-concepts)
4. [Types of CRDTs](#types-of-crdts)
5. [State-Based CRDTs (CvRDTs)](#state-based-crdts-cvrdts)
6. [G-Counter (Grow-Only Counter)](#g-counter-grow-only-counter)
7. [PN-Counter (Positive-Negative Counter)](#pn-counter-positive-negative-counter)
8. [G-Set (Grow-Only Set)](#g-set-grow-only-set)
9. [2P-Set (Two-Phase Set)](#2p-set-two-phase-set)
10. [Summary of Part 1](#summary-of-part-1)

---

## Introduction to CRDTs

### What are CRDTs?

**CRDT (Conflict-free Replicated Data Types)** are data structures that can be replicated across multiple nodes in a distributed system. They guarantee that replicas will eventually converge to the same state, even when operations are applied in different orders or when nodes are temporarily disconnected.

### Key Characteristics

1. **Commutative Operations**: Operations can be applied in any order
2. **Idempotent Operations**: Applying the same operation multiple times has the same effect as applying it once
3. **Associative Operations**: Operations can be grouped in any way
4. **No Coordination Required**: Nodes don't need to coordinate to achieve consistency
5. **Eventual Consistency**: All replicas converge to the same state

### Why "Conflict-Free"?

CRDTs are designed so that **conflicts cannot occur**. The data structure itself ensures that concurrent operations can be merged deterministically without requiring conflict resolution.

---

## Why CRDTs?

### The Problem with Traditional Approaches

#### Problem 1: Distributed Consensus is Expensive

```java
// Traditional approach: Need consensus for every operation
public class TraditionalCounter {
    private int value;
    
    public synchronized void increment() {
        // Requires coordination with all nodes
        // Expensive: O(n) messages for n nodes
        value++;
        broadcastToAllNodes(value);
        waitForAcknowledgment();
    }
}
```

**Issues:**
- Requires coordination
- High latency
- Network partitions cause unavailability
- Expensive (many messages)

#### Problem 2: Conflict Resolution is Complex

```java
// Traditional approach: Need to resolve conflicts
public class TraditionalSet {
    private Set<String> items = new HashSet<>();
    
    public void add(String item) {
        // May conflict with concurrent add/remove
        // Need complex conflict resolution
        items.add(item);
    }
    
    public void remove(String item) {
        // May conflict with concurrent operations
        items.remove(item);
    }
}
```

**Issues:**
- Complex conflict resolution logic
- May lose user intent
- Hard to implement correctly
- Requires additional metadata

### CRDT Solution

```java
// CRDT approach: No conflicts, automatic merging
public class CRDTSet {
    private Map<String, Boolean> added = new HashMap<>();
    private Set<String> removed = new HashSet<>();
    
    public void add(String item) {
        added.put(item, true);
        // No coordination needed!
    }
    
    public void remove(String item) {
        removed.add(item);
        // No coordination needed!
    }
    
    public Set<String> lookup() {
        return added.keySet().stream()
            .filter(item -> !removed.contains(item))
            .collect(Collectors.toSet());
    }
    
    // Automatic merging - no conflicts!
    public void merge(CRDTSet other) {
        this.added.putAll(other.added);
        this.removed.addAll(other.removed);
    }
}
```

**Benefits:**
- ✅ No coordination needed
- ✅ Automatic conflict resolution
- ✅ Works offline
- ✅ Low latency
- ✅ Simple implementation

---

## Core Concepts

### Mathematical Properties

For CRDTs to work, operations must satisfy certain mathematical properties:

#### 1. Commutativity

```java
// Operations commute: a ∘ b = b ∘ a
// Order doesn't matter

// Example: Counter increment
increment() ∘ increment() = increment() ∘ increment()
// Both result in +2
```

#### 2. Associativity

```java
// Operations associate: (a ∘ b) ∘ c = a ∘ (b ∘ c)
// Grouping doesn't matter

// Example: Set union
(A ∪ B) ∪ C = A ∪ (B ∪ C)
```

#### 3. Idempotence

```java
// Operations are idempotent: a ∘ a = a
// Applying twice is same as applying once

// Example: Set add
add(x) ∘ add(x) = add(x)
```

### State vs Operation-Based

**State-Based (CvRDT - Convergent Replicated Data Type):**
- Replicas exchange their entire state
- Merge function combines states
- Requires idempotent merge

**Operation-Based (CvRDT - Commutative Replicated Data Type):**
- Replicas exchange operations
- Operations must be commutative
- More efficient (only send changes)

---

## Types of CRDTs

### Classification

1. **State-Based CRDTs (CvRDTs)**
   - G-Counter (Grow-Only Counter)
   - PN-Counter (Positive-Negative Counter)
   - G-Set (Grow-Only Set)
   - 2P-Set (Two-Phase Set)
   - LWW-Register (Last-Write-Wins Register)

2. **Operation-Based CRDTs (CmRDTs)**
   - OR-Set (Observed-Remove Set)
   - Counter (Operation-based)
   - Map (Operation-based)

3. **Hybrid CRDTs**
   - Combine state and operation-based approaches

---

## State-Based CRDTs (CvRDTs)

### How State-Based CRDTs Work

1. **Each replica maintains full state**
2. **Periodically exchange states with other replicas**
3. **Merge states using merge function**
4. **Merge must be:**
   - Commutative: `merge(a, b) = merge(b, a)`
   - Associative: `merge(merge(a, b), c) = merge(a, merge(b, c))`
   - Idempotent: `merge(a, a) = a`

### Merge Function Properties

```java
public interface Mergeable<T> {
    /**
     * Merge this state with another state
     * Must be commutative, associative, and idempotent
     */
    T merge(T other);
}
```

---

## G-Counter (Grow-Only Counter)

### Definition

A **G-Counter** (Grow-Only Counter) is a counter that can only be incremented. It cannot be decremented, which ensures all operations are commutative.

### How It Works

- Each node maintains its own counter
- Total value = sum of all node counters
- Increments are commutative (order doesn't matter)

### Implementation

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Objects;

/**
 * G-Counter: Grow-Only Counter
 * Can only increment, never decrement
 * Operations are commutative
 */
public class GCounter implements Mergeable<GCounter> {
    private final String nodeId;
    private final Map<String, Long> counters; // Node ID -> counter value
    
    public GCounter(String nodeId) {
        this.nodeId = nodeId;
        this.counters = new HashMap<>();
        this.counters.put(nodeId, 0L);
    }
    
    /**
     * Increment counter for this node
     */
    public void increment() {
        increment(1);
    }
    
    /**
     * Increment counter by delta
     */
    public void increment(long delta) {
        if (delta < 0) {
            throw new IllegalArgumentException("G-Counter can only grow");
        }
        counters.put(nodeId, counters.getOrDefault(nodeId, 0L) + delta);
    }
    
    /**
     * Get total value (sum of all node counters)
     */
    public long value() {
        return counters.values().stream()
            .mapToLong(Long::longValue)
            .sum();
    }
    
    /**
     * Merge with another G-Counter
     * Takes maximum value for each node
     */
    @Override
    public GCounter merge(GCounter other) {
        GCounter merged = new GCounter(this.nodeId);
        
        // Collect all node IDs
        merged.counters.putAll(this.counters);
        
        // Merge: take maximum for each node
        for (Map.Entry<String, Long> entry : other.counters.entrySet()) {
            String nodeId = entry.getKey();
            Long otherValue = entry.getValue();
            Long currentValue = merged.counters.getOrDefault(nodeId, 0L);
            merged.counters.put(nodeId, Math.max(currentValue, otherValue));
        }
        
        return merged;
    }
    
    /**
     * Create a copy of this counter
     */
    public GCounter copy() {
        GCounter copy = new GCounter(this.nodeId);
        copy.counters.putAll(this.counters);
        return copy;
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        GCounter gCounter = (GCounter) o;
        return Objects.equals(counters, gCounter.counters);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(counters);
    }
    
    @Override
    public String toString() {
        return "GCounter{value=" + value() + ", counters=" + counters + "}";
    }
}
```

### Usage Example

```java
public class GCounterExample {
    public static void main(String[] args) {
        // Node A
        GCounter counterA = new GCounter("node-a");
        counterA.increment(5);
        counterA.increment(3);
        System.out.println("Node A: " + counterA.value()); // 8
        
        // Node B
        GCounter counterB = new GCounter("node-b");
        counterB.increment(10);
        counterB.increment(2);
        System.out.println("Node B: " + counterB.value()); // 12
        
        // Node C
        GCounter counterC = new GCounter("node-c");
        counterC.increment(7);
        System.out.println("Node C: " + counterC.value()); // 7
        
        // Merge A and B
        GCounter mergedAB = counterA.merge(counterB);
        System.out.println("Merged A+B: " + mergedAB.value()); // 20
        
        // Merge with C
        GCounter finalMerged = mergedAB.merge(counterC);
        System.out.println("Final merged: " + finalMerged.value()); // 27
        
        // Verify: All nodes see same value after merge
        GCounter mergedAC = counterA.merge(counterC);
        GCounter mergedBC = counterB.merge(counterC);
        GCounter mergedABC = mergedAC.merge(counterB);
        
        System.out.println("All merges result in: " + mergedABC.value()); // 27
        assert finalMerged.value() == mergedABC.value(); // ✓
    }
}
```

### Visual Example

```
Initial State:
Node A: {node-a: 0}
Node B: {node-b: 0}
Node C: {node-c: 0}

After Operations:
Node A: {node-a: 8}  (incremented 5, then 3)
Node B: {node-b: 12} (incremented 10, then 2)
Node C: {node-c: 7}  (incremented 7)

After Merge:
All Nodes: {node-a: 8, node-b: 12, node-c: 7}
Total Value: 8 + 12 + 7 = 27
```

### Properties Verification

```java
public class GCounterPropertiesTest {
    
    @Test
    public void testCommutativity() {
        // merge(a, b) = merge(b, a)
        GCounter a = new GCounter("node-a");
        a.increment(5);
        
        GCounter b = new GCounter("node-b");
        b.increment(10);
        
        GCounter mergeAB = a.merge(b);
        GCounter mergeBA = b.merge(a);
        
        assert mergeAB.value() == mergeBA.value(); // ✓ Commutative
    }
    
    @Test
    public void testAssociativity() {
        // merge(merge(a, b), c) = merge(a, merge(b, c))
        GCounter a = new GCounter("node-a");
        a.increment(5);
        
        GCounter b = new GCounter("node-b");
        b.increment(10);
        
        GCounter c = new GCounter("node-c");
        c.increment(7);
        
        GCounter mergeAB_then_C = a.merge(b).merge(c);
        GCounter mergeA_then_BC = a.merge(b.merge(c));
        
        assert mergeAB_then_C.value() == mergeA_then_BC.value(); // ✓ Associative
    }
    
    @Test
    public void testIdempotence() {
        // merge(a, a) = a
        GCounter a = new GCounter("node-a");
        a.increment(5);
        
        GCounter merged = a.merge(a);
        
        assert merged.value() == a.value(); // ✓ Idempotent
    }
}
```

---

## PN-Counter (Positive-Negative Counter)

### Definition

A **PN-Counter** (Positive-Negative Counter) can both increment and decrement. It uses two G-Counters: one for increments (P) and one for decrements (N). Value = P - N.

### How It Works

- Maintains two G-Counters: `p` (positive) and `n` (negative)
- Increment: increment `p`
- Decrement: increment `n`
- Value: `p.value() - n.value()`
- Merge: merge both G-Counters separately

### Implementation

```java
/**
 * PN-Counter: Positive-Negative Counter
 * Can increment and decrement
 * Uses two G-Counters internally
 */
public class PNCounter implements Mergeable<PNCounter> {
    private final GCounter p; // Positive counter (increments)
    private final GCounter n; // Negative counter (decrements)
    
    public PNCounter(String nodeId) {
        this.p = new GCounter(nodeId + "-p");
        this.n = new GCounter(nodeId + "-n");
    }
    
    /**
     * Increment counter
     */
    public void increment() {
        increment(1);
    }
    
    /**
     * Increment by delta
     */
    public void increment(long delta) {
        if (delta < 0) {
            throw new IllegalArgumentException("Delta must be positive");
        }
        p.increment(delta);
    }
    
    /**
     * Decrement counter
     */
    public void decrement() {
        decrement(1);
    }
    
    /**
     * Decrement by delta
     */
    public void decrement(long delta) {
        if (delta < 0) {
            throw new IllegalArgumentException("Delta must be positive");
        }
        n.increment(delta); // Decrement = increment negative counter
    }
    
    /**
     * Get current value
     */
    public long value() {
        return p.value() - n.value();
    }
    
    /**
     * Merge with another PN-Counter
     */
    @Override
    public PNCounter merge(PNCounter other) {
        PNCounter merged = new PNCounter(this.p.nodeId.replace("-p", ""));
        merged.p.counters.putAll(this.p.merge(other.p).counters);
        merged.n.counters.putAll(this.n.merge(other.n).counters);
        return merged;
    }
    
    public PNCounter copy() {
        PNCounter copy = new PNCounter(this.p.nodeId.replace("-p", ""));
        copy.p.counters.putAll(this.p.counters);
        copy.n.counters.putAll(this.n.counters);
        return copy;
    }
    
    @Override
    public String toString() {
        return "PNCounter{value=" + value() + ", p=" + p.value() + ", n=" + n.value() + "}";
    }
}
```

### Usage Example

```java
public class PNCounterExample {
    public static void main(String[] args) {
        // Node A
        PNCounter counterA = new PNCounter("node-a");
        counterA.increment(10);
        counterA.increment(5);
        counterA.decrement(3);
        System.out.println("Node A: " + counterA.value()); // 12
        
        // Node B
        PNCounter counterB = new PNCounter("node-b");
        counterB.increment(20);
        counterB.decrement(8);
        counterB.decrement(2);
        System.out.println("Node B: " + counterB.value()); // 10
        
        // Merge
        PNCounter merged = counterA.merge(counterB);
        System.out.println("Merged: " + merged.value()); // 22
        
        // Verify
        // A: +15, -3 = 12
        // B: +20, -10 = 10
        // Merged: +35, -13 = 22 ✓
    }
}
```

### Why PN-Counter Works

```
Node A Operations:
  increment(10)  → p: +10
  increment(5)    → p: +15
  decrement(3)    → n: +3
  Value: 15 - 3 = 12

Node B Operations:
  increment(20)   → p: +20
  decrement(8)    → n: +8
  decrement(2)    → n: +10
  Value: 20 - 10 = 10

After Merge:
  p: max(15, 20) = 20 (but actually sum: 15 + 20 = 35)
  n: max(3, 10) = 10 (but actually sum: 3 + 10 = 13)
  
Wait, that's wrong! Let me fix the implementation...

Actually, PN-Counter uses G-Counters per node, so:
Node A: {node-a-p: 15, node-a-n: 3}
Node B: {node-b-p: 20, node-b-n: 10}

After merge:
{node-a-p: 15, node-a-n: 3, node-b-p: 20, node-b-n: 10}
Value: (15 + 20) - (3 + 10) = 35 - 13 = 22 ✓
```

---

## G-Set (Grow-Only Set)

### Definition

A **G-Set** (Grow-Only Set) is a set that can only add elements, never remove them. This ensures all operations are commutative.

### How It Works

- Elements can only be added
- Merge: union of all sets
- Simple but limited (no removal)

### Implementation

```java
import java.util.HashSet;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * G-Set: Grow-Only Set
 * Can only add elements, never remove
 * Operations are commutative
 */
public class GSet<T> implements Mergeable<GSet<T>> {
    private final Set<T> elements;
    
    public GSet() {
        this.elements = new HashSet<>();
    }
    
    private GSet(Set<T> elements) {
        this.elements = new HashSet<>(elements);
    }
    
    /**
     * Add element to set
     */
    public void add(T element) {
        elements.add(element);
    }
    
    /**
     * Add multiple elements
     */
    public void addAll(Set<T> elements) {
        this.elements.addAll(elements);
    }
    
    /**
     * Check if element exists
     */
    public boolean contains(T element) {
        return elements.contains(element);
    }
    
    /**
     * Get all elements
     */
    public Set<T> elements() {
        return new HashSet<>(elements);
    }
    
    /**
     * Get size
     */
    public int size() {
        return elements.size();
    }
    
    /**
     * Merge with another G-Set
     * Union of both sets
     */
    @Override
    public GSet<T> merge(GSet<T> other) {
        GSet<T> merged = new GSet<>(this.elements);
        merged.elements.addAll(other.elements);
        return merged;
    }
    
    /**
     * Create a copy
     */
    public GSet<T> copy() {
        return new GSet<>(this.elements);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        GSet<?> gSet = (GSet<?>) o;
        return Objects.equals(elements, gSet.elements);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(elements);
    }
    
    @Override
    public String toString() {
        return "GSet" + elements;
    }
}
```

### Usage Example

```java
public class GSetExample {
    public static void main(String[] args) {
        // Node A
        GSet<String> setA = new GSet<>();
        setA.add("apple");
        setA.add("banana");
        System.out.println("Set A: " + setA); // [apple, banana]
        
        // Node B
        GSet<String> setB = new GSet<>();
        setB.add("banana");
        setB.add("cherry");
        System.out.println("Set B: " + setB); // [banana, cherry]
        
        // Node C
        GSet<String> setC = new GSet<>();
        setC.add("date");
        System.out.println("Set C: " + setC); // [date]
        
        // Merge
        GSet<String> merged = setA.merge(setB).merge(setC);
        System.out.println("Merged: " + merged); // [apple, banana, cherry, date]
        
        // All nodes eventually see same set
        assert merged.contains("apple");
        assert merged.contains("banana");
        assert merged.contains("cherry");
        assert merged.contains("date");
    }
}
```

### Limitations

```java
// ❌ G-Set cannot remove elements
GSet<String> set = new GSet<>();
set.add("apple");
// set.remove("apple"); // Not possible!

// Workaround: Use 2P-Set or OR-Set (covered in Part 2)
```

---

## 2P-Set (Two-Phase Set)

### Definition

A **2P-Set** (Two-Phase Set) allows both addition and removal, but with a restriction: once an element is removed, it can never be added again. It uses two G-Sets: one for added elements (A) and one for removed elements (R).

### How It Works

- **Add Set (A)**: G-Set of added elements
- **Remove Set (R)**: G-Set of removed elements
- **Lookup**: Element exists if it's in A but not in R
- **Add**: Add to A (if not in R)
- **Remove**: Add to R
- **Merge**: Merge both G-Sets separately

### Implementation

```java
/**
 * 2P-Set: Two-Phase Set
 * Can add and remove, but removed elements cannot be re-added
 * Uses two G-Sets: added and removed
 */
public class TwoPSet<T> implements Mergeable<TwoPSet<T>> {
    private final GSet<T> added;   // Elements that have been added
    private final GSet<T> removed; // Elements that have been removed
    
    public TwoPSet() {
        this.added = new GSet<>();
        this.removed = new GSet<>();
    }
    
    private TwoPSet(GSet<T> added, GSet<T> removed) {
        this.added = added;
        this.removed = removed;
    }
    
    /**
     * Add element to set
     * Only works if element hasn't been removed
     */
    public void add(T element) {
        if (!removed.contains(element)) {
            added.add(element);
        }
        // If already removed, silently ignore (element is "tombstoned")
    }
    
    /**
     * Remove element from set
     */
    public void remove(T element) {
        removed.add(element);
    }
    
    /**
     * Check if element exists
     */
    public boolean contains(T element) {
        return added.contains(element) && !removed.contains(element);
    }
    
    /**
     * Get all elements (added but not removed)
     */
    public Set<T> elements() {
        return added.elements().stream()
            .filter(e -> !removed.contains(e))
            .collect(Collectors.toSet());
    }
    
    /**
     * Get size
     */
    public int size() {
        return (int) added.elements().stream()
            .filter(e -> !removed.contains(e))
            .count();
    }
    
    /**
     * Merge with another 2P-Set
     */
    @Override
    public TwoPSet<T> merge(TwoPSet<T> other) {
        GSet<T> mergedAdded = this.added.merge(other.added);
        GSet<T> mergedRemoved = this.removed.merge(other.removed);
        return new TwoPSet<>(mergedAdded, mergedRemoved);
    }
    
    /**
     * Create a copy
     */
    public TwoPSet<T> copy() {
        return new TwoPSet<>(this.added.copy(), this.removed.copy());
    }
    
    @Override
    public String toString() {
        return "TwoPSet" + elements();
    }
}
```

### Usage Example

```java
public class TwoPSetExample {
    public static void main(String[] args) {
        // Node A
        TwoPSet<String> setA = new TwoPSet<>();
        setA.add("apple");
        setA.add("banana");
        setA.remove("apple");
        System.out.println("Set A: " + setA); // [banana]
        
        // Node B
        TwoPSet<String> setB = new TwoPSet<>();
        setB.add("banana");
        setB.add("cherry");
        System.out.println("Set B: " + setB); // [banana, cherry]
        
        // Node A tries to re-add "apple" (was removed)
        setA.add("apple"); // Silently ignored - cannot re-add removed element
        System.out.println("Set A after re-add: " + setA); // Still [banana]
        
        // Merge
        TwoPSet<String> merged = setA.merge(setB);
        System.out.println("Merged: " + merged); // [banana, cherry]
        // "apple" is not in merged set because it was removed in A
    }
}
```

### Limitations

```java
// ❌ 2P-Set limitation: Cannot re-add removed elements
TwoPSet<String> set = new TwoPSet<>();
set.add("apple");
set.remove("apple");
set.add("apple"); // Ignored! Element is "tombstoned"

// This is a limitation - use OR-Set (Part 2) if you need re-addition
```

### Why 2P-Set Works

```
Node A Operations:
  add("apple")    → added: {apple}
  add("banana")   → added: {apple, banana}
  remove("apple") → removed: {apple}
  Result: {banana}

Node B Operations:
  add("banana")   → added: {banana}
  add("cherry")   → added: {banana, cherry}
  Result: {banana, cherry}

After Merge:
  added: {apple, banana, cherry}
  removed: {apple}
  Result: {banana, cherry} ✓

Even if operations arrive in different order, result is same!
```

---

## Summary of Part 1

### Key Concepts Covered

1. **CRDTs** are conflict-free replicated data types
2. **State-Based CRDTs (CvRDTs)** exchange full state
3. **Merge function** must be commutative, associative, and idempotent
4. **G-Counter**: Grow-only counter (increment only)
5. **PN-Counter**: Positive-negative counter (increment and decrement)
6. **G-Set**: Grow-only set (add only)
7. **2P-Set**: Two-phase set (add and remove, but no re-addition)

### Properties Summary

| CRDT | Add | Remove | Re-add | Merge Complexity |
|------|-----|--------|--------|------------------|
| **G-Counter** | ✅ | ❌ | N/A | O(n) nodes |
| **PN-Counter** | ✅ | ✅ | N/A | O(n) nodes |
| **G-Set** | ✅ | ❌ | ✅ | O(m) elements |
| **2P-Set** | ✅ | ✅ | ❌ | O(m) elements |

### When to Use State-Based CRDTs

✅ **Use when:**
- Simple data structures needed
- Can tolerate sending full state
- Network bandwidth is not a concern
- Need guaranteed convergence

❌ **Don't use when:**
- Large state size
- Need efficient network usage
- Need element re-addition (use OR-Set)
- Need more sophisticated operations

### Next: Part 2

In Part 2, we'll cover:
- Operation-Based CRDTs (CmRDTs)
- OR-Set (Observed-Remove Set)
- LWW-Register (Last-Write-Wins)
- CRDTs for Text Editing
- Real-World Applications
- Advanced Patterns

---

*Part 1 covers the fundamentals of CRDTs and state-based implementations. These form the foundation for understanding more advanced CRDT types covered in Part 2.*



