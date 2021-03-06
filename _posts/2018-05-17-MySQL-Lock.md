---
layout : post
title : MySQL Lock
categories : MySQL
---

MySQL 5.7 InnoDB

### Lock Type

  1. Shared(S) lock
  2. Exclusive(X) lock
  3. Intention shared(IS) lock - `select .. lock in share mode`
  4. Intention exclusive(IX) lock - `select .. for update`
  5. Record lock
  6. Gap lock
  7. Next-key lock
  8. Insert intention lock
  9. Auto-inc lock
  10. Predicate lock for spatial indexes
    
#### Lock compatibility
  
  ```
  T1                                                   T2
  SELECT * FROM A where id = 1 for update;
                                                       SELECT * FROM test.test where id = 1 for update;
  ```
  
  `select for update` of T2 blocked. But `select for update` sets `IX` lock and according to above diagram, we can see that
  `IX` is compatiable with `IX`, thus why there is lock wait in this case?  
  
#### Row-level lock

#### Table-level lock

  `LOCK TABLES` sets table locks which is set by MySQL layer. 
  
  InnoDB is aware of table locks if `innodb_table_locks = 1` (the default) and `autocommit = 0`, and the MySQL layer knows about row-level locks.

### Consistent Read

  Consistent read is the default mode in which InnoDB processes plain SELECT statements in `READ COMMITTED` and `REPEATABLE READ` 
  isolation levels. 

  In `REPEATABLE READ`, consistent read means the plain select statement only read the snapshot established by the first such read
  (first read snapshot, not the snapshot of first read finished) in that transaction or the changes made by earlier statements within 
  the same transaction. It will not see changes made by other committed transaction.
  
  In `READ COMMITTED`, each consistent read sees the changes made by other committed transaction or the 
  changes made by earlier statements within the same transaction. 
  
#### Exception

  Consistent read does not work over certain DDL statements:

  1. DROP TABLE, because MySQL cannot use a table that has been dropped and InnoDB destroys the table.

  2. ALTER TABLE, because that statement makes a temporary copy of the original table and deletes the original table when the temporary 
     copy is built. When you reissue a consistent read within a transaction, rows in the new table are not visible because those rows 
     did not exist when the transaction's snapshot was taken. In this case, the transaction returns an error: ER_TABLE_DEF_CHANGED, 
     “Table definition has changed, please retry transaction”.

#### Undo log

  A storage area that holds copies of data modified by active transactions. If another transaction needs to see the original data (as 
  part of a consistent read operation), the unmodified data is retrieved from this storage area.

  In MySQL 5.6 and higher, you can use the `innodb_undo_tablespaces` to create undo logs in undo tablespaces, optionally stored on 
  another storage device such as an SSD. In MySQL 8.0, undo logs reside in undo tablespaces by default.

  The undo log is split into separate portions, the insert undo buffer and the update undo buffer.
  
### Locking reads

  - SELECT ... LOCK IN SHARE MODE

    Sets `IS` lock on table first, then sets a shared mode lock on any rows that are read. 
    
    Other sessions can read the rows, but cannot modify them until your transaction commits. If any of these rows were changed by 
    another transaction that has not yet committed, your query waits until that transaction ends and then uses the latest values.

  - SELECT ... FOR UPDATE

    Sets `IX` lock on table first, then for index records the search encounters, locks the rows and any associated index entries, the 
    same as if you issued an UPDATE statement for those rows. 
    
    Other transactions are blocked from updating those rows, from doing `SELECT ... LOCK IN SHARE MODE`, or from reading the data in 
    certain transaction isolation levels. 
    
    Consistent reads ignore any locks set on the records that exist in the read view. (Old versions of a record cannot be locked; they 
    are reconstructed by applying undo logs on an in-memory copy of the record.)    

    > Locking of rows for update using SELECT FOR UPDATE only applies when autocommit is disabled (either by beginning transaction with 
    > START TRANSACTION or by setting autocommit to 0. If autocommit is enabled, the rows matching the specification are not locked. 
    
  **All locks set by LOCK IN SHARE MODE and FOR UPDATE queries are released when the transaction is committed or rolled back.**
  
### Secondary index

### Clustered index
    
### Locks set by different sql

  A locking read, an UPDATE, or a DELETE generally set record locks on every index record that is scanned in the processing of 
  the SQL statement. It does not matter whether there are WHERE conditions in the statement that would exclude the row. 
  
  InnoDB does not remember the exact WHERE condition, but only knows which index ranges were scanned. 
 
  If a secondary index is used in a search and index record locks to be set are exclusive, InnoDB also retrieves the corresponding 
  clustered index records and sets locks on them.

  If you have no indexes suitable for your statement and MySQL must scan the entire table to process the statement, 
  every row of the table becomes locked, which in turn blocks all inserts by other users to the table. 
   
  1. `SELECT ... FROM`
  
     It is a consistent read, reading a snapshot of the database and setting no locks unless the transaction isolation 
     level is set to SERIALIZABLE. 
     
     For SERIALIZABLE level, the search sets shared next-key locks on the index records it encounters. However, only an index record 
     lock is required for statements that lock rows using a unique index to search for a unique row.

  2. `SELECT ... FOR UPDATE` or `SELECT ... LOCK IN SHARE MODE`
   
     Locks are acquired for scanned rows, and expected to be released for rows that do not qualify for inclusion in the result set (for 
     example, if they do not meet the criteria given in the WHERE clause). 
     
     However, in some cases, rows might not be unlocked immediately because the relationship between a result row and its original 
     source is lost during query execution. For example, in a UNION, scanned (and locked) rows from a table might be inserted into a 
     temporary table before evaluation whether they qualify for the result set. In this circumstance, the relationship of the rows in 
     the temporary table to the rows in the original table is lost and the latter rows are not unlocked until the end of query 
     execution.

     `SELECT ... LOCK IN SHARE MODE` sets shared next-key locks on all index records the search encounters, 
     While `SELECT ... FOR UPDATE` sets an exclusive next-key lock on every record the search encounters. 
     
     However, for them, only an index record lock is required for statements that lock rows using a unique index to search for a unique row.

  3. `UPDATE ... WHERE ...` 
  
     Sets an exclusive next-key lock on every record the search encounters. However, only an index record lock is required for 
     statements that lock rows using a unique index to search for a unique row.

     When `UPDATE` modifies a clustered index record, implicit locks are taken on affected secondary index records. 
     The UPDATE operation also takes shared locks on affected secondary index records when performing duplicate check scans prior to 
     inserting new secondary index records, and when inserting new secondary index records.

  4. `DELETE FROM ... WHERE ...` 
  
     Sets an exclusive next-key lock on every record the search encounters. However, only an index record lock is required for 
     statements that lock rows using a unique index to search for a unique row.

  5. `INSERT`
  
     Sets an exclusive lock on the inserted row. This lock is an index-record lock, not a next-key lock (that is, there is no gap lock) 
     and does not prevent other sessions from inserting into the gap before the inserted row.

     Prior to inserting the row, a type of gap lock called an insert intention gap lock is set. This lock signals the intent to insert 
     in such a way that multiple transactions inserting into the same index gap need not wait for each other if they are not inserting 
     at the same position within the gap. 
     
     Suppose that there are index records with values of 4 and 7. Separate transactions that attempt to insert values of 5 and 6 each 
     lock the gap between 4 and 7 with insert intention locks prior to obtaining the exclusive lock on the inserted row, but do not 
     block each other because the rows are nonconflicting.

     If a duplicate-key error occurs, a shared lock on the duplicate index record is set. This use of a shared lock can result in deadlock should there be multiple sessions trying to insert the same row if another session already has an exclusive lock. This can occur if another session deletes the row.

  6. `INSERT ... ON DUPLICATE KEY UPDATE` 
  
     It differs from a simple INSERT in that an exclusive lock rather than a shared lock is placed on the row to be updated when a 
     duplicate-key error occurs. An exclusive index-record lock is taken for a duplicate primary key value. An exclusive next-key lock 
     is taken for a duplicate unique key value.

  7. `REPLACE`
  
     Like an INSERT if there is no collision on a unique key. Otherwise, an exclusive next-key lock is placed on the row to be replaced.

  8. `INSERT INTO T SELECT ... FROM S WHERE ...` 
  
     Sets an exclusive index record lock (without a gap lock) on each row inserted into T. If the transaction isolation level is `READ COMMITTED`, or `innodb_locks_unsafe_for_binlog` is enabled and the transaction isolation level is not `SERIALIZABLE`, InnoDB does the search on `S` as a consistent read (no locks). Otherwise, InnoDB sets shared next-key locks on rows from S. 
     
     InnoDB has to set locks in the latter case: During roll-forward recovery using a statement-based binary log, every SQL statement must be executed in exactly the same way it was done originally.

  9. `CREATE TABLE ... SELECT ...` 
  
     performs the SELECT with shared next-key locks or as a consistent read, as for INSERT ... SELECT.

     When a SELECT is used in the constructs `REPLACE INTO t SELECT ... FROM s WHERE ...` or `UPDATE t ... WHERE col IN (SELECT ... FROM s ...)`, InnoDB sets shared next-key locks on rows from table s.
  
**References**

[innodb-locks-set](https://dev.mysql.com/doc/refman/5.7/en/innodb-locks-set.html)

[innodb-locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)

[innodb-consistent-read](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)
