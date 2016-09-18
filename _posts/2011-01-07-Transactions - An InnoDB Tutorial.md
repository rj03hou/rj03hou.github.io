---
layout: post 
title: "Transactions - An InnoDB Tutorial"
subTitle: 
heroImageUrl: 
date: 2011-1-7
tags: ["MySQL"]
keywords: 
---

非常给力的一篇文章，原文地址：http://mysqldump.azundris.com/archives/77-Transactions-An-InnoDB-Tutorial.html

InnoDB does transactions. Meaning: It collects statements working on InnoDB tables and applies them on COMMIT to all tables "at once". Either all of these statements inside one transaction succeed ("commit") or all of them fail ("rollback"), changing nothing.

By default, the database is in AUTOCOMMIT mode. Meaning: The server sees a virtual COMMIT command after each statement. You can disable autocommit completely, or you are starting an explicit transaction inside autocommit using the BEGIN statement.
<pre lang='sql'>
root on mysql.sock [innodemo]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> insert into kris ( d ) values ( "vier" );

Query OK, 1 row affected (0.00 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' zwei '

'  3 ' drei '

'  4 ' vier '

+----+------+

4 rows in set (0.00 sec)

root on mysql.sock [innodemo]> rollback;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' zwei '

'  3 ' drei '

+----+------+

3 rows in set (0.00 sec)
</pre>
Here we start a transaction with BEGIN, temporarily disabling autocommit. This is the recommended way to work. A row is inserted into the table. For our own connection this row is now visible. We then end the transaction (returning to autocommit), but not with COMMIT, but ROLLBACK instead. This undoes all changes inside the transaction, which we can verify with another SELECT.

**InnoDB stuff on disk**

<pre lang='sql'>
linux:/export/data/rootforum/data # ls -lh ib* innodemo

-rw-rw---- 1 mysql mysql   5M Jan  9 18:27 ib_logfile0

-rw-rw---- 1 mysql mysql   5M Jan  9 18:28 ib_logfile1

-rw-rw---- 1 mysql mysql  10M Jan  9 18:27 ibdata1

innodemo:

total 112K

-rw-rw---- 1 mysql mysql   61 Jan  9 18:03 db.opt

-rw-rw---- 1 mysql mysql 8.4K Jan  9 18:28 kris.frm

-rw-rw---- 1 mysql mysql  96K Jan  9 18:28 kris.ibd
</pre>
Each InnoDB installation has at least one datafile, commonly called ibdata1 and at least two redo logfiles named ib_logfile0 and ib_logfile1. Position, size, number and name of these files can be configured with some freedom.

Our InnoDB uses the configuration setting innodb_file_per_table = 1. Meaning: Our data is being stored in .ibd files next to the .frm files inside the schema directory. In our example this comes down to $datadir/innodemo/kris.ibd. The extension .ibd is for InnoBase Data.

Despite the fact that actual data is being stored in .ibd files we still need the central ibdata file and at least two redo logs. The ibdata file houses the InnoDB shadow data dictionary, a copy of the table definitions from the .frm files. It also contains the undo log, which we will visit later in more detail.

The redo logfiles log all data modifying operations.

Unlike MyISAM you cannot move around .ibd files at the filesystem level - do not move or rename these files from outside the database. Also, do not copy them into another instance of MySQL. In a best case scenario InnoDB detects this and rejects the file. The worst case is data loss.

If you happen to operate innodb with innodb_file_per_table = 0 there is only one ibdata file and it will contain all data. The schema directory then has only .frm files.

Space inside an ibdata file can be freed and reclaimed, but the file itself will never shrink. If you are changing an instance that has been operating with innodb_file_per_table = 0 to innodb_file_per_table = 1, you may export data from inside the ibdata file to individual .ibd files using "ALTER TABLE t ENGINE=InnoDB". This will remove the table from the ibdata file and free the space inside you ibdata. But still your ibdata will not shrink. The only way to get rid of the large ibdata file is to dump the database and then recreate a new instance from the dump.

The amount of free space inside your ibdata or .ibd file is show as a comment in the output of SHOW TABLE STATUS for each table. The numbers are of course always multiples of 16K, the InnoDB page size.

It is possible to have more than one ibdata file, but it is not possible at all to control which tables or part of a table will be inside which ibdata file. If you set innodb_file_per_table, each table will be completely self-contained inside its own .idb file (but that information is of very limited use).

**The Undo Log**

In InnoDB, each table has two hidden columns, a transaction number and a pointer to the previous version of that row inside the undo log, called the rollback pointer.

Whenever we are chaning a row in InnoDB the old version of that row including its old transaction number is being copied into the undo log. The new version of the row including a new transaction number is installed inside the .ibd file. The rollback pointer of the new row will point to the old version of that row inside the undo log.
<pre lang='sql'>
root on mysql.sock [innodemo]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> update kris set d = "one" where id = 1;

Query OK, 1 row affected (0.01 sec)

Rows matched: 1  Changed: 1  Warnings: 0

root on mysql.sock [innodemo]> select * from kris;

+----+------+------+          +----+------+------+

' id ' d    ' txn# '          ' id ' d    ' txn# '

+----+------+------+          +----+------+------+

'  1 ' one  '   2  ' ------>  '  1 ' eins '   1  '

'  2 ' zwei '   1  '          +----+------+------+

'  3 ' drei '   1  '

+----+------+------+

3 rows in set (0.00 sec)
</pre>
The example above shows these hidden columns and the undo log entry for illustrative purposes only. I have been faking this by hand, there is no statement that produces such output.

The linked list of row versions can span multiple versions of a row, that is, each entry inside the undo log can point to an even older version of that row and so on. If the undo log is never purged, it will eventually go back to the time when the database server had been installed. But fortunately there is the purge thread which will automatically collect all rows from the undo log that are older than the oldest active transaction inside the system. This prevents infinite growth of the undo log.

Copying a version of a row from the table into the undo log and the linked list of row versions are called MVCC, Multiversion Concurrency Control.

The way InnoDB implements MVCC is optimized for COMMIT. When a row is being committed, nothing needs to be done. The transaction is marked as committed, the rollback pointers are kept and all is fine.

<pre lang='sql'>
root on mysql.sock [innodemo]> rollback;

Query OK, 0 rows affected (0.01 sec)

+----+------+------+          +----+------+------+

' id ' d    ' txn# '          ' id ' d    ' txn# '

+----+------+------+ ROLLBACK +----+------+------+

'  1 ' eins '   1  ' <------- '    '      '      ' '  2 ' zwei '   1  '          +----+------+------+ '  3 ' drei '   1  ' +----+------+------+ 3 rows in set (0.00 sec) A rollback is not so easy: We have quite some work to do. For each row affected by the rollback we have to retrieve the data from the undo log and put it back into place to undo our change to the original table. If we had been affecting a lot of rows with our transaction this can take quite some time, because the undo log is organized by row and not by page. It is adviseable not to make your transactions to large: In many cases a transaction size of 1.000 to 10.000 rows is quite fast enough for bulk data load but also acceptable in terms of rollback time. Transaction Isolation Level Read Uncomitted When we are looking at our transaction example from above from the point of view of a second connection it may be that we can observe the change or not. The first connection does: CODE: root on mysql.sock [innodemo]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> update kris set d = "one" where id = 1;

Query OK, 1 row affected (0.00 sec)

Rows matched: 1  Changed: 1  Warnings: 0

We are leaving this open not doing either COMMIT or ROLLBACK. In a second window we are opening a connection to the database, checking what is visible:

CODE:
root on mysql.sock [(none)]> use innodemo;

Database changed

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' zwei '

'  3 ' drei '

+----+------+

3 rows in set (0.00 sec)
</pre>
The first thing we observe is that our read access does not hang or wait despite the fact that a transaction is in progress. In MVCC, reading and writing operations never block each other. This is a big difference to MyISAM and one reason why InnoDB is the recommended storage engine for use cases with a high write concurrency - even if it has to make a copy of each row it changes.

We are seeing the old value of the row with id = 1. We do know from above that the .ibd file alreay contains the changed value, but we do not see it. This indicates that the database has been following the rollback pointer into the undo log and retrived an old version of that row for us. This is different from the "inside" view the first connection gets of its own transaction, where we can see our own changes immediately even before we COMMIT.

But we can change our second external connection in a way that it will see this uncomitted data as well. For this we change the TRANSACTION ISOLATION LEVEL to READ UNCOMMITTED.
<pre lang='sql'>
root on mysql.sock [innodemo]> set transaction isolation level read uncommitted;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' one  '

'  2 ' zwei '

'  3 ' drei '

+----+------+

3 rows in set (0.00 sec)
</pre>
We learn: The writing connection will always perform the same amount of work. It will always make a copy of the data it is changing and put it into the undo log. This is necessary to enable rollback. So multiple versions of the row always exist.

A reading connection can choose on the fly which version of the data shall be visible to it. Each reader can chose its own isolation level without affecting other connections, and each reader can change its own isolation level as it wishes, because all versions of the data will always be present.

In READ UNCOMITTED mode we always look at data inside the .ibd file and never follow any rollback pointers. We may see data that is not yet committed and may never be committed. We are looking at a hypothetical version of reality that may never be realized. For applications that is often not the desired mode of operation.

Transaction Isolation Level Read Comitted

Changing the TRANSACTION ISOLATION LEVEL to READ COMMITTED a reader will not see uncommitted rows in an .ibd file. For all rows that have not been changed by the uncommitted transaction we get data from the .ibd file as before, but for all rows that hold uncommitted data InnoDB follows the rollback pointer one step into the past into the undo log, fetching the newest committed version of that row. Thus we will always see data that is "really there" and all those hypothetical versions of reality that are part of READ UNCOMMITTED are being filtered out.

This is a lot better, but still not good enough. It may be that we are looking at the same data twice, and the data may be changing between these two looks. For example we might have an application which maintains a counter inside the database. That is, inside the application we might have a sequence of commands such as "begin; update kris set d = d + 1 where id = 2; commit;". This will increment a counter for the row id = 2

If in a second connection we are looking at the table with "READ COMMITTED" we will see the counter move for that row id = 2. That is the case even if our second connection is inside its own transaction - the sequence BEGIN-SELECT-SELECT-COMMIT, a read-only transaction, has no meaning in READ COMMITTED.

<pre lang='sql'>
root on mysql.sock [innodemo]> set transaction isolation level read committed;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' zwei '

'  3 ' drei '

+----+------+

3 rows in set (0.01 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' 1    '

'  3 ' drei '

+----+------+

3 rows in set (0.00 sec)

root on mysql.sock [innodemo]> select * from kris;

+----+------+

' id ' d    '

+----+------+

'  1 ' eins '

'  2 ' 2    '

'  3 ' drei '

+----+------+

3 rows in set (0.00 sec)

root on mysql.sock [innodemo]> commit;

Query OK, 0 rows affected (0.00 sec)
</pre>
**Transaction Isolation Level Repeatable-Read**

This is precisely what is different once we move to TRANSACTION ISOLATION LEVEL REPEATABLE READ. The moment we start a transaction with BEGIN our view of the database becomes an unchanging snapshot which is held until we end our transaction. In REPEATABLE READ there is such a thing as a read-only transaction.

Internally this is being done by following the rollback pointer for a row not just one step, but as long until we find the newest version of that row that had been visible for the reader when the reader started its transaction. So while writing connections change the database again and again more and more old versions of the row go into the undo log where they are being archived.

While READ COMMITTED only dives just a single step into the rows past, it may be that many versions of a rows past have to be skipped for a certain reader, diving into the undo log to an arbitrary depth. Thus the lifetime of entries in the undo log is no longer fixed and the undo log can grow and shrink. InnoDB has a global purge thread (which is not shown in SHOW PROCESSLIST). The purge thread looks at the list of currently active transactions and determines which is the oldest entry inside the undo log that is still needed by the oldest transaction. It will then delete all entries from the undo log that are even older than this.

This may mean that a long running transaction effectively stalls the purge thread. If your database for example runs "mysqldump --single-transaction" to export a large database it may be that this "single transaction" will take many minutes or even hours to finish. The purge thread cannot proceed during this time. If you have many writes to this database during this time your undo log will grow, potentially even grow a lot.

As said before the undo log is always part of your "ibdata" file, even if you are running with innodb_file_per_table = 1. So even on such servers the ibdata file can grow - 256M to 1G are completely normal.

If an ibdata file is too small and cannot grow because the innodb_data_file_path is defined without "autoextend" or because the disk is full this may lead to error messages that are hard to debug ("table full" despite the fact that a lot of space is available) and it will lead to unwanted rollbacks.

Transaction Isolation Level SERIALIZEABLE and SELECT ... FOR UPDATE

While REPEATABLE READ will serve all our readers needs we are still missing a mechanism for dealing with READ-MODIFY-WRITE cycles correctly. A RMW cycle is an access where the application is reading data, changes it inside the application and then put the changed data back into the database. For this to happen consistently we must make sure that the data inside the database will not change after we have read it into the application. Otherwise we will end up with an ugly race condition when two connections are trying to change the same row at the same time.

We achive this consistency by using a transaction in which we are reading data using a SELECT ... FOR UPDATE statement. This is a select statement that performs a normal read but generates locks like an update statement. Meaning in our case that we will get exclusive locks (X locks) on all rows read by the select through an index. The locks are held until the transaction ends. "FOR UPDATE" essentially serializes all transactions on specified rows through these locks.

And as usual there are a few tricks things to take care of:

The first tricky thing is that the locks are generated using the index. When we are looking at an EXPLAIN plan with "using where" this often means that the index selects more lines than we will see in the result set. This is because there is an additional constraint inside the WHERE clause which further reduces the result set generated by the index accesses. But because locks are generated by the index accesses it may also mean that we generate more locks than just for the rows visible in the result set of our SELECT ... FOR UPDATE statement. Maybe a LOT more locks.

To demonstrate that we create a table with two columns a and b. a is our primary key and b is not indexed. Running a SELECT ... FOR UPDATE with WHERE a > ... AND b = ... we will see that all rows found by the a-condition are locked - even those where the b-condition is not true.
<pre lang='sql'>
--

-- Create table

--

root on mysql.sock [kris]> create table t (

a integer not null,

b integer not null

) engine = innodb;

Query OK, 0 rows affected (0.16 sec)

--

-- Generate data

--

root on mysql.sock [kris]> insert into t values ( rand() * 100000, rand() * 10);

Query OK, 1 row affected (0.01 sec)

root on mysql.sock [kris]>  insert into t select rand() * 100000, rand() * 10 from t;

Query OK, 1 row affected (0.01 sec)

Records: 1  Duplicates: 0  Warnings: 0

...

root on mysql.sock [kris]>  insert into t select rand() * 100000, rand() * 10 from t;

Query OK, 4096 rows affected (0.26 sec)

Records: 4096  Duplicates: 0  Warnings: 0

-- We still have duplicates preventing

-- creation of a primary key

root on mysql.sock [kris]> create table dup as select a from t group by a having count(a) > 1 ;

Query OK, 300 rows affected (0.08 sec)

Records: 300  Duplicates: 0  Warnings: 0

root on mysql.sock [kris]> delete from t where a in ( select a from dup );

Query OK, 608 rows affected (5.65 sec)

root on mysql.sock [kris]> alter table t add primary key (a);

Query OK, 7584 rows affected (0.40 sec)

Records: 7584  Duplicates: 0  Warnings: 0

root on mysql.sock [kris]> drop table dup;

Query OK, 0 rows affected (0.00 sec)

--

-- the a-condition selects 62 rows

--

root on mysql.sock [kris]>  select a, b from t where a > 99000;

...

' 99490 '  8 '

...

62 rows in set (0.00 sec)

--

-- The actual demo: inside a transaction

-- do a SELECT ... FOR UPDATE

--

root on mysql.sock [kris]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [kris]>  select a, b from t where a > 99000 and b = 10;

+-------+----+

' a     ' b  '

+-------+----+

' 99839 ' 10 '

' 99970 ' 10 '

+-------+----+

2 rows in set (0.00 sec)
</pre>
We leave this hanging with no COMMIT or ROLLBACK. Inside a second connection we now try to change the pair (99490, 8). We see: The statement hangs due to an X-lock on that row.
<pre lang='sql'>
root on mysql.sock [kris]> update t set b = 502 where a = 99490;

... hang ...

By creating an additional INDEX (a,b) and forcing its use our locking improves a lot: Only the two records (99839, 10) and (99970, 10) get X-locks and the second parallel update does not wait.
CODE:
root on mysql.sock [kris]> begin;

Query OK, 0 rows affected (0.00 sec)

root on mysql.sock [kris]>  select * from t force index (a) where a > 99000 and b = 10;

+-------+----+

' a     ' b  '

+-------+----+

' 99839 ' 10 '

' 99970 ' 10 '

+-------+----+

2 rows in set (0.00 sec)

root on mysql.sock [kris]> explain select * from t force index (a) where a > 99000 and b = 10\G

*************************** 1. row ***************************

id: 1

select_type: SIMPLE

table: t

type: range

possible_keys: a

key: a

key_len: 4

ref: NULL

rows: 63

Extra: Using where; Using index

1 row in set (0.00 sec)

The second connection:
CODE:
root on mysql.sock [kris]> update t set b = 503 where a = 99490;

Query OK, 1 row affected (0.09 sec)

Rows matched: 1  Changed: 1  Warnings: 0
</pre>
The way X-locks are generated through the index has far reaching consequences: We absolutely have to double check each and every query plan of our SELECT ... FOR UPDATE statements. Not using proper indices here has detrimental effects on our performance. An ALL or INDEX inside the type column of such an EXPLAIN would indicate an index scan - and because we scan the entire index this is just a very expensive and slow way to create a table lock from row locks.

Another tricky thing is that InnoDB does not just lock rows, but also the gap behind a row in order to simplify its implementation of REPEATABLE READ. This "next key locking" can be turned of using a configuration statement with the unlikely name "innodb_locks_unsafe_for_binlog". If you turn this on, InnoDB will generate only row locks and not next key locks that span the gap to the next record as well.

At the transaction isolation level SERIALIZEABLE the system will behave just as with REPEATABLE READ, but it will execute each and every SELECT in a way as if it was a SELECT ... FOR UPDATE. Meaning: Each and every SELECT statement will create X-locks as if it were an UPDATE statement, each read will lock like a write. The will ultimately lead to a situation where even concurrent reads (locking like writes) will lock each other out. This is a behaviour even worse than MyISAM.

The transaction isolation level SERIALIZEABLE is unnecessary: It is never needed as long as the SQL inside the application will create proper locks using ... FOR UPDATE. Only applications that do not do this and cannot be fixed need SERIALIZEABLE.