## MySQL's Logical Architecture

* The storage engines don’t parse SQL or communicate with each other; they simply respond to requests from the server.

#### Connection Management and Security

* Each client connection gets its own thread within the server process. The connection’s queries execute within that single thread, which in turn resides on one core or CPU. The server caches threads, so they don’t need to be created and destroyed for each new connection.
* MySQL server authenticates clients (applications) based on username, originating host and password
* Once authenticated, server verifies whether client has priveleges for each query

#### Optimization and Execution

* MySQL parses queries to create an internal structure, **the parse tree**, then applies optimizations
  * rewriting query
  * order which it will read tables
  * choosing which index to use
* You can pass hints to the optimizer through special keywords in the query
* You can ask server to explain optimization which can be a **reference point for reworking queries, schemas and settings for efficiency**
* Before parsing the query, server consults the query cache which stores **only** SELECT statements and their result sets

## Concurrency Control

Problem: more than one query needs to change data at the same time

#### Read/Write Locks

Shared locks / Exclusive locks
Read locks / Write locks

Read locks: shared, mutually non blocking, many clients can read from a resource at the same time and not interfere with each other
Write locks: exclusive, block read locks and other write locks, one client can write at a given time and prevent all other reads and writes

#### Lock Granularity

Don't lock the entire resource; lock on the part that contains the data you need to change. Minimize the amount of data you lock at any given time.

* Locks consumer resources
  * Getting a lock
  * Checking to see whether a lock is free
  * Releasing a lock
* Locking strategy: compromise between lock overhead and data safety
* MySQL offers locking policies and lock granularities

### Lock Strategies

#### Table Locks

* Most basic
* Lowest overhead
* Locks the entire table when the client wishes to write to a table - no other read and writes
* Read locks
* Write locks have higher priority than read locks

### Row Locks

* Offers greatest concurrency
* Greatest overhead
* Implemented in the storage engine (not the server)
* Server is unaware (dumb)

## Transactions

Goup of SQL queries that are treated as a single unit of work

* All or nothing
* ACID transactions
  * Atomicity - transaction indivisible unit of work
  * Consistency - DB moves from one consistent state to another
  * Isolation - results of transaction are invisible to other transactions until it is complete
  * Durability - permanent changes
* DB server with ACID transactions requires more CPY
* MySQL allows you to decide whether your application needs transactions and can use a nontransactional storage engine for some kinds of queries

#### Isolation Levels

* READ UNCOMMITTED
  * transactions can view the results of uncommitted transactions
  * rarely used
  * _dirty read_
* READ COMMITTED
  * default isolation for most systems but **NOT MySQL**
  * transaction can only see changes from other transactions that were already committed when that transaction ran
  * nonrepeatable read
* REPEATABLE READ
  * phantom reads
  * guarantees that any rows a transaction reads will "look the same" in subsequent reads within the same transaction
  * TODO: read more about this
  * MySQL's default transaction isolation level
* SERIALIZABLE
  * highest level of isolation
  * forces transactions to be ordered so that they can't possibly conflict
  * places a lock on every row it reads
  * rarely used

#### Deadlocks

Two or more transactions are mutually holding and requesting locks on the same resources.
 * Two separate transactions on the same resource (i.e. X WHERE id = 1)
 * Lock behavior and order are storage engine specific
 * Can't be broken without rolling back one of the transactions either partially or wholly

#### Transaction Logging

* write-ahead logging: storage engine can change its in-memory copy of the data, then write a record of the change to the transaction log; then, at a later time, a process can update the table on disk
* If tehre's a crash after the update is written to the transaction log but before the changes are made to the data itself, the storage engine can still recover the changes upon restart.

#### Transactions in MySQL

Two transactional storage engines: InnoDB and NDB Cluster

* AUTOCOMMIT
  * MySQL operates in AUTOCOMMIT mode by default
  * Automatically executes each query in a separate transaction

Storage Engines implement transactions themselves. You can't mix different engines in a single transaction. Nontransactional table changes won't be undone in a rollback. 

## Multiversion Concurrency Control

MVCC - keeps a snapshot of the data as it existed at some point in time; different transactions can see different data in the same tables at the same time

* InnoDB MVCC: stores with each row when it was created and when it was expired (or deleted) with version number of system at time each event occurred
  * Version number increments each time a transaction begins
  * Each transaction keeps its own record of the current system version, as of the time it began

## MySQL's Storage Engines

* database == schema
* When you create a table, MySQL stores the table definition in a .frm file with the same name as the table, i.e. MyTable.frm

#### InnoDB Engine

Default transactional storage engine for MySQL. Most important and broadly used engine overall.

* Designed for many short-lived transactions that usually complete rather than being rolled back.
* Stores its data in a series of one or more dta files that are known as tablespace
* Tablespace: black box that InnoDB manages all by itself; InnoDB can store each table's data and indexes in separate files
* Uses MVCC and defaults to REPEATABLE READ isolation level
