## Data partitioning (also known as sharding) 
- Sharding is a technique to break up a big database (DB) into many smaller parts. 
- It is the process of splitting up a DB/table across multiple machines to improve the manageability, performance, availability and load balancing of an application. The justification for data sharding is that, after
a certain scale point, it is cheaper and more feasible to scale horizontally by adding more machines than to
grow it vertically by adding beefier servers.
1. Partitioning Methods
There are many different schemes one could use to decide how to break up an application database into
multiple smaller DBs. Below are three of the most popular schemes used by various large scale applications.