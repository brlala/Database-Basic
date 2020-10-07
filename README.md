# Database-Basic
A-tomicity C-onsistency I-solation D-urability

## Indexing
### Indexing in Database
Indexing will create an index table whcih will have an underlying pointer to the document in the table, this allow improved searching such as using binary trees, binary search etc.
![Database Index](https://i.imgur.com/AwYydmV.png)

We introduce a different data index instead of ordering the true data because there may be in occassion where we want to order the datas differently(based on 2 columns or more). Managing the constant reordering of database table is very time consuming.

In NoSQL database, index are mostly created as composite index hence we cannot avoid the redundancy/discouraging the use of normalization.

The key chose must be a uniform key such that the data is distributed uniformly. We can assume that the has function provided by database is already a uniform hash algorithm.

### Underlying detail
The index is usually a B-Tree, which means that if the index is (type, status, user_id) the database can normally still use it to search for (type, status) because it's the first part of the combined index. But that would not be the case if you use (status, user_id).
![data](https://i.imgur.com/SBVb3HO.png) ![B+tree on database pages](https://i.imgur.com/wJMD0kn.png)

### Index Clustering
Clustered index means that the rows are stored physically closed to each other on the disk. Therefore there can only be one clustered index. Each new index will increase the time it takes to write new records because the rows have to be rearranged. Read on clustered index will generally be faster because we can directly read from the table(already sort close to each other).

## Data Replication (alternatively we can do data sharding)
Advantages: backup, scale out read operations
Master-Slave pattern is most commonly used(must be odd video)
1. Never allowed servers to send write commands to slave
2. Propagate the to query to a master server
3. Both Master nodes, this is an anti-pattern because in a distributed environment, if the connection breaks, both A and B will think themselves as master servers and will both write different data.

Solution is distributed consensus: A way which multiple nodes agree on a value. Some common protocols are 2Phase Commit, 3 Phase COmmit, Multi Version Concurrency Control(MVCC), SAGA.

MVCC, it keeps multiple copies of the same data, depending on the setting, it can allow dirty reads, no phantom reads etc based on setting.
SAGA, it's a really long transaction which for any point that may fail, for example if you have multiple withdraws from your bank account, it will first lock the total amount, then break it into multiple small transaction. At the end then it will decide if the SAGA is completed or all should be rolled back.

## ACID Properties
*Commit*: After all instructions of a transaction are successfully executed, the changes made by transaction are made permanent in the database.
*Rollback*: If a transaction is not able to execute all operations successfully, all the changes made by transaction are undone.

Properties of a transaction
*Atomicity*: As a transaction is set of logically related operations, either all of them should be executed or none. A debit transaction discussed above should either execute all three operations or none.
    - Abort: If a transaction aborts, changes made to database are not visible.
    - Commit: If a transaction commits, changes made are visible.
*Consistency*: Database must be consistent before and after the trnasaction.
*Isolation*: Result of a transaction should not be visible to others before transaction is committed (Dirty reads)
    - This property ensures that the execution of transactions concurrently will result in a state that is equivalent to a state achieved these were executed serially in some order.
*Durable*: Once database has committed a transaction, the changes made by the transaction should be permanent.
    - persist even if a system failure occurs.
