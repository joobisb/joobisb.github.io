+++
title = 'Postgres MVCC and Isolation Levels'
date = 2023-10-18T00:23:24+05:30
readTime = true
+++

Postgres uses a multiversion model for concurrency control unlike few other databases which uses locks. They call this **MVCC(Multi Version Concurrency Control)**.

This means that when a transaction is running, Postgres creates a snapshot of the database at the start of the transaction. This protects the transactions from viewing inconsistent data.

An important point to note is that MVCC read locks doesn't conflict with the locks acquired for writing. 
i.e. **reading never blocks writing and writing never blocks reading.**

### Why are we using MVCC or locks for that matter?

It is to prevent the problems that arise due to concurrent execution of transactions.

So what are those problems?

1. **Dirty reads**

![](/images/postgres-mvcc/dirty-reads.png)

It's pretty self-explanatory from the above diagram. A dirty read is when a transaction reads data that has not been committed by another transaction.

2. **Non-repeatable reads**

![](/images/postgres-mvcc/non-rr.png)

Here the difference is that the transaction is trying to read the same data again but it is not the same as the first read, cause T2 committed after T1's first read. This creates an inconsistency within T1. This is not a dirty read cause its reading the data that has been committed.


3. **Phantom reads**

![](/images/postgres-mvcc/phantom-reads.png)

This is a phantom read cause the collection of rows returned by the second query in T1 is different from the first.


### So how do we address these phenomenas?

*ANSI/ISO SQL* specifies four levels of transaction isolation to address the above problems.


 **Read Uncommitted**

This is the *lowest level* of isolation. It allows *dirty reads*, *non-repeatable reads*, and *phantom reads*. 

This is used where you need the absolute highest level of concurrency and can tolerate incosistent data.

 **Read Committed**

*Read Committed* is the default isolation level in Postgres.

It never sees the uncommitted data or changes committed during query executions. However, it does not prevent *non-repeatable reads* or *phantom reads*.


![](/images/postgres-mvcc/read-committed.png)

*Note:*

>  *SELECT* does see the effects of previous updates executed within the same transaction, even though they are not yet committed.

> If a transaction executing an UPDATE / DELETE / SELECT FOR UPDATE is already been updated by another concurrent transaction, the second transaction will wait for other to commit or roll back.


 **Repeatable Read**

This level addresses the problem of non-repeatable reads. However, it does not prevent *phantom reads*. 


![](/images/postgres-mvcc/repeatable-read.png)

 **Serializable**

This is the highest level of isolation. This emulates a serial transaction, as if they are been executed one after another, serially, rather than concurrently.


![](/images/postgres-mvcc/serializable.png)

Here when T1 it tries to update the row, it fails because of the conflict with the other transaction's update. Hence, the transaction needs to be retried.

*Note:*

>  *SELECT* does see the effects of previous updates executed within the same transaction, even though they are not yet committed.

> If a transaction executing an UPDATE / DELETE / SELECT FOR UPDATE is already been updated by another concurrent transaction, the second transaction will wait for other to commit or roll back. In case of rollback, the second transaction can proceed to change the row, but in case of commit, the serializable transaction will be rolled back with `ERROR: Can't serialize access due to concurrent update.`
