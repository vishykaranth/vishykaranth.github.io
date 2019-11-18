---
layout: page
title: HashMap  
permalink: /google-ds-hashmap/
---



## HashMap

### Hashing Introduction

#### What is Hashing?
- Algorithm that calculates a fixed-size bit value of a given resource(block of data). 
- Hashing transforms this data into a far shorter fixed-length value or key. 
- The hash value can be considered the distilled summary of everything within that file.
- A good hashing algorithm would exhibit a property called the avalanche effect, where the resulting hash output would change significantly or entirely even when a single bit or byte of data within a file is changed. 
- A hash function that does not do this is considered to have poor randomization, which would be easy to break by hackers.
- A hash is usually a hexadecimal string of several characters. 
- Hashing is also a unidirectional process so you can never work backwards to get back the original data.
- A good hash algorithm should be complex enough such that it does not produce the same hash value from two different inputs. 
- If it does, this is known as a hash collision 
- A hash algorithm can only be considered good and acceptable if it can offer a very low chance of collision.

#### Why do we need it?
- If you are transferring a file from one computer to another, how do you ensure that the copied file is the same as the source? 
- One method you could use is called hashing, which is essentially a process that translates information about the file into a code. 
- Two hash values (of the original file and its copy) can be compared to ensure the files are equal.

#### Benefits of Hashing?
- One main use of hashing is to compare two files for equality. 
- Without opening two document files to compare them word-for-word, the calculated hash values of these files will allow the owner to know immediately if they are different.
- Hashing is also used to verify the integrity of a file after it has been transferred from one place to another, typically in a file backup program like SyncBack. 
- To ensure the transferred file is not corrupted, a user can compare the hash value of both files. If they are the same, then the transferred file is an identical copy.
- In some situations, an encrypted file may be designed to never change the file size nor the last modification date and time (for example, virtual drive container files). 
- In such cases, it would be impossible to tell at a glance if two similar files are different or not, but the hash values would easily tell these files apart if they are different.
- Ex. Verifying checksum of a deployed war or jar file on server

#### Types of Hashing
There are many different types of hash algorithms such as RipeMD, Tiger, xxhash and more, but the most common type of hashing used for file integrity checks are MD5, SHA1 and CRC32.
- **MD5** - An MD5 hash function encodes a string of information and encodes it into a 128-bit fingerprint. MD5 is often used as a checksum to verify data integrity. However, due to its age, MD5 is also known to suffer from extensive hash collision vulnerabilities, but it’s still one of the most widely used algorithms in the world.
- **SHA1** – SHA1, developed by the National Security Agency (NSA), is a cryptographic hash function. Results from SHA1 are expressed as a 160-bit hexadecimal number. This hash function is widely considered the successor to MD5.
- **CRC32** – A cyclic redundancy check (CRC) is an error-detecting code often used for detection of accidental changes to data. Encoding the same data string using CRC32 will always result in the same hash output, thus CRC32 is sometimes used as a hash algorithm for file integrity checks.


## HashCode and Equals contract in Java
Each java object that we create has a parent class called Object which has default implementation for hashCode() and equals(). Therefore each java object inherits the default behaviour of hashCode() and equals().

#### Usage
hashCode() method is used to get unique integer representation for a object. This integer is a representation of memory address where this object will be stored. This is used to determine bucket location if we have to deal with hash tables whereas equals() method is used to check the equality of two objects. Default implementation simply checks the memory reference of two objects. 

Everything will work fine until you do not override these methods, but if you override one then ensure you override the second one as well. This is called contract between hashcode and equals. Both of them work hand in hand.

Let's take an example where we have a Employee class with equals() method implemented in it. Default implementation can be generated from IDE. Eclipse, IntellJ and NetBeans all three provide it.

```java
class Employee {
	String empId;
	String empDob;

	public Employee(String id, String dob) {
		empId = id;
		empDob = dob;
	}

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;
		Employee employee = (Employee) o;
		if (empDob != null ? !empDob.equals(employee.empDob) : employee.empDob != null) return false;
		if (empId != null ? !empId.equals(employee.empId) : employee.empId != null) return false;
		return true;
	}

}

```

Now, we have a main method in our class which calls another method to populate a cache of employees and then lookup one of the employee from the map.

```java
/**
* Main method to start the flow
* @param args
*/
public static void main(String[] args) {
	Map<Employee, String> cache = loadEmployeeCache();
	Employee lookUpKey = new Employee("101", "10111992");
	String empName = cache.get(lookUpKey);
	System.out.println(empName);
}

/**
* Method to load employee cache
*/
static Map<Employee, String> loadEmployeeCache() {
	Employee e1 = new Employee("100", "10111991");
	Employee e2 = new Employee("101", "10111992");
	Employee e3 = new Employee("102", "10111993");
	Map<Employee, String> cacheMap = new HashMap<>();
	cacheMap.put(e1, "Alice");
	cacheMap.put(e2, "Bob");
	cacheMap.put(e3, "Steve");
	return cacheMap;
}
```

We will get output as null because we have overridden the implementation of equals() method but not hashCode(). Let's add hashCode() method to Employee class and check the output again.

```java
@Override
public int hashCode() {
	int result = empId != null ? empId.hashCode() : 0;
	result = 31 * result + (empDob != null ? empDob.hashCode() : 0);
	return result;
}
```

Output will be Bob now since we have hashCode implementation as well. Let's try one more example. Remove the hashCode() implementation again and implement a set in main method.

```java
Set<Employee> employeeSet = new HashSet<>();
Employee e1 = new Employee("100", "10111990");
Employee e2 = new Employee("101", "10111992");
Employee e3 = new Employee("101", "10111992");
Employee e4 = new Employee("102", "10111991");
Employee e5 = new Employee("102", "10111991");
employeeSet.add(e1);
employeeSet.add(e2);
employeeSet.add(e3);
employeeSet.add(e4);
employeeSet.add(e5);
System.out.println(employeeSet);
```

Implement a toString() method in Employee class so that we can print the items.

```java
@Override
public String toString() {
	return "Employee [empId=" + empId + ", empDob=" + empDob + "]";
}
```

Since set cannot contain duplicate elements, when we execute this code our out put will have duplicate elements because we are missing hashCode() implementation. Add the hashCode() implementation back and check the output.
This time it will be unique elements printed.

Complete implementation of this program is [here](../Hashing/HashCodeAndEquals.java)

#### Important things to remember
```
- Always use same attributes of an object to generate hashCode() and equals() both.
- Whenever a.equals(b), then a.hashCode() must be same as b.hashCode().
- If you override one, then you should override the other.
- In case of ORM ensure you use getters and not field references.
```
	
Here is a nice article about it, 
[HashCode and Equals](http://www.javaranch.com/journal/2002/10/equalhash.html)	

### Iterator v/s Enumeration

Below are some significant differences between the two,

1. Both of these allows to traverse through the collection of elements. 
2. Both Iterator and Enumeration are interfaces, found in java.util package
    - Iterator has 3 methods, 
        - hasNext()
        - next()
        - remove()
    - Enumeration has 2 methods, 
        - hasMoreElements()
        - nextElement()
3. Iterator differs from Enumeration because iterator allows the caller to remove the elements while traversing through the collection. Enumeration doesn't have remove method. Enumeration is a legacy class and not all data structures supports it. for ex. Vector supports enumeration but a array list doesn't. On the other hand iterator is a standard class for traversal of elements.
4. Enumeration is older and exists from JDK 1.0, whereas iterator was introduced later.
5. Functionality of enumeration interface is duplicated in Iterator with some extra features on top of it.
6. Enumeration acts as read only interface, because it has methods only to traverse the elements. Whereas, iterator can modify the collection too, because remove method exits in it.
7. Iterator is more secure and safe, because it does not allow any other thread to modify the collection if one thread is working on the collection already. Iterator will throw ConcurrentModification Exception in this use case. 

#### When to use?

1. Use enumeration when only reading of elements is associated. 


### HashTable v/s HashMap

Before this, I would suggest to go through the topic Iterator V/S Enumeration here.
There are some significant factors that makes both of these data structures different

#### 1. Synchronization or Thread Safety
```
- This is the most important difference between the two. 
- Hash tables are synchronized and thread safe whereas Hash maps are not.
```

#### 2. Null keys and Null values
```
- HashMap allows one null key and multiple null values, whereas hash table doesn't allow neither null values not null keys.
```

#### 3. Iterating the Values
```
- Values in hash map are iterated using an iterator, whereas in hash table values are iterated using enumerator.
- Only Vector other then hash table uses enumerator to iterate through elements. 
```

#### 4. Fail Fast Iterator
```
- Iterator in hash map is fail fast iterator, whereas enumerator for hash table is not.
- If hash table is structurally modified at any time after the iterator is created other then iterators own remove method, it will throw Concurrent Modification exception.
- Structural modification means adding or removing elements from the collection.
```

#### 5. Super class and Legacy
```
- HashTable is a subclass of Dictionary, which is now obsolete from JDK 1.7
```

#### 6. Performance
```
- HashMap is much faster and uses less memory because it is not synchronized. 
```

#### Similarities:
```
- Order of elements cannot be guaranteed, because both of these work based on the hashing logic. use Linked Hash map for that
- Both of them comes from Map interface.
- Both provides constant time for performance for put and get methods, assuming that objects are distributed uniformly.
- Both works on the principle of hashing
```

#### When to use?
```
Avoid using HashTable, as they are obsolete now, ConcurrentHashMap has replaced it.
Single threaded apps - use HashMap
```
 


## Concurrent HashMap in Java

- ConcurrentHashMap is introduced as an alternative of Hashtable and provided all functions supported by Hashtable with an additional feature called "concurrency level".

- Concurrent HashMap allows multiple readers to read concurrently without any blocking. 
This is achieved by partitioning Map into different parts based on concurrency level and locking only a portion of Map during updates. Default concurrency level is 16, and accordingly Map is divided into 16 part and each part is governed with a different lock. This means, 16 thread can operate on Map simultaneously until they are operating on different part of Map. 

- This makes ConcurrentHashMap high performance despite keeping thread-safety intact.
Concurrent HashMap is thread safe i.e code can be accessed by a single thread at a time and it does not allow null keys or values.

#### Important concepts
- Since ConcurrentHashMap implementation doesn't lock whole Map, there is chance of read overlapping with update operations like put() and remove(). In that case result returned by get() method will reflect most recently completed operation from there start.

#### When to use?
- ConcurrentHashMap is best suited when you have multiple readers and few writers. If writers out number reader, or writer is equal to reader, than performance of ConcurrentHashMap effectively reduces to synchronized map or Hashtable. 

- Performance of CHM drops, because you got to lock all portion of Map, and effectively each reader will wait for another writer, operating on that portion of Map. 

- ConcurrentHashMap is a good choice for caches, which can be initialized during application start up and later accessed my many request processing threads.
 
- As javadoc states, CHM is also a good replacement of Hashtable and should be used whenever possible, keeping in mind, that CHM provides slightly weeker form of synchronization than Hashtable.

- Below table shows comparison of HashMap, Hashtable and ConcurrentHashMap,

<img width="588" alt="screen shot 2016-07-04 at 3 11 34 pm" src="https://cloud.githubusercontent.com/assets/3439029/16570432/bffc35f4-41f9-11e6-9ebf-e6873c2dc660.png">

#### ConcurrentHashMap issue with Long while using in a multi-threaded environment is shown below, 
Lets say I have a ConcurrentHashMap called ordersMap which holds name of some cities and orders from those cities, 

```java
static Map<String, Long> ordersMap = new ConcurrentHashMap<>();
```

In my main method, I will put some cities with 0 orders initially. I have a service with two threads, 
which will call a method to update the count of orders. 

```java
public static void main(String[] args) throws InterruptedException {
	ordersMap.put("Delhi", 0l);
	ordersMap.put("London", 0l);
	ordersMap.put("New York", 0l);
	ordersMap.put("Sydney", 0l);

	// Executor service with 2 threads
	ExecutorService service = Executors.newFixedThreadPool(2);
		
	// Submitting two tasks to service
	service.submit(new Runnable() {
		@Override
		public void run() {
			processOrders();
		}
	});
		
	service.submit(new Runnable() {
		@Override
		public void run() {
			processOrders();
		}
	});

	// Waiting for 1 second and then shutting down the service
	service.awaitTermination(1, TimeUnit.SECONDS);
	service.shutdown();
	System.out.println(ordersMap);
}
``` 

Let's implement processOrders() method,

```java
static void processOrders() {
	for (String city : ordersMap.keySet()) {
		for (int i = 0; i < 50; i++) {
			Long oldOrder = ordersMap.get(city);
			ordersMap.put(city, oldOrder + 1);
		}
	}
}
```

If you will run it once with two threads, most probably it will give you 100 for each of the city. 
Something like this
```
{Delhi=100, New York=100, London=100, Sydney=100}
```
But let's run it under some stress. I will run it from terminal for some time (maybe 10-15 runs)

<img width="740" alt="screen shot 2016-07-04 at 3 26 43 pm" src="https://cloud.githubusercontent.com/assets/3439029/16570539/d68af86c-41fb-11e6-9ead-8dc2e5bce303.png">

Here is the output, as we can see values are not consistent. 

<img width="354" alt="screen shot 2016-07-04 at 3 26 27 pm" src="https://cloud.githubusercontent.com/assets/3439029/16570544/e35d3014-41fb-11e6-9cb5-aae2fdcd628f.png">

Lets change our map to use AtomicLong instead of Long, 

```java
static Map<String, AtomicLong> ordersMap = new ConcurrentHashMap<>();
```

Put Values in Map, 
```java
ordersMap.put("Delhi", new AtomicLong());
ordersMap.put("London", new AtomicLong());
ordersMap.put("New York", new AtomicLong());
ordersMap.put("Sydney", new AtomicLong());
```

and change implementation of processOrders() method, 
```java
static void processOrders() {
	for (String city : ordersMap.keySet()) {
		for (int i = 0; i < 50; i++) {
			ordersMap.get(city).getAndIncrement();
		}
	}
}
```

Run the program again and observe the output, 

<img width="379" alt="screen shot 2016-07-04 at 3 27 00 pm" src="https://cloud.githubusercontent.com/assets/3439029/16570553/4cc9fdde-41fc-11e6-87b3-28a064a4fe55.png">

Complete program is [here](../Hashing/ConcurrentHashMapImplementation.java)