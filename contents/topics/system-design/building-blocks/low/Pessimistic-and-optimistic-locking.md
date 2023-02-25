## Pessimistic and optimistic locking
- Pessimistic and optimistic locking are two different approaches to manage concurrency control in database systems.
- Pessimistic locking is a technique in which a resource is locked from the moment it is read or accessed, to the moment it is released. 
  - Pessimistic locking is useful when there is a high probability of concurrent access to the same resource, and 
  - there is a risk of conflicts or inconsistencies occurring. 
  - By locking a resource, other processes or transactions that try to access it must wait until the lock is released, 
  - ensuring that the resource is accessed by only one process or transaction at a time. 
  - Pessimistic locking is more suitable for scenarios where the update operations are more frequent than the read operations.
- Optimistic locking is a technique in which a resource is not locked when it is read or accessed, and 
  - the database assumes that no conflicts will occur. 
  - When a transaction attempts to update the resource, 
    - the database checks if the resource has been modified by any other transaction in the meantime. 
    - If the resource has been modified, 
    - the transaction is aborted, and 
    - the changes are rolled back. 
  - Optimistic locking is more suitable for scenarios where read operations are more frequent than update operations.
- In summary, 
  - pessimistic locking is a more cautious approach that guarantees exclusive access to a resource, 
  - while optimistic locking allows concurrent access and 
    - assumes that conflicts will be rare. 
  - The choice of locking technique depends on the specific requirements of the system and the balance between performance and data consistency.


### Pessimistic locking

- Pessimistic locking is a technique used to ensure data consistency and prevent data conflicts in a concurrent environment. 
  - It is commonly used in situations where multiple users or processes may access the same data at the same time. 
- Here are some real-world examples of pessimistic locking:
  - Banking systems: 
    - When multiple users access their bank accounts online, 
    - the system uses pessimistic locking to ensure that only one user can modify their account details or transactions at a time. 
    - This is to prevent conflicting transactions, 
      - such as two users trying to transfer the same amount of money from their account to a third-party account simultaneously.
  - Inventory management systems: 
    - In a warehouse management system, 
      - multiple users may access the same inventory data simultaneously. 
      - Pessimistic locking is used to ensure that only one user can modify inventory data at a time to avoid conflicts such as two users trying to update the same inventory record simultaneously.
  - Reservation systems: 
    - Pessimistic locking is used in reservation systems, 
    - such as airline reservation systems, 
    - to ensure that a seat or a room can be reserved by only one user at a time. 
    - This ensures that no two users can book the same seat or room at the same time.
  - Healthcare systems: 
    - In healthcare systems, 
    - doctors and nurses may access a patient's medical records concurrently. 
    - Pessimistic locking is used to ensure that only one user can modify the patient's record at a time to prevent data conflicts and 
    - maintain data integrity.
- These are just a few examples of how pessimistic locking is used in real-world systems to ensure data consistency and prevent data conflicts in concurrent environments.


### Additional Notes 
- Transactional isolation is usually implemented by locking whatever is accessed in a transaction. 
  - There are two different approaches to transactional locking: 
    - Pessimistic locking and 
    - optimistic locking.
- The disadvantage of pessimistic locking is that a resource is locked from the time it is first accessed in a transaction until the transaction is finished, 
  - making it inaccessible to other transactions during that time. 
  - If most transactions simply look at the resource and never change it, 
  - an exclusive lock may be overkill as it may cause lock contention, and 
  - optimistic locking may be a better approach. 
  - With pessimistic locking, locks are applied in a fail-safe way. 
  - In the banking application example, an account is locked as soon as it is accessed in a transaction. 
    - Attempts to use the account in other transactions while it is locked will either result 
      - in the other process being delayed until the account lock is released, or 
      - that the process transaction will be rolled back. 
  - The lock exists until the transaction has either been committed or rolled back.
- With optimistic locking, 
  - a resource is not actually locked when it is first is accessed by a transaction. 
  - Instead, the state of the resource at the time when it would have been locked with the pessimistic locking approach is saved. 
  - Other transactions are able to concurrently access to the resource and the possibility of conflicting changes is possible. 
  - At commit time, when the resource is about to be updated in persistent storage, 
    - the state of the resource is read from storage again and compared to the state that was saved when the resource was first accessed in the transaction. 
    - If the two states differ, a conflicting update was made, and the transaction will be rolled back.
- In the banking application example, 
  - the amount of an account is saved when the account is first accessed in a transaction. 
  - If the transaction changes the account amount, the amount is read from the store again just before the amount is about to be updated. 
  - If the amount has changed since the transaction began, the transaction will fail itself, otherwise the new amount is written to persistent storage.
  