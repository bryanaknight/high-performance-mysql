## Benchmarking MySQL

#### Benchmarking Strategies

* Full-stack: Benchmark application as a whole
* Single-component: Benchmark isolated MySQL

* Throughput: Number of transactions per unit of time (second)
* Response time/latency: total time a task requires (milliseconds, seconds, minutes); common to use percentiles
* Concurrency: How many simultaneous requests are running at any given time
  * Working Concurrency: # of threads or connections doing work simultaneously
* Scalability: An ideal system should get twice as much work done (throughput) when you double the # of workers trying to complete tasks

#### Designing and Planning a benchmark

* Take a snapshot of your production dataset
* Log production queries for a subset on time
  * You can enable MySQL's query log, be sure to recreate separate threads for each connection when replaying it
* Metrics to measure: CPU usage, disk I/O, network traffic statistics, coutners from `SHOW GLOBAL STATUS;`, etc.
