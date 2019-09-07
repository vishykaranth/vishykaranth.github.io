#Problem Sets @ Walmart Labs  

### Problem 01
~~~text
You have a database table that stores information about the price change of various product with time. It's append only table. Whenever the price of the product p1 is changed to c1 at time t1, a new row will be appended.
product_id price time
p1 10 4
p2 40 4
p1 20 5
p1 25 6
p2 55 7
...

Write an SQL query that will give the price of every product at time t1.
e.g. at time 6
product_id price
p1 25
p2 40

Note: Consider the case where two updates are made at the same time. Don't assume that the table is sorted by time.

SELECT PRODUCT_ID,PRICE FROM
(
SELECT S.*,
RANK() OVER (PARTITION BY PRODUCT_ID ORDER BY PRICE_TIME DESC) RANKING
FROM PRODUCT_SALES S
WHERE S.PRICE_TIME < SYSDATE-2
) WHERE RANKING=1;

SELECT p, c FROM price where t<=6 GROUP BY p;

~~~
