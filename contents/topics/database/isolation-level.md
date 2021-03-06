## Isolation Level 

- READ UNCOMMITTED: 
    - **UserA will see the change made by UserB.** 
    - This isolation level is called dirty reads, which means that read data is not consistent with other parts of the table or the query, and may not yet have been committed. 
    - This isolation level ensures the quickest performance, as data is read directly from the table’s blocks with no further processing, verifications or any other validation. 
    - The process is quick and the data is as dirty as it can get.
- READ COMMITTED: 
    - **UserA will not see the change made by UserB.** 
    - This is because in the READ COMMITTED isolation level, the rows returned by a query are the rows that were committed when the query was started. 
    - The change made by UserB was not present when the query started, and therefore will not be included in the query result.
- REPEATABLE READ: 
    - **UserA will not see the change made by UserB.** 
    - This is because in the REPEATABLE READ isolation level, the rows returned by a query are the rows that were committed when the transaction was started. 
    - The change made by UserB was not present when the transaction was started, and therefore will not be included in the query result.
    - This means that “All consistent reads within the same transaction read the snapshot established by the first read” 
        - (from MySQL documentation. See http://dev.mysql.com/doc/refman/5.1/en/innodb-consistent-read.html).
- SERIALIZABLE: 
    - This isolation level specifies that all transactions occur in a completely isolated fashion, 
        - meaning as if all transactions in the system were executed serially, one after the other. 
    - The DBMS can execute two or more transactions at the same time only if the illusion of serial execution can be maintained.
    - In practice, SERIALIZABLE is similar to REPEATABLE READ, but uses a different implementation for each database engine. 
    - In Oracle, the REPEATABLE READ level is not supported and SERIALIZABLE provides the highest isolation level. 
    - This level is similar to REPEATABLE READ, but InnoDB implicitly converts all plain SELECT statements to “SELECT … LOCK IN SHARE MODE.