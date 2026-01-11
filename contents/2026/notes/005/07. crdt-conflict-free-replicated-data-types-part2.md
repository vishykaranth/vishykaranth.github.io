# CRDT (Conflict-free Replicated Data Types) - Part 2: Operation-Based CRDTs & Advanced Topics

## Table of Contents
1. [Operation-Based CRDTs (CmRDTs)](#operation-based-crdts-cmrdts)
2. [OR-Set (Observed-Remove Set)](#or-set-observed-remove-set)
3. [LWW-Register (Last-Write-Wins Register)](#lww-register-last-write-wins-register)
4. [LWW-Map (Last-Write-Wins Map)](#lww-map-last-write-wins-map)
5. [CRDTs for Text Editing](#crdts-for-text-editing)
6. [Real-World Applications](#real-world-applications)
7. [Advanced Patterns](#advanced-patterns)
8. [Performance Considerations](#performance-considerations)
9. [Best Practices](#best-practices)
10. [Summary](#summary)

---

## Operation-Based CRDTs (CmRDTs)

### How Operation-Based CRDTs Work

**Operation-Based CRDTs (CmRDTs - Commutative Replicated Data Types)** exchange operations instead of full state. Operations must be:

1. **Commutative**: `op1 ∘ op2 = op2 ∘ op1`
2. **Delivered exactly once**: Operations are idempotent or delivery is guaranteed
3. **Causally ordered**: Operations are delivered in causal order

### State-Based vs Operation-Based

| Aspect | State-Based (CvRDT) | Operation-Based (CmRDT) |
|--------|---------------------|-------------------------|
| **Exchange** | Full state | Operations only |
| **Network** | More bandwidth | Less bandwidth |
| **Complexity** | Simpler merge | More complex delivery |
| **Use Case** | Small state | Large state, frequent updates |

### Operation Delivery Requirements

```java
public interface OperationDelivery {
    /**
     * Operations must be delivered:
     * 1. At least once (or exactly once with idempotency)
     * 2. In causal order
     * 3. Eventually to all replicas
     */
    void deliver(Operation op);
}
```

---

## OR-Set (Observed-Remove Set)

### Definition

An **OR-Set** (Observed-Remove Set) allows both addition and removal, and **can re-add removed elements**. It uses unique tags to track which add operation created each element.

### How It Works

- Each element is tagged with a unique identifier when added
- Remove operations remove all tags for that element
- Lookup: Element exists if it has at least one tag
- Re-add: Creates new tag (element can be re-added)

### Implementation

```java
import java.util.*;
import java.util.stream.Collectors;

/**
 * OR-Set: Observed-Remove Set
 * Can add, remove, and re-add elements
 * Uses unique tags to track elements
 */
public class ORSet<T> implements Mergeable<ORSet<T>> {
    // Element -> Set of unique tags
    private final Map<T, Set<UUID>> elementTags;
    // Tag -> Element (reverse mapping for efficient lookup)
    private final Map<UUID, T> tagToElement;
    
    public ORSet() {
        this.elementTags = new HashMap<>();
        this.tagToElement = new HashMap<>();
    }
    
    private ORSet(Map<T, Set<UUID>> elementTags, Map<UUID, T> tagToElement) {
        this.elementTags = new HashMap<>(elementTags);
        this.tagToElement = new HashMap<>(tagToElement);
    }
    
    /**
     * Add element to set
     * Creates a new unique tag for this add operation
     */
    public UUID add(T element) {
        UUID tag = UUID.randomUUID();
        elementTags.computeIfAbsent(element, k -> new HashSet<>()).add(tag);
        tagToElement.put(tag, element);
        return tag;
    }
    
    /**
     * Remove element from set
     * Removes all tags for this element
     */
    public void remove(T element) {
        Set<UUID> tags = elementTags.remove(element);
        if (tags != null) {
            tags.forEach(tagToElement::remove);
        }
    }
    
    /**
     * Remove element by tag (for operation-based delivery)
     */
    public void removeTag(UUID tag) {
        T element = tagToElement.remove(tag);
        if (element != null) {
            Set<UUID> tags = elementTags.get(element);
            if (tags != null) {
                tags.remove(tag);
                if (tags.isEmpty()) {
                    elementTags.remove(element);
                }
            }
        }
    }
    
    /**
     * Check if element exists
     */
    public boolean contains(T element) {
        Set<UUID> tags = elementTags.get(element);
        return tags != null && !tags.isEmpty();
    }
    
    /**
     * Get all elements
     */
    public Set<T> elements() {
        return elementTags.keySet().stream()
            .filter(element -> contains(element))
            .collect(Collectors.toSet());
    }
    
    /**
     * Get all tags for an element
     */
    public Set<UUID> getTags(T element) {
        return new HashSet<>(elementTags.getOrDefault(element, Collections.emptySet()));
    }
    
    /**
     * Get size
     */
    public int size() {
        return (int) elementTags.values().stream()
            .filter(tags -> !tags.isEmpty())
            .count();
    }
    
    /**
     * Merge with another OR-Set
     * Union of all tags
     */
    @Override
    public ORSet<T> merge(ORSet<T> other) {
        ORSet<T> merged = new ORSet<>();
        
        // Copy all tags from this set
        for (Map.Entry<T, Set<UUID>> entry : this.elementTags.entrySet()) {
            T element = entry.getKey();
            Set<UUID> tags = entry.getValue();
            merged.elementTags.put(element, new HashSet<>(tags));
            tags.forEach(tag -> merged.tagToElement.put(tag, element));
        }
        
        // Merge tags from other set
        for (Map.Entry<T, Set<UUID>> entry : other.elementTags.entrySet()) {
            T element = entry.getKey();
            Set<UUID> otherTags = entry.getValue();
            
            Set<UUID> mergedTags = merged.elementTags.computeIfAbsent(
                element, k -> new HashSet<>());
            mergedTags.addAll(otherTags);
            
            otherTags.forEach(tag -> merged.tagToElement.put(tag, element));
        }
        
        return merged;
    }
    
    /**
     * Create a copy
     */
    public ORSet<T> copy() {
        return new ORSet<>(
            new HashMap<>(this.elementTags),
            new HashMap<>(this.tagToElement)
        );
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        ORSet<?> orSet = (ORSet<?>) o;
        return Objects.equals(elementTags, orSet.elementTags);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(elementTags);
    }
    
    @Override
    public String toString() {
        return "ORSet" + elements();
    }
}
```

### Usage Example

```java
public class ORSetExample {
    public static void main(String[] args) {
        // Node A
        ORSet<String> setA = new ORSet<>();
        UUID tag1 = setA.add("apple");
        UUID tag2 = setA.add("banana");
        System.out.println("Set A: " + setA); // [apple, banana]
        
        // Node B
        ORSet<String> setB = new ORSet<>();
        UUID tag3 = setB.add("banana"); // Different tag for same element
        UUID tag4 = setB.add("cherry");
        System.out.println("Set B: " + setB); // [banana, cherry]
        
        // Node A removes "apple"
        setA.remove("apple");
        System.out.println("Set A after remove: " + setA); // [banana]
        
        // Node A can re-add "apple" (creates new tag)
        UUID tag5 = setA.add("apple");
        System.out.println("Set A after re-add: " + setA); // [banana, apple]
        
        // Merge
        ORSet<String> merged = setA.merge(setB);
        System.out.println("Merged: " + merged); // [banana, cherry, apple]
        
        // "banana" appears in both sets with different tags
        // After merge, it has tags from both: {tag2, tag3}
        System.out.println("Banana tags: " + merged.getTags("banana")); // 2 tags
    }
}
```

### Operation-Based OR-Set

```java
/**
 * Operation-based OR-Set
 * Operations are sent instead of full state
 */
public class OperationBasedORSet<T> {
    private final ORSet<T> set;
    private final String nodeId;
    private final OperationQueue operationQueue;
    
    public OperationBasedORSet(String nodeId) {
        this.set = new ORSet<>();
        this.nodeId = nodeId;
        this.operationQueue = new OperationQueue();
    }
    
    public static class Operation {
        public enum Type { ADD, REMOVE }
        public Type type;
        public T element;
        public UUID tag; // For ADD operations
        public String nodeId;
        public long timestamp;
    }
    
    /**
     * Add element - creates operation
     */
    public void add(T element) {
        UUID tag = set.add(element);
        
        // Create operation
        Operation op = new Operation();
        op.type = Operation.Type.ADD;
        op.element = element;
        op.tag = tag;
        op.nodeId = this.nodeId;
        op.timestamp = System.currentTimeMillis();
        
        // Broadcast operation
        broadcastOperation(op);
    }
    
    /**
     * Remove element - creates operation
     */
    public void remove(T element) {
        Set<UUID> tags = set.getTags(element);
        set.remove(element);
        
        // Create remove operations for each tag
        for (UUID tag : tags) {
            Operation op = new Operation();
            op.type = Operation.Type.REMOVE;
            op.element = element;
            op.tag = tag;
            op.nodeId = this.nodeId;
            op.timestamp = System.currentTimeMillis();
            
            broadcastOperation(op);
        }
    }
    
    /**
     * Apply operation from another node
     */
    public void applyOperation(Operation op) {
        if (op.type == Operation.Type.ADD) {
            // Add with specific tag
            set.elementTags.computeIfAbsent(op.element, k -> new HashSet<>())
                .add(op.tag);
            set.tagToElement.put(op.tag, op.element);
        } else {
            // Remove specific tag
            set.removeTag(op.tag);
        }
    }
    
    private void broadcastOperation(Operation op) {
        // Send to all other nodes via network
        operationQueue.enqueue(op);
    }
}
```

### Why OR-Set Works

```
Node A Operations:
  add("apple") with tag1
  add("banana") with tag2
  remove("apple") → removes tag1
  add("apple") with tag5 (re-add creates new tag)

Node B Operations:
  add("banana") with tag3 (different tag!)
  add("cherry") with tag4

After Merge:
  "apple": {tag5} → exists ✓
  "banana": {tag2, tag3} → exists ✓ (both tags preserved)
  "cherry": {tag4} → exists ✓

Even if operations arrive in different order, result is same!
```

---

## LWW-Register (Last-Write-Wins Register)

### Definition

An **LWW-Register** (Last-Write-Wins Register) stores a single value with a timestamp. When merging, the value with the latest timestamp wins.

### How It Works

- Each write operation includes a timestamp
- Merge: Keep value with maximum timestamp
- If timestamps are equal, use tie-breaker (e.g., node ID)

### Implementation

```java
import java.util.Objects;

/**
 * LWW-Register: Last-Write-Wins Register
 * Stores a single value with timestamp
 * Latest write wins on merge
 */
public class LWWRegister<T> implements Mergeable<LWWRegister<T>> {
    private T value;
    private long timestamp;
    private String nodeId; // For tie-breaking
    
    public LWWRegister(String nodeId) {
        this.nodeId = nodeId;
        this.timestamp = 0L;
        this.value = null;
    }
    
    private LWWRegister(T value, long timestamp, String nodeId) {
        this.value = value;
        this.timestamp = timestamp;
        this.nodeId = nodeId;
    }
    
    /**
     * Write value with current timestamp
     */
    public void write(T value) {
        this.value = value;
        this.timestamp = System.currentTimeMillis();
    }
    
    /**
     * Write value with specific timestamp (for operation-based)
     */
    public void write(T value, long timestamp) {
        if (timestamp > this.timestamp) {
            this.value = value;
            this.timestamp = timestamp;
        }
    }
    
    /**
     * Read current value
     */
    public T read() {
        return value;
    }
    
    /**
     * Get timestamp
     */
    public long getTimestamp() {
        return timestamp;
    }
    
    /**
     * Merge with another LWW-Register
     * Keeps value with latest timestamp
     */
    @Override
    public LWWRegister<T> merge(LWWRegister<T> other) {
        if (other.timestamp > this.timestamp) {
            return new LWWRegister<>(other.value, other.timestamp, this.nodeId);
        } else if (other.timestamp < this.timestamp) {
            return new LWWRegister<>(this.value, this.timestamp, this.nodeId);
        } else {
            // Timestamps equal - use tie-breaker (node ID)
            if (other.nodeId.compareTo(this.nodeId) > 0) {
                return new LWWRegister<>(other.value, other.timestamp, this.nodeId);
            } else {
                return new LWWRegister<>(this.value, this.timestamp, this.nodeId);
            }
        }
        // Note: This creates a new instance, but we could modify in place
    }
    
    /**
     * Merge in place (more efficient)
     */
    public void mergeInPlace(LWWRegister<T> other) {
        if (other.timestamp > this.timestamp) {
            this.value = other.value;
            this.timestamp = other.timestamp;
        } else if (other.timestamp == this.timestamp) {
            // Tie-breaker
            if (other.nodeId.compareTo(this.nodeId) > 0) {
                this.value = other.value;
            }
        }
    }
    
    /**
     * Create a copy
     */
    public LWWRegister<T> copy() {
        return new LWWRegister<>(this.value, this.timestamp, this.nodeId);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        LWWRegister<?> that = (LWWRegister<?>) o;
        return timestamp == that.timestamp &&
               Objects.equals(value, that.value);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(value, timestamp);
    }
    
    @Override
    public String toString() {
        return "LWWRegister{value=" + value + ", timestamp=" + timestamp + "}";
    }
}
```

### Usage Example

```java
public class LWWRegisterExample {
    public static void main(String[] args) {
        // Node A
        LWWRegister<String> regA = new LWWRegister<>("node-a");
        regA.write("Hello");
        Thread.sleep(10); // Ensure different timestamp
        regA.write("World");
        System.out.println("Reg A: " + regA.read()); // "World"
        
        // Node B
        LWWRegister<String> regB = new LWWRegister<>("node-b");
        regB.write("Hi");
        System.out.println("Reg B: " + regB.read()); // "Hi"
        
        // Merge
        LWWRegister<String> merged = regA.merge(regB);
        System.out.println("Merged: " + merged.read()); // "World" (latest timestamp wins)
        
        // If timestamps are equal, node ID determines winner
        LWWRegister<String> regC = new LWWRegister<>("node-c");
        regC.write("Equal", 1000L);
        LWWRegister<String> regD = new LWWRegister<>("node-d");
        regD.write("Equal2", 1000L);
        
        LWWRegister<String> merged2 = regC.merge(regD);
        // Winner determined by node ID comparison
        System.out.println("Merged (equal time): " + merged2.read());
    }
}
```

### Limitations

```java
// ❌ LWW-Register can lose updates
LWWRegister<String> reg = new LWWRegister<>("node-a");
reg.write("Value1", 1000L);
reg.write("Value2", 999L); // Older timestamp - ignored!
// Result: "Value1" (Value2 is lost)

// Use vector clocks or other mechanisms if you need to preserve all updates
```

---

## LWW-Map (Last-Write-Wins Map)

### Definition

An **LWW-Map** is a map where each key-value pair is stored as an LWW-Register. This allows per-key last-write-wins semantics.

### Implementation

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * LWW-Map: Last-Write-Wins Map
 * Each key-value pair is an LWW-Register
 */
public class LWWMap<K, V> implements Mergeable<LWWMap<K, V>> {
    private final Map<K, LWWRegister<V>> registers;
    private final String nodeId;
    
    public LWWMap(String nodeId) {
        this.nodeId = nodeId;
        this.registers = new HashMap<>();
    }
    
    private LWWMap(Map<K, LWWRegister<V>> registers, String nodeId) {
        this.nodeId = nodeId;
        this.registers = new HashMap<>();
        registers.forEach((k, v) -> 
            this.registers.put(k, v.copy()));
    }
    
    /**
     * Put key-value pair
     */
    public void put(K key, V value) {
        LWWRegister<V> reg = registers.computeIfAbsent(
            key, k -> new LWWRegister<>(nodeId));
        reg.write(value);
    }
    
    /**
     * Get value for key
     */
    public V get(K key) {
        LWWRegister<V> reg = registers.get(key);
        return reg != null ? reg.read() : null;
    }
    
    /**
     * Remove key
     */
    public void remove(K key) {
        registers.remove(key);
    }
    
    /**
     * Check if key exists
     */
    public boolean containsKey(K key) {
        return registers.containsKey(key);
    }
    
    /**
     * Get all keys
     */
    public Set<K> keySet() {
        return registers.keySet();
    }
    
    /**
     * Get size
     */
    public int size() {
        return registers.size();
    }
    
    /**
     * Merge with another LWW-Map
     */
    @Override
    public LWWMap<K, V> merge(LWWMap<K, V> other) {
        LWWMap<K, V> merged = new LWWMap<>(this.nodeId);
        
        // Copy all registers from this map
        this.registers.forEach((key, reg) -> 
            merged.registers.put(key, reg.copy()));
        
        // Merge with other map
        for (Map.Entry<K, LWWRegister<V>> entry : other.registers.entrySet()) {
            K key = entry.getKey();
            LWWRegister<V> otherReg = entry.getValue();
            
            LWWRegister<V> thisReg = merged.registers.get(key);
            if (thisReg == null) {
                merged.registers.put(key, otherReg.copy());
            } else {
                thisReg.mergeInPlace(otherReg);
            }
        }
        
        return merged;
    }
    
    /**
     * Create a copy
     */
    public LWWMap<K, V> copy() {
        return new LWWMap<>(this.registers, this.nodeId);
    }
    
    @Override
    public String toString() {
        return "LWWMap" + registers.entrySet().stream()
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                e -> e.getValue().read()
            ));
    }
}
```

### Usage Example

```java
public class LWWMapExample {
    public static void main(String[] args) {
        // Node A
        LWWMap<String, String> mapA = new LWWMap<>("node-a");
        mapA.put("name", "Alice");
        mapA.put("age", "30");
        System.out.println("Map A: " + mapA);
        
        // Node B
        LWWMap<String, String> mapB = new LWWMap<>("node-b");
        mapB.put("name", "Bob");
        mapB.put("city", "NYC");
        System.out.println("Map B: " + mapB);
        
        // Merge
        LWWMap<String, String> merged = mapA.merge(mapB);
        System.out.println("Merged: " + merged);
        // "name" will be "Bob" or "Alice" depending on timestamp
        // "age": "30"
        // "city": "NYC"
    }
}
```

---

## CRDTs for Text Editing

### The Challenge

Text editing requires:
- Insert characters at positions
- Delete characters at positions
- Handle concurrent edits
- Maintain cursor positions

### Character-Level CRDT

```java
import java.util.*;

/**
 * Character-level CRDT for text editing
 * Each character has a unique ID and position
 */
public class TextCRDT {
    // List of characters in order
    private final List<TextChar> characters;
    // Map for quick lookup
    private final Map<UUID, TextChar> charMap;
    
    public TextCRDT() {
        this.characters = new ArrayList<>();
        this.charMap = new HashMap<>();
    }
    
    /**
     * Character with unique ID and position
     */
    public static class TextChar implements Comparable<TextChar> {
        public final UUID id;
        public final char value;
        public final UUID positionId; // ID of character before this one
        public final String nodeId;
        public final long timestamp;
        
        public TextChar(char value, UUID positionId, String nodeId) {
            this.id = UUID.randomUUID();
            this.value = value;
            this.positionId = positionId;
            this.nodeId = nodeId;
            this.timestamp = System.currentTimeMillis();
        }
        
        @Override
        public int compareTo(TextChar other) {
            // Compare by position ID, then timestamp, then node ID
            int posCompare = this.positionId.compareTo(other.positionId);
            if (posCompare != 0) return posCompare;
            
            int timeCompare = Long.compare(this.timestamp, other.timestamp);
            if (timeCompare != 0) return timeCompare;
            
            return this.nodeId.compareTo(other.nodeId);
        }
    }
    
    /**
     * Insert character at position
     */
    public UUID insert(char c, int position) {
        UUID positionId = position == 0 ? null : 
            (position < characters.size() ? characters.get(position - 1).id : null);
        
        TextChar textChar = new TextChar(c, positionId, "node-id");
        characters.add(textChar);
        charMap.put(textChar.id, textChar);
        
        // Sort to maintain order
        characters.sort(TextChar::compareTo);
        
        return textChar.id;
    }
    
    /**
     * Delete character
     */
    public void delete(UUID charId) {
        TextChar removed = charMap.remove(charId);
        if (removed != null) {
            characters.remove(removed);
        }
    }
    
    /**
     * Get text as string
     */
    public String getText() {
        // Filter deleted characters and build string
        return characters.stream()
            .map(c -> String.valueOf(c.value))
            .collect(java.util.stream.Collectors.joining());
    }
    
    /**
     * Merge with another TextCRDT
     */
    public void merge(TextCRDT other) {
        // Add all characters from other
        for (TextChar otherChar : other.characters) {
            if (!charMap.containsKey(otherChar.id)) {
                characters.add(otherChar);
                charMap.put(otherChar.id, otherChar);
            }
        }
        
        // Sort to maintain order
        characters.sort(TextChar::compareTo);
    }
}
```

### Logoot Algorithm (Advanced)

```java
/**
 * Logoot: Advanced CRDT for text editing
 * Uses position identifiers instead of integer positions
 */
public class LogootCRDT {
    private final List<LogootChar> characters;
    
    public static class LogootChar {
        public final List<Long> position; // List of integers for position
        public final char value;
        public final String nodeId;
        public final int clock;
        
        public LogootChar(List<Long> position, char value, String nodeId, int clock) {
            this.position = position;
            this.value = value;
            this.nodeId = nodeId;
            this.clock = clock;
        }
        
        public int compareTo(LogootChar other) {
            // Compare positions lexicographically
            int minLen = Math.min(this.position.size(), other.position.size());
            for (int i = 0; i < minLen; i++) {
                int cmp = Long.compare(this.position.get(i), other.position.get(i));
                if (cmp != 0) return cmp;
            }
            return Integer.compare(this.position.size(), other.position.size());
        }
    }
    
    /**
     * Generate position between two positions
     */
    private List<Long> generatePositionBetween(List<Long> pos1, List<Long> pos2) {
        // Generate a position between pos1 and pos2
        // This is complex - see Logoot paper for details
        List<Long> newPos = new ArrayList<>();
        // Simplified version
        if (pos1 == null) {
            newPos.add(0L);
        } else if (pos2 == null) {
            newPos.add(pos1.get(0) + 1);
        } else {
            // Find first differing element and generate between
            int i = 0;
            while (i < pos1.size() && i < pos2.size() && 
                   pos1.get(i).equals(pos2.get(i))) {
                newPos.add(pos1.get(i));
                i++;
            }
            if (i < pos1.size() && i < pos2.size()) {
                long mid = (pos1.get(i) + pos2.get(i)) / 2;
                newPos.add(mid);
            } else {
                newPos.add(0L);
            }
        }
        return newPos;
    }
}
```

---

## Real-World Applications

### 1. Collaborative Text Editors

```java
/**
 * Real-world example: Collaborative editor
 */
public class CollaborativeEditor {
    private final TextCRDT document;
    private final String userId;
    private final WebSocketClient websocket;
    
    public CollaborativeEditor(String userId, String documentId) {
        this.userId = userId;
        this.document = new TextCRDT();
        this.websocket = new WebSocketClient("ws://server/docs/" + documentId);
        this.websocket.onMessage(this::handleRemoteOperation);
    }
    
    public void onUserType(char c, int position) {
        // Insert character
        UUID charId = document.insert(c, position);
        
        // Send operation to server
        Operation op = new Operation(Operation.Type.INSERT, charId, c, position);
        websocket.send(op);
    }
    
    public void onUserDelete(int position) {
        // Delete character
        TextChar charToDelete = document.characters.get(position);
        document.delete(charToDelete.id);
        
        // Send operation
        Operation op = new Operation(Operation.Type.DELETE, charToDelete.id);
        websocket.send(op);
    }
    
    private void handleRemoteOperation(Operation op) {
        // Apply remote operation
        if (op.type == Operation.Type.INSERT) {
            TextChar textChar = new TextChar(op.value, op.positionId, op.nodeId);
            document.characters.add(textChar);
            document.charMap.put(textChar.id, textChar);
            document.characters.sort(TextChar::compareTo);
        } else {
            document.delete(op.charId);
        }
        
        // Update UI
        updateUI();
    }
}
```

### 2. Distributed Counters

```java
/**
 * Real-world example: Like counter
 */
public class LikeCounter {
    private final PNCounter counter;
    private final String postId;
    
    public LikeCounter(String postId, String nodeId) {
        this.postId = postId;
        this.counter = new PNCounter(nodeId);
    }
    
    public void like() {
        counter.increment();
        broadcastOperation(new LikeOperation(postId, true));
    }
    
    public void unlike() {
        counter.decrement();
        broadcastOperation(new LikeOperation(postId, false));
    }
    
    public long getCount() {
        return counter.value();
    }
    
    public void merge(LikeCounter other) {
        this.counter.mergeInPlace(other.counter);
    }
}
```

### 3. Shopping Cart

```java
/**
 * Real-world example: Shopping cart with OR-Set
 */
public class ShoppingCart {
    private final ORSet<CartItem> items;
    private final String userId;
    
    public ShoppingCart(String userId) {
        this.userId = userId;
        this.items = new ORSet<>();
    }
    
    public void addItem(String productId, int quantity) {
        CartItem item = new CartItem(productId, quantity);
        items.add(item);
    }
    
    public void removeItem(String productId) {
        // Find and remove item
        items.elements().stream()
            .filter(item -> item.productId.equals(productId))
            .forEach(items::remove);
    }
    
    public Set<CartItem> getItems() {
        return items.elements();
    }
    
    public void merge(ShoppingCart other) {
        this.items.merge(other.items);
    }
}
```

---

## Advanced Patterns

### 1. CRDT Composition

```java
/**
 * Compose multiple CRDTs
 */
public class CompositeCRDT {
    private final ORSet<String> tags;
    private final LWWMap<String, String> metadata;
    private final PNCounter viewCount;
    
    public CompositeCRDT(String nodeId) {
        this.tags = new ORSet<>();
        this.metadata = new LWWMap<>(nodeId);
        this.viewCount = new PNCounter(nodeId);
    }
    
    public void merge(CompositeCRDT other) {
        this.tags.merge(other.tags);
        this.metadata.merge(other.metadata);
        this.viewCount.merge(other.viewCount);
    }
}
```

### 2. CRDT with Tombstones

```java
/**
 * CRDT with tombstone cleanup
 */
public class TombstoneCRDT<T> {
    private final ORSet<T> active;
    private final Set<T> tombstones; // Removed elements
    private long lastCleanup;
    private static final long CLEANUP_INTERVAL = 24 * 60 * 60 * 1000; // 24 hours
    
    public void remove(T element) {
        active.remove(element);
        tombstones.add(element);
    }
    
    public void cleanup() {
        long now = System.currentTimeMillis();
        if (now - lastCleanup > CLEANUP_INTERVAL) {
            // Remove old tombstones (if safe)
            // This is complex - need to ensure all nodes have seen removal
            lastCleanup = now;
        }
    }
}
```

### 3. CRDT with Vector Clocks

```java
/**
 * CRDT with vector clocks for causal ordering
 */
public class VectorClockCRDT {
    private final Map<String, Long> clock; // Node ID -> clock value
    
    public VectorClockCRDT(String nodeId) {
        this.clock = new HashMap<>();
        this.clock.put(nodeId, 0L);
    }
    
    public void tick(String nodeId) {
        clock.put(nodeId, clock.getOrDefault(nodeId, 0L) + 1);
    }
    
    public boolean happenedBefore(VectorClockCRDT other) {
        // Check if this clock happened before other
        boolean strictlyLess = false;
        for (String nodeId : clock.keySet()) {
            long thisValue = clock.getOrDefault(nodeId, 0L);
            long otherValue = other.clock.getOrDefault(nodeId, 0L);
            if (thisValue > otherValue) return false;
            if (thisValue < otherValue) strictlyLess = true;
        }
        return strictlyLess;
    }
}
```

---

## Performance Considerations

### 1. Memory Optimization

```java
/**
 * Optimized OR-Set with tag compression
 */
public class OptimizedORSet<T> {
    // Use more efficient data structures
    private final Map<T, BitSet> elementTags; // Use BitSet instead of Set<UUID>
    
    // Or use bloom filters for large sets
    private final BloomFilter<T> removedElements;
}
```

### 2. Network Optimization

```java
/**
 * Delta-based synchronization
 */
public class DeltaCRDT<T extends Mergeable<T>> {
    private T state;
    private T lastSyncedState;
    
    public Delta getDelta() {
        // Compute difference between current and last synced
        return computeDelta(lastSyncedState, state);
    }
    
    public void applyDelta(Delta delta) {
        // Apply delta to state
        state = applyDelta(state, delta);
    }
}
```

### 3. Batch Operations

```java
/**
 * Batch operations for efficiency
 */
public class BatchedCRDT {
    private final List<Operation> pendingOperations;
    
    public void addOperation(Operation op) {
        pendingOperations.add(op);
        
        if (pendingOperations.size() >= BATCH_SIZE) {
            flushOperations();
        }
    }
    
    private void flushOperations() {
        // Send batch to other nodes
        broadcastBatch(pendingOperations);
        pendingOperations.clear();
    }
}
```

---

## Best Practices

### 1. Choose the Right CRDT

```java
// ✅ Use G-Counter for like counts
GCounter likes = new GCounter("node-id");

// ✅ Use PN-Counter for upvotes/downvotes
PNCounter votes = new PNCounter("node-id");

// ✅ Use OR-Set for shopping cart
ORSet<Item> cart = new ORSet<>();

// ✅ Use LWW-Register for user profile
LWWRegister<Profile> profile = new LWWRegister<>("node-id");
```

### 2. Handle Network Partitions

```java
/**
 * Handle partitions gracefully
 */
public class PartitionAwareCRDT {
    private final CRDT crdt;
    private final List<String> connectedNodes;
    
    public void merge(CRDT other) {
        // Always merge, even if nodes were partitioned
        crdt.merge(other);
    }
    
    public boolean isPartitioned() {
        return connectedNodes.size() < totalNodes / 2;
    }
}
```

### 3. Monitor and Debug

```java
/**
 * Add monitoring to CRDTs
 */
public class MonitoredCRDT<T extends Mergeable<T>> {
    private final T crdt;
    private final MetricsCollector metrics;
    
    public void merge(T other) {
        long startTime = System.nanoTime();
        crdt.merge(other);
        long duration = System.nanoTime() - startTime;
        metrics.recordMerge(duration);
    }
}
```

---

## Summary

### Key Takeaways

1. **Operation-Based CRDTs** exchange operations instead of state
2. **OR-Set** allows add, remove, and re-add with unique tags
3. **LWW-Register** uses timestamps for conflict resolution
4. **LWW-Map** provides per-key last-write-wins semantics
5. **Text CRDTs** use character-level or position-based approaches
6. **Real-world applications** include collaborative editors, counters, and carts

### CRDT Selection Guide

| Use Case | Recommended CRDT |
|----------|------------------|
| Like counter | G-Counter |
| Vote counter | PN-Counter |
| Shopping cart | OR-Set |
| User profile | LWW-Register |
| Key-value store | LWW-Map |
| Text editor | Character-level CRDT or Logoot |
| Tags | OR-Set |
| Distributed lock | LWW-Register with lease |

### When to Use CRDTs

✅ **Use CRDTs when:**
- Need eventual consistency
- Can tolerate some temporary inconsistency
- Want low latency
- Need offline support
- Operations are commutative

❌ **Don't use CRDTs when:**
- Need strong consistency
- Operations are not commutative
- Need transactional guarantees
- State size is extremely large

### Part 1 + Part 2 Complete Coverage

- ✅ State-Based CRDTs (G-Counter, PN-Counter, G-Set, 2P-Set)
- ✅ Operation-Based CRDTs (OR-Set, LWW-Register, LWW-Map)
- ✅ Text Editing CRDTs
- ✅ Real-World Applications
- ✅ Advanced Patterns
- ✅ Performance Considerations
- ✅ Best Practices

---

*Part 2 completes the comprehensive guide to CRDTs, covering operation-based implementations and advanced topics. Combined with Part 1, this provides a complete understanding of Conflict-free Replicated Data Types.*



