# Data Structures Examples

## Stack

Backed by array:

```java
public class Stack {
    private int[] arr;
    private int i = -1;

    public Stack(int capacity) {
        arr = new int[capacity];
    }

    public int size() {
        return i + 1;
    }

    public void push(int x) {
        if (i == arr.length - 1) throw new RuntimeException("Stack is full");
        arr[++i] = x;
    }

    public int pop() {
        if (i == -1) throw new RuntimeException("Stack is empty");
        return arr[i--];
    }
}
```

## Queue

Backed by array:

```java
public class Queue {
    private int[] arr;
    private int front = -1;
    private int rear = -1;

    public Queue(int capacity) {
        arr = new int[capacity];
    }

    public int size() {
        if (rear == -1 && front == -1) return 0;
        else if (rear >= front) return rear - front + 1;
        else return arr.length + rear - front + 1;
    }

    public void enqueue(int x) {
        if ((rear + 1) % arr.length == front) throw new RuntimeException("Queue is full");

        if (size() == 0) {
            front = 0;
            rear = 0;
            arr[front] = x;
        } else {
            rear = (rear + 1) % arr.length;
            arr[rear] = x;
        }
    }

    public int dequeue() {
        if (size() == 0) throw new RuntimeException("Queue is empty");
        int res = arr[front];

        if (front == rear) {
            front = -1;
            rear = -1;
        } else {
            front = (front + 1) % arr.length;
        }

        return res;
    }
}
```

## Binary Tree

```java
class Tree {
    public int value;
    public Tree left;
    public Tree right;

    public Tree(int value) {
        this(value, null, null)
    }

    public Tree(int value, Tree left, Tree right) {
        this.value = value;
        this.left = left;
        this.right = right;
    }
}
```

## Graph

```java
public class Edge {
    public int weight;
    public int from;
    public int to;
}

public class Vertex implements Comparable<Vertex> {
    public int id;
    public Set<Edge> edges = new HashSet<>();
    public Vertex parent = null;

    public int distance = Integer.MAX_VALUE; // for dijkstra
    public boolean discovered = false;       // for bfs/bfs traversal

    public Vertex(int id) {
        this.id = id;
    }

    /*
     * For dijkstra priority queue
     */
    @Override
    public int compareTo(Vertex o) {
        if (this.distance < o.distance) return -1;
        else if (o.distance < this.distance) return 1;
        else return 0;
    }
}

public class Graph {
    public Map<Integer, Vertex> vertices = new HashMap<>();
}
```

## Trie

```java
public class Trie {
    public Character c;
    public Map<Character, Trie> chars = new HashMap<>();
    public boolean isLeaf = false;

    /*
     * For root element that has no char
     */
    public Trie() {
        this(null);
    }

    public Trie(Character c) {
        this.c = c;
    }
}
```

## BitSet

```java
public class BitSet {
    private byte[] a;

    public BitSet(int size) {
        this.a = new byte[size];
    }

    public boolean get(int i) {
        return (a[i / 8] & 1 << (i % 8)) == 1 << (i % 8);
    }

    public void set(int i, boolean value) {
        if (value)
            a[i / 8] |= 1 << (i % 8);
        else
            a[i / 8] ^= 1 << (i % 8);
    }
}
```
