---
layout: page
title: DS Intro  
permalink: /google-ds-intro/
---



# Database Intro 

## Arrays

- Arrays are the most basic type of data structure that stores similar kind of values. 
- Each item in an array is an element and can be accessed using an index. 
    - Normally arrays have 0 based index which means indexing starts from 0 and goes till _n - 1_ where n is the number of elements in an array.
    - Arrays can have 0 based indexing, 1 based indexing or n based indexing (for negative values)
- Size of an array is fixed at runtime when initialized. It can't be changed after initialization.
- If the size has to be changed after initialization, use _ArrayList_ (Collection class) instead.
- Size of an array can be specified using int only since arrays are int based index.
- The ArrayIndexOutOfBoundsException is thrown when a non-existing index of an array is being accessed.
- Arrays are used to implement mathematical vectors and matrices, as well as other kinds of rectangular tables. 
- Many databases, small and large, consist of (or include) one-dimensional arrays whose elements are records.
- Arrays are used to implement other data structures, such as heaps, hash tables, deques, queues, stacks, strings, and VLists.
- Below is the representation, 

<img width="426" alt="arrays-1" src="https://user-images.githubusercontent.com/3439029/32186457-5b7d71d2-bd5f-11e7-8ab8-5ee19f34ff6b.png">
**************************************

**Defining Arrays**
- Java offers several ways of defining and initializing arrays, including literal and constructor notations. When declaring arrays without explicit values, each element will be initialized with a default value:
    - **0** for primitive numerical types: _byte, short, char, int, long, float, and double._
    - **false** for the _boolean_ type.
    - **null** for _reference_ types.

In Java, it is possible to have arrays of size 0:
```java 
int[] array = new int[0]; // Compiles and runs fine.
```
However, since it's an empty array, no elements can be assigned to it:
```java
array[0] = 1; // Throws java.lang.ArrayIndexOutOfBoundsException.
```
Such empty arrays are typically used as return values so that the calling code only has to worry about dealing with an array, rather than a potentially null value that may lead to a NullPointerException.
The length of an array must be a non-negative integer:
```java
int[] array = new int[-1]; // Throws java.lang.NegativeArraySizeException
```
The array size can be determined using a public final field called length:
```java
System.out.println(array.length); // Prints 0 in this case.
```
**Note:** _array.length_ returns the actual size of the array and not the number of array elements which were assigned a value, unlike ArrayList#size() which returns the number of array elements which were assigned a value.
**************************************

**Syntax**
```java
ArrayType[] variableName; // Declaring arrays
ArrayType variableName[]; // Another valid syntax (less commonly used)
ArrayType[][][] variableName; // Declaring multi-dimensional jagged arrays (repeat []s)
ArrayType myVar = array[index]; // Accessing (reading) element at index
array[index] = value; val
ArrayType[] myArray = new ArrayType[arrayLength]; // Array initialization syntax
int[] ints = {1, 2, 3}; // Array initialization syntax with values provided, length is inferred from the number of provided values: {[value1[, value2]*]}
new int[]{4, -5, 6} // Can be used as argument, without a local variable
int[] ints = new int[3]; // same as {0, 0, 0}
int[][] ints = {{1, 2}, {3}, null}; val
```
**************************************

**Multi Dimensional Arrays**  
```java
int[][] a = new int[3][4]; // Creates a matrix with 3 rows and 4 columns
```
Below is the representation of multi dimensional array  

![arrays-2](https://user-images.githubusercontent.com/3439029/32186494-70b12fa8-bd5f-11e7-83d4-5f61f38ccb75.png)

There are two orders that can exist in a multidimensional array, i.e  
_Row major_ and _Column major_  
ex, consider below matrix

<img width="185" alt="arrays-3" src="https://user-images.githubusercontent.com/3439029/32186517-7c5991ba-bd5f-11e7-9c82-f369b3590492.png">

Here, in row-major layout, elements can be represented as **[1,2,3,4,5,6,7,8,9]** whereas in column-major layout, elements can be represented as **[1,4,7,2,5,8,3,6,9]**

**************************************

**Creating and initializing generic type arrays**  

In generic classes, arrays of generic types cannot be initiated like so due to type erasure:
```java
public class MyGenericClass<T> {
    private T[] a;

    public MyGenericClass() {
        a = new T[5]; // Compile time error: generic array creation
    }
}
```
Instead, they can be created using one of the following methods:  

**1.** By creating an Object array, and casting it to the generic type:
```java
a = (T[])new Object[5];
```
This is the simplest method, but since the underlying array is still of type Object[], this method does not provide type safety, so the array is best used only within the generic class, not exposed publicly.  

**2.** By using Array.newInstance with a class parameter:
```java
public MyGenericClass(Class<T> clazz) {
    a = (T[]) Array.newInstance(clazz, 5);
}
```
Here the class of T has to be explicitly passed to the constructor. The return type of Array.newInstance is always Object. However, the newly created array is in fact of type T[], and can be safely externalized.
**************************************

**Sorted Arrays**  

Arrays are considered to be sorted in ascending order when each element on the left of that element in an array is smaller than that element itself. Vice versa goes for descending order.

**Pros:**
- Accessing an element is fast using the index - access time is O(1).
- Much faster to process - see this question.  
    
**Cons:**
- Insertion and deletion are slow, subsequent elements must be moved - complexity for insertion, in that case is O(n).
- A large enough block of memory is needed to hold the array.
- Easily corrupted (data could be inserted in the middle).  

**************************************

**Arrays v/s List**

| **Property**        | **Array**           | **List**  |
| ------------- |-------------| -----|
| **Size** | _Fixed length. Cannot change the size after creation_ | _Dynamic in size. Capacity grows as the elements are added_ |
| **Content** | _Can contain primitive data types and objects_ | _Objects only. No primitive data types_ |
| **Dimension** | _Can be multi-dimensional_ | _Normally single dimensional, but can be made multi dimensional_ |
| **Type Safety** | _Typesafe, meaning that array will contain objects of specific class or primitives of specific data type_ | _Not type-safe by default. Generics can be used to make a List type safe_ |
| **Insertion/Deletion** | _Shifting existing elements may be needed if action is performed not at the end of the array_ | _Easy insertion/deletion methods provided_ |
**************************************

**Disadvantages of Arrays**
- **Fixed Size**: The size of the array is static, i.e we define the size when we create the array, which can't change later on.
- **One block allocation**: To allocate the array, in the beginning, we may not have enough memory to hold entire array because of continuous block of allocation.
- **Complex position based insertion**: This is applicable only in case of dynamic array or list, where size can change. If we want to insert an element at position n, we have to shift all the elements to the right by one position to make space for this new element, which is a time consuming and costly operation.

**Operations on Arrays** 

Below operations on Arrays are implemented [here](../Arrays/BasicOperations.java)
- Iterating over arrays in regular order and reverse order
- Converting arrays to Stream
- Finding an element in array
- Getting the length of the array
- Traversing through the elements of array
- Copying/Resizing arrays
- Removing elements from arrays
- Sorting elements in an array
- Arrays as method params
- Arrays in memory
- Casting arrays
- Comparing arrays for equality
- Converting array of objects to array of primitives
- Converting arrays between primitives and boxed types
- Reversing an array

**************************************

**Comparison with other Data Structures**

<img width="972" alt="arrays-4" src="https://user-images.githubusercontent.com/3439029/32186532-87cb9778-bd5f-11e7-86de-179d914b0d80.png">


## Caching Introduction

Caching is a technique of storing the data in a temporary place since fetching the original data over API call can be expensive. From cache, we can retrieve it faster. Cache is made of pool of entries, which are copy of real data. Each entry is associated with a tag (key identifier) which is used for retrival. 

Lets assume we have some products in the backend and each of the product has a product id which is unique.  
Here are some of the terms used with caching,  

**Cache Hit**  
When client invokes the request to view product details, we first check the cache if details of that product are available in cache. If a entry is found with the tag requested, we fetch the details from the cache. This is called cache hit i.e requested value is available in cache. Percentage of cache hit is called hit ratio or hit rate.  

**Cache Miss**  
On the contrary, when tag is not found in the cache, it is called as cache miss. In this case call to backend is made and cache is updated so that next time we can avoid cache miss in future.  
If we encounter cache miss, there are two possible scenarios that can happen :  
- We have enough free space in the cache since cache hasn't reached its limit, so the object that caused cache miss will be fetched from backend and will be inserted in the cache.  
- There is no free space in the cache since cache has reached its capacity, so the object that caused the cache miss will be fetched from backend and we have to decide which object can be evicted (deleted) now to make free space for new one. This is done by replacement policy which will be discussed later.  
Below diagram shows the concept of cache hit and cache miss,  

![cache-hit-ratio-latency-graph](https://cloud.githubusercontent.com/assets/3439029/19374486/c270bd1a-9180-11e6-8df4-66fd7aab4a8e.png)

**Storage Cost**  
When a cache miss happens, data is fetched from backend and stored in cache. But how much space is needed to store this new object. The cost associated with this operation is called storage cost.  

**Retrival Cost**  
When we have to load the data, we must know how much cost is associated with it. It is called retrival cost.  

**Invalidation**  
Objects are stored in the cache for very long time, but in the meantime it is very much possible that actual data changes in backend. So, we need to invalidate the current cache at regular interval of time so that new updated data can be populated in cache.  

**Replacement policy**  
When cache miss happens, it ejects some objects to make space for new entries. We have to select some objects for deletion at this point of time. Policy which decides which elements will be replaced is called replacement policy. For example, a cache entry that is not going to be used for the next 10 seconds will be replaced by an entry that is going to be used within the next 2 seconds.  

_Below are some of the caching algorithms (Replacement policy),_  

**1. Least frequently used cache (_LFU_)**  
- This cache keeps a track how often a entry is fetched from cache by keeping a counter against each entry. When time for eviction comes, entries with least number in counter will be deleted. This is not that good since keeping a track of extra counter is a overhead.  

**2. Least recently used cache (_LRU_)**  
- This cache removes the least recently used item from the cache i.e the one which is not used for longest time. Web browser uses this technique for caching. New items are placed on top of cache and when limit exceeds, items from bottom of the list are removed. So, when a new item is accessed it is placed on top of the list. This means, items which are frequently accessed tends to remain in the cache. There are 2 ways to implement this i.e either using Arrays or Linked List. 
There are some improved versions of LRU as well normally known as LRU 2 and 2Q.  

**3. Least recently used cache 2 (_LRU 2_)**  
- This cache add entry to it when a element is accessed twice in a row. When cache becomes full it removes the element which has second most recent access. Because of need to track two most recently accessed items, there is an overhead involved. This also needs to keep track of elements which are accessed one time, so that they can be added to cache when accessed twice.  

**4. Two Queues**  
- Two queues add entries to cache as and when they are accessed. If a entry is accessed twice, it is moved to a bigger cache. This is called two queue cache because it is maintained by two different queues. First queue (small one) is generally kept 1/3rd of the size of second queue. It provides advantage of LRU2 cache and reduce the effort of maintaining the track of two items.  

**5. Adaptive replacement cache (_ARC_)**  
- Sometimes it is said that this cache balances out between LFU and LRU, but that's not 100% true. In this cache, 2 lists are maintained. First list contains the entries that are seen just once recently, whereas second list contains the entries which are seen atleast twice recently. So, first list maintains _"recency"_ and second list maintains _"frequency"_. This is fast and considered one of the best replacement algorithm.  

**6. Most recently used cache (_MRU_)**  
- Unlike LRU, this cache removes the most recently accessed element from the cache first when cache becomes full. This is because accessing resources is highly unpredictable. It works on the theory that if a element is accessed just now, it won't be accesed for some time now and the one's which were accessed last, it's there turn now. So this removes the item which is most recently used.  
ex. You are standing at a bus stop and checking the entries of buses based on the route number. So, if you have seen a bus of route 16 entering the stop just now, it is very unlikely to see another bus of route number 16 very soon.  

**7. First in first out (_FIFO_)**  
- This is first in first out cache, and is implemented using a queue. This queue keeps the most recently accessed entry at the back and least recently at the front. When cache becomes full or we have to make space for the new entry, we remove the entry that is at the front. 
Accessed Just Now **> > > > > >** Accessed some time back.  

**8. Second Chance**  
- This works on the same concept as FIFO works, i.e it keeps the track of entries in the queue. But when cache becomes full, instead of removing the first entry blindly, this checks if there is a bit set to the entry. This bit is set based on the fact if this entry was accessed earlier as well. For ex. If a entry is accessed once bit can have 0, or if it is accessed more then once, bit can have value as 1. So if this cache finds the bit associated with a entry as 1, it knows that is entry was accessed twice before and it moves it back to the queue instead of removing it. This can be implemented using a circular list.  

**9. Simple time based**  
- This works based on the time. Entries in the cache remains for some specific amount of time. When that time limit hits, this cache invalidates all the entries and populate the cache again with fresh data.  



## Introduction to Graphs

Graph is a Non-Linear type of data structure which contains Vertices and Edges. Wikipedia defines it as _"A graph data structure consists of a finite (and possibly mutable) set of vertices or nodes or points, together with a set of unordered pairs of these vertices for an undirected graph or a set of ordered pairs for a directed graph."_ Even tree is a special kind of graph.  

In Mathematical terms, a Graph can be represented by below definition,  
A graph **G** is an ordered pair of set **V** of vertices and set **E** of edges. i.e **G = (V, E)**
Since it is an ordered pair, so sequence of items matter. 
	
**Ordered Pair v/s UnOrdered Pair**
- **Ordered Pair :** A pair in which order matters and changing the order is not allowed is called an ordered pair. ex. (a, b) != (b, a) if a != b
- **UnOrdered Pair :** A pair in which order doesn't matter is called an unordered pair. ex. {a, b} = {b, a}
	
Now, let's say we have a following graph defined
<img width="644" alt="basic_graph" src="https://cloud.githubusercontent.com/assets/3439029/21968825/ea5fcb4c-db4d-11e6-9eb4-1e40569b230c.png">

We have 6 vertices defined here and 9 edges, which can be represented as below,  
- **V =** _{V0, V1, V2, V3, V4, V5}_
- **E** = _{{V0, V4}, {V0, V5}, {V3, V5} ...}_

Edge is represented like this because it is a pair of 2 vertices i.e start vertex and end vertex. 
Edges can be of two types, i.e either Directed edges or UnDirected Edges.  
Below are the graphs which are created using directed edges and undirected edges. Based on edges, graphs can be categorized in two categories, i.e Directed graphs (or Digraph) and UnDirected graphs.

![directed_vs_undirected](https://cloud.githubusercontent.com/assets/3439029/21968901/a5e4f63a-db4e-11e6-84ba-bac70fed4b69.png)

Directed edge is represented as **(S, D)** where **S** is source and **D** is destination, whereas UnDirected edge is represented as **{S, D}**. Typically in a graph, either all edges are directed or all of them are undirected but it can be both as well.

**Undirected v/s Directed Graphs**
- A social media network like **Facebook** can be a example of **Undirected Graph** where each user is connected to some other user and vice versa. There is a mutual relationship here (i.e if you are a friend of some other person, then he is also your friend).  Now, let's talk about a use case of _"Suggest Friends"_. We can do this by suggesting friends of friends which are not connected to each other i.e find all the nodes where path = 2 (2 because 1 refers to our own friend and second will be the friend of friend).
- **WWW (World Wide Web)** can be a perfect example of **Directed Graph** where each of the web page is connected to some other web page. Here relationship is not mutual. i.e if Page A has a link to Page B, then it's not necessary that Page B will also have a link to Page A. Though there can be self links to the pages. We Crawler does the same thing as well. It traverses through the web pages and create a directed graph.

## Linked List

We already have arrays, what's the problem with it that we have to move with a new data structure??
- Arrays are fixed size data structure. Once created in memory we cannot cross that limit. Resizing can be done but there is no surety that we will have that much memory available later on. Arrays are contiguous block of memory. For ex. We created a array of initial size 10. Application is running and it has taken lot of memory. We got a need to increase the size of out array to 25 now. We don't have enough contiguous memory to create it. **Problem !!!!**  
- Array insertion and deletion may require adjustment in size. (Overhead)  
- The best part with Arrays is, each element can be accessed in a very efficient manner since index is known.

This is how a simple example of adding a new node to a linked list looks like,

![linked_list](https://cloud.githubusercontent.com/assets/3439029/22178994/1d5fc414-dffb-11e6-8177-7a5628c09b46.png)

Applications which can be better managed without using a contiguous memory based data structure, Linked lists are introduced.

So, a proper **definition** would be something : Linked list is a collection of objects (called Nodes) connected to each other in the memory. Each Node contains some data and pointer to some other Node. Last node points to null reference. Entry point to the linked list is called first node or head node. If list is empty, head will be null.  
Unlike arrays, nodes cannot be accessed using index. Also they don't take contiguous memory.
If we want to traverse through the list, we have to start from head and then check each element.

We have three types of Linked List : _Singly Linked List, Doubly Linked List and Circular Linked List._

- **Singly Linked List** : It provides access to list from the head node. Traversal is allowed only one way and there is no going back. Some of the applications of singly linked list are,
    - Implement stacks, queues, graphs, trees.
    - Symbol table management in compiler design
    - Separate chaining in Hash tables
    - Applications that have an MRU list
    - Cache in browser
    - Undo functionality in software
    - Represent a deck of cards in a game.
    - Represent polynomials

Complexity of various operations can be defines as follows,  
Note : Addition and deletion here is happening at head. If it happens at tail, complexity will be O(n)

<img width="493" alt="singly_linked_list" src="https://cloud.githubusercontent.com/assets/3439029/22179122/e3ad7596-dffe-11e6-81bf-823e8ed38c64.png">

- **Doubly Linked List** : It provides access to list from head and tail nodes. Traversal is allowed from both the sides of the list. Some of the applications of doubly linked list are,
    - Maintain references to active processes, threads, and other dynamic objects in operating system.
    - Stack, hash table, and binary tree can be implemented using a doubly linked list.
    - Separate chaining in Hash tables
    - Applications that have an MRU list (a linked list of file names)

Complexity of various operations can be defines as follows, 

<img width="493" alt="singly_linked_list" src="https://cloud.githubusercontent.com/assets/3439029/22179122/e3ad7596-dffe-11e6-81bf-823e8ed38c64.png">

- **Circular Linked List** : It provides access to list from the head node. But pointer of last node again points back to first node. Circular Queue is important data structure when the data needs to be cache. Some of the applications of circular linked list are,
    - Corners of a polygon.
    - Pool of buffers that are used and released in FIFO ("first in, first out") order.
    - Set of processes that should be time-shared in round-robin order

### Operations that can be performed on Linked Lists :
- Traversing through linked list
- Check if list is empty
- Check the size of the list
- Inserting a element in the list - *This can be at beginning, at end or at a given position*
- Search a element by index
- Search a element by value
- Delete a element from the list - *This can again be at beginning, at end or at given position*
- Converting a list to and from a Array 
- Reverse a linked list using iteration and recursion
- Print elements in forward and reverse order

## Queue Introduction

A queue is a container of objects (a linear collection) that are inserted and removed according to the first-in first-out (FIFO) principle. An excellent example of a queue is a line of students in the food court. New additions to a line are made to the back of the queue, while removal (or serving) happens in the front. In the queue only two operations are allowed enqueue and dequeue. Enqueue means to insert an item into the back of the queue, dequeue means removing the front item. 
The picture demonstrates the FIFO access.

![queues](https://cloud.githubusercontent.com/assets/3439029/22179653/301eab38-e00f-11e6-908a-eea4c8515da2.png)

The difference between stacks and queues is in removing. In a stack we remove the item the most recently added; in a queue, we remove the item the least recently added.

There are various applications of Queue :

Queue, as the name suggests is used whenever we need to have any group of objects in an order in which the first one coming in, also gets out first while the others wait for there turn, like in the following scenarios,
- Serving requests on a single shared resource, like a printer, CPU task scheduling etc.
- In real life, Call Center phone systems will use Queues, to hold people calling them in an order, until a service representative is free.
- Handling of interrupts in real-time systems. The interrupts are handled in the same order as they arrive, First come first served.


## Stack Introduction

Stack is a logical data structure that can be logically thought as a linear structure and can be represented in real life as pile of plates or deck of cards. In this structure, insertion and deletion of items takes place only at one end called "_Top_" of the stack. The basic concept can be illustrated by thinking of your data set as a stack of plates or books where you can only take the top item off the stack in order to remove things from it.

The basic implementation of a stack is also called a **LIFO** (Last In First Out) to demonstrate the way it accesses data. There are basically three operations that can be performed on stacks . They are 1) inserting an item into a stack (_push_). 2) deleting an item from the stack (_pop_). 3) displaying the contents of the stack(_peek_).   

![push_pop](https://cloud.githubusercontent.com/assets/3439029/22179613/b78d8fd2-e00d-11e6-9f38-63628ad02f16.png)

All operations except size() can be performed in **O(1)** time. size() runs in at worst **O(N)**.  
It can be implemented using either Arrays, LinkedList or Queues.

There are various applications of Stack :  
- Converting a decimal number to binary number.
- Tower of Hanoi
- Expression evaluation and syntax parsing
- Conversion of Infix to Postfix
- QuickSort etc.


## Strings Introduction

- Strings are basically sequence of characters stored in a memory. 
- They are immutable in java i.e you cannot change the string object itself but you can change the reference of the object. 
- Java provides a String class to create a manipulate strings. These are objects backed up by char array.
- When a string is created, it's value is cached in string pool and then stored in Heap. 
- Below is the example of creation of string using, both literal and object. When it's created using literals, it's value is cached but when created using the new object, it does not get's cached, until we explicitly asks to cache it.

```java
String s1 = "Hello"; // New string creation using literals.
String s2 = "Hello";
String s3 = new String("Hello"); val
System.out.println(s1 == s2); // Prints True
System.out.println(s1 == s3); // Prints False
```

- To make s2 and s3 point to same reference, we have to invoke the API **intern()** of String class. This API will ensure that string gets cached in the string pool.

```java
String s3 = new String("Hello").intern();
```
<img width="614" alt="screen shot 2017-02-12 at 11 51 43 am" src="https://cloud.githubusercontent.com/assets/3439029/22865449/c407fafa-f119-11e6-89ca-04d45abe425b.png">

- Strings are marked as final in java, i.e once a value is assigned to the string, it cannot change. 
- For ex., the method toUpperCase() constructs and returns a new String instead of modifying the its existing content.
- Moreover, Strings are thread safe as well in java.

**StringBuffer and StringBuilder :**

- As explained earlier, Strings are immutable because String literals with same content share the same storage in the string common pool. Modifying the content of one String directly may cause adverse side-effects to other Strings sharing the same storage.
- JDK provides two classes to support mutable strings: StringBuffer and StringBuilder (in core package java.lang) . A StringBuffer or StringBuilder object is just like any ordinary object, which are stored in the heap and not shared, and therefore, can be modified without causing adverse side-effect to other objects.
- StringBuilder class was introduced in JDK 1.5. It is the same as StringBuffer class, except that StringBuilder is not synchronized for multi-thread operations. However, for single-thread program, StringBuilder, without the synchronization overhead, is more efficient.

**StringTokenizer**

- Very often, you need to break a line of texts into tokens delimited by white spaces. The java.util.StringTokenizer class supports this.

**String API's :** 

```java
- Below are some basic API's. All are not covered here, 
    1. charAt()
        - returns the character located at the specified index.
    2. equalsIgnoreCase()
        - determines the equality of two Strings, ignoring their case (upper or lower case doesn't matters with this function.
    3. length()
        - returns the number of characters in a String.
    4. replace()
        - replaces occurrences of character with a specified new character.
    5. substring()
        - returns a part of the string. This method has two forms,
            - public String substring(int begin);
            - public String substring(int begin, int end); 
    6. toLowerCase()
        - returns string with all uppercase characters converted to lowercase
    7. valueOf()
        - used to convert primitive data types into Strings.
    8. toString()
        - returns the string representation of the object used to invoke this method. 
        - toString() is used to represent any Java Object into a meaningful string representation
    9. trim()
        - returns a string from which any leading and trailing white spaces has been removed
```


## Introduction to Trees

Tree data structure consists of Nodes. Below are some of the aspects of nodes, 
- Each tree has a root node.
- Root node has zero or more child nodes.
- Each child node has zero or more child nodes and so on.  

A tree cannot contain cycles. The nodes may or may not be in a particular order. They could have any data value stored in them. They may or may not have links to there parent nodes, but each parent node will have links to the child nodes.  
NOTE : Tree is actually a type of Graph

**Trees vs. Binary Trees**  
A binary tree is a tree in which each node has atmost 2 childrens. Not all trees are binary trees. Ex, below tree is not a binary tree, we call it a ternary tree. 

![ternary_tree](https://cloud.githubusercontent.com/assets/3439029/19421752/c5ca281e-93bc-11e6-9a64-56ef41710140.png)

There are ocassions when we may have to deal with non-binary trees. Ex. If we want a tree to represent bunch of phone numbers, we may use a 10-ary tree with each node having upto 10 nodes(one for each digit). Node is called a "leaf" if it has no childrens. 

**Binary Trees vs. Binary Search Trees**  
Every node in Binary Search Tree(BST) satisfies a property i.e _All left Nodes <= Root Node < All Right Nodes_ whereas Binary Trees does not hold any such property, they just make sure that each node except leaf nodes have atmost 2 childrens.  

**Balanced vs. UnBalanced**  
While many trees are balanced but not all. Two most common balanced trees are AVL Trees and Red Black Trees. Some definitions to check at this point of time are, 
 - _Complete Binary Tree_ : In this tree every level of the tree is fully filled, except the last level. If last level is filled, then it is filled from left to right.
 - _Full Binary Tree_ : A tree is considered as full Binary Tree if each node has either Zero or Two childs i.e none of the node should have 1 child.
 - _Perfect Binary Tree_ : A tree is considered as Perfect Binary Tree if it is both complete and full.
 
**Binary Tree Traversal**
There are three types of traversal in general that can be applied to a binary tree, 
i.e InOrder, PreOrder and PostOrder
 - _InOrder Traversal_ : Left > Current > Right i.e Visit the left sub tree first, then current node and then right sub tree.
 - _PreOrder Traversal_ : Current > Left > Right i.e Visit the current node first then its child nodes (That's why it's called Pre-Order)
 - _PostOrder Traversal_ : Left > Right > Current i.e Visit the child nodes first and then current node (That's why it's called Post Order)
 
**AVL Tree**  
This is a self balancing BST. In an AVL tree, height of two subtrees of a given node differ by at most one. If at any time they differ by more then one, re balancing is done to restore this property.  
Lookup, insertion and deletion all of them takes O(log n) time in both average and worst cases, where n is the number of nodes in the tree prior to operation.   

AVL trees are often compared with Red-Black trees because both the them supports same set of operations and takes same amount of time.
Because AVL trees are more rigidly balanced, they are faster than red-black trees for lookup intensive applications. However, red-black trees are faster for insertion and removal.
Term used in AVL trees is balance factor, which is derived as
	BalanceFactor = height(left subtree) - height(right subtree)
At any given node, if this balance factor becomes > 1, then re balancing happens. If for some reason re balancing fails, then this tree is not considered as AVL tree.


Rotations :
To make itself balanced this tree performs 4 different kind of operations, 
	- Left Rotation
	- Right Rotation
	- Left-Right Rotation
	- Right-Left Rotation
	
Left Rotation : If a tree becomes unbalanced when a node is inserted in right sub tree, then left rotation is done.
Right Rotation : AVL tree may become unbalanced when a node is inserted in left sub tree, this will result in right rotation.
Left Right Rotation : Double rotations are slightly complex. This is left rotation followed by right rotation. 
Right Left Rotation : This is right rotation followed by left rotation.

	
## Trie Introduction

**Trie**, also known as _Digital Tree_ or _Prefix Tree_ (as they can be searched using a prefix string) is a kind of search tree where keys are usually strings. Unlike a binary search tree, no node in the tree stores the key associated with that node; instead its position in the tree is defined by the key with which it is associated. All the descendants of a node have a common prefix and root is kept as empty. Values are not necessarily associated with all the nodes. Below is a example of a Trie,

<img width="420" alt="trie" src="https://cloud.githubusercontent.com/assets/3439029/22075109/8c5ecc28-dd5f-11e6-8592-f04d13aa73a5.png">

In the above example shown, keys are listed in the nodes and values below them. Each complete English word has an arbitrary integer value associated with it. Though tries are usually keyed by character strings, they need not be. The same algorithms can be adapted to serve similar functions of ordered lists of any construct, e.g. permutations on a list of digits or shapes.

**Advantages v/s Disadvantages of Tries**  

Trie has a number of advantages over binary search trees. A trie can also be used to replace a hash table, over which it has the following advantages:
- Looking up data in a trie is faster in the worst case, O(m) time (where m is the length of a search string), compared to an imperfect hash table. An imperfect hash table can have key collisions. A key collision is the hash function mapping of different keys to the same position in a hash table. The worst-case lookup speed in an imperfect hash table is O(N) time, but far more typically is O(1), with O(m) time spent evaluating the hash.
- There are no collisions of different keys in a trie.
- Buckets in a trie, which are analogous to hash table buckets that store key collisions, are necessary only if a single key is associated with more than one value.
- There is no need to provide a hash function or to change hash functions as more keys are added to a trie.
- A trie can provide an alphabetical ordering of the entries by key.

Tries do have some **drawbacks** as well:

- Tries can be slower in some cases than hash tables for looking up data, especially if the data is directly accessed on a hard disk drive or some other secondary storage device where the random-access time is high compared to main memory.
- Some keys, such as floating point numbers, can lead to long chains and prefixes that are not particularly meaningful. Nevertheless, a bitwise trie can handle standard IEEE single and double format floating point numbers.
- Some tries can require more space than a hash table, as memory may be allocated for each character in the search string, rather than a single chunk of memory for the whole entry, as in most hash tables.

A special kind of trie, called a suffix tree, can be used to index all suffixes in a text in order to carry out fast full text searches.

## Vector Introduction

The _Vector_ class implements a growable array of objects. Like an array it contains components that can be accessed using an integer index. However, the size of a vector can grow or shrink as needed to accommodate addition and removal of elements. Each vector tries to optimize storage management by maintaining a _capacity_ and a _capacityIncrement_. The _capacity_ is always at least as large as the vector size; it is usually larger because as components are added to the vector, the vector's storage increases in chunks the size of _capacityIncrement_. An application can increase the capacity of a vector before inserting a large number of components; this reduces the amount of incremental reallocation.

**Synchronization :**
- Vector synchronizes on each individual operation. That's almost never what you want to do. Generally you want to synchronize a whole sequence of operations. Synchronizing individual operations is both less safe (if you iterate over a Vector, for instance, you still need to take out a lock to avoid anyone else changing the collection at the same time, which would cause a ConcurrentModificationException in the iterating thread) but also slower (why take out a lock repeatedly when once will be enough)?
- Vector also has a bunch of methods around enumeration and element retrieval which are different than the List interface, and developers (especially those who learned Java before 1.2) can tend to use them if they are in the code. Although Enumerations are faster, they don't check if the collection was modified during iteration, which can cause issues, and given that Vector might be chosen for its synchronization - with the attendant access from multiple threads, this makes it a particularly pernicious problem. Usage of these methods also couples a lot of code to Vector, such that it won't be easy to replace it with a different List implementation.

**Difference between ArrayList and Vector :**
- **Synchronization**
    - ArrayList is non-synchronized which means multiple threads can work on ArrayList at the same time. For e.g. if one thread is performing an add operation on ArrayList, there can be an another thread performing remove operation on ArrayList at the same time in a multithreaded environment while Vector is synchronized. This means if one thread is working on Vector, no other thread can get a hold of it. Unlike ArrayList, only one thread can perform an operation on vector at a time.
- **Resize** 
    - Both ArrayList and Vector can grow and shrink dynamically to maintain the optimal use of storage, however the way they resized is different. ArrayList grow by half of its size when resized while Vector doubles the size of itself by default when grows.
- **Performance** 
    - ArrayList gives better performance as it is non-synchronized. Vector operations gives poor performance as they are thread-safe, the thread which works on Vector gets a lock on it which makes other thread wait till the lock is released.
- **fail-fast** 
    - First let me explain what is fail-fast: If the collection (ArrayList, vector etc) gets structurally modified by any means, except the add or remove methods of iterator, after creation of iterator then the iterator will throw ConcurrentModificationException. Structural modification refers to the addition or deletion of elements from the collection. As per the Vector _javadoc_ the Enumeration returned by Vector is not fail-fast. On the other side the iterator and listIterator returned by ArrayList are fail-fast.
- **Who belongs to collection framework really?** 
    - The vector was not the part of collection framework, it has been included in collections later. It can be considered as Legacy code. There is nothing about Vector which List collection cannot do. Therefore Vector should be avoided. If there is a need of thread-safe operation make ArrayList synchronized as discussed in the next section of this post or use CopyOnWriteArrayList which is a thread-safe variant of ArrayList.

**Similarities :**
- Both Vector and ArrayList use growable array data structure.
- The iterator and listIterator returned by these classes (Vector and ArrayList) are fail-fast.
- They both are ordered collection classes as they maintain the elements insertion order.
- Vector & ArrayList both allows duplicate and null values.
- They both grows and shrinks automatically when overflow and deletion happens.

**When to use ArrayList and when to use vector?**
- It totally depends on the requirement. If there is a need to perform “thread-safe” operation the vector is your best bet as it ensures that only one thread access the collection at a time.
- Performance 
    - Synchronized operations consumes more time compared to non-synchronized ones so if there is no need for thread safe operation, ArrayList is a better choice as performance will be improved because of the concurrent processes
  