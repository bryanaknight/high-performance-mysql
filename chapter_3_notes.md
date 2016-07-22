## Profiling Server Performance

**Performance is how long it takes to complete a task. It is response time, measured by time per query**

* A database server's purpose is to execute SQL statements (queries)

**Performance optimization is reducing response time as much as possible for a given workload**

^^That does not necessarily mean reducing CPU consumption. Sometimes optimization includes increasing CPU consumption.

* If you upgrade MySQL with a new version of InnoDB, CPU may increase but what you should look at is query response time to know if an ugrade is an improvement.
* Increased CPU is a symptom, not a goal.
* More queries per second is a throughput optimization that is a result of performance optimization. The faster the queries run, the more queries complete within a unit of time.
* To optimize queries, we have to measure where the time is spent on the query.
* Most of the time optimizing queries should be spent measuring ~90% and ~10% changing query

### Profiling MySQL Queries

* Profile whole server to find out which queries contribute to most of its load and then drill down into specific queries to measure which subtasks contribute most to their response times.
* Reducing load on the server will make all queries faster by reducing contention for shared resources **collateral benefit**

#### MySQL Slow Query Log (queries on server)
* Set the `long_query_time` server variable to zero will capture all queries
* Log analysis tool: pt-query-digest

#### Profiling a Single Query

* `SHOW PROFILE FOR QUERY X` after `SET profiling = 1`, do query, `SHOW PROFILES` (to get query id)
* `SHOW STATUS` after `FLUSH STATUS`

**`\G` at the end of a MySQL command for tabular view**
