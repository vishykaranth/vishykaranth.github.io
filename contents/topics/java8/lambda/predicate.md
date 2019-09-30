## Predicate 
- Predicate is functional interface. 
- It mean we can pass lambda expressions wherever predicate is expected. 
- For example one such method is filter() from Stream interface,  we can use stream and predicate to –
    - first filter certain elements from a group, and
    - then perform some operation on filtered elements.
- Predicate move your conditions (sometimes business logic) to a central place. This helps in unit-testing them separately.
- Any change need not be duplicated into multiple places. Java predicate improves code maintenance.
- The code e.g. “filterEmployees(employees, isAdultFemale())” is very much readable than writing a if-else block.    