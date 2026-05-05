# CS 544 Final Exam Question Bank by Topic

Compiled from past CS 544 exams: Spring 2024 (Exam 2), Fall 2024 (Exam 3), Spring 2025 (Exam 3), Fall 2025 (Exam 3), Spring 2026 (Exam 1, Exam 2, Exam 3). Organized by the final exam topic distribution.

## Topic Distribution (Final)

| Topic | Target |
|-------|--------|
| HDFS | 4 |
| Spark | 5 |
| Cassandra | 4 |
| Kafka | 5 |
| BigQuery | 5 |
| Other Cloud | 4 |
| Minor systems (MySQL/MapReduce/HBase) | 3 |
| Review | 10 |
| **Total** | **40** |

Tags used below: **S25** = Spring 2025 Exam 3, **F25** = Fall 2025 Exam 3, **F24** = Fall 2024 Exam 3, **Sp24** = Spring 2024 Exam 2, **Sp26-E1** = Spring 2026 Exam 1 (versions A/B/C/D), **Sp26-E2** = Spring 2026 Exam 2, **Sp26-E3** = Spring 2026 Exam 3.

---

## HDFS

### S25 Q4 — Client connection
To connect to an HDFS cluster, what does a client need, at a minimum?
- (A) address of any DataNode
- (B) addresses of all the DataNodes
- (C) address of the NameNode
- (D) addresses of NameNode and all DataNodes

<details>
<summary><b>Show answer</b></summary>

**(C) address of the NameNode.** The NameNode holds all metadata; it tells the client which DataNodes hold which blocks. Clients are then redirected to DataNodes for actual block I/O.

</details>

### S25 Q11 — Load balancing
In an HDFS cluster, load is poorly balanced across DataNodes. What is most likely to help improve balance?
- (A) using smaller blocks
- (B) using bigger blocks

<details>
<summary><b>Show answer</b></summary>

**(A) smaller blocks.** Smaller blocks = more blocks per file = finer-grained distribution across DataNodes. Big blocks concentrate file weight on fewer nodes.

</details>

### S25 Q24 — Replication I/O
You write 50 MB to a 3x replicated file in HDFS, then later read it back. How much data will be read and written to disks across the cluster?
- (A) 50 MB written, 150 MB read
- (B) 150 MB written, 50 MB read
- (C) 50 MB written, 50 MB read
- (D) 150 MB written, 150 MB read

<details>
<summary><b>Show answer</b></summary>

**(B) 150 MB written, 50 MB read.** Writes hit every replica → 50 × 3 = 150 MB to disk. Reads pull from one replica only → 50 MB.

</details>

### F25 Q4 — Client network bytes
A client writes a 80 MB file to HDFS with 1x replication. The block size is 16 MB. How much data does the client send over the network?
- (A) 5 MB  (B) 16 MB  (C) 80 MB  (D) 112 MB  (E) 400 MB

<details>
<summary><b>Show answer</b></summary>

**(C) 80 MB.** With pipelined writes, the client sends each byte exactly once regardless of replication or block size. 1x replication means the first DataNode keeps the only copy; client just sends 80 MB total.

</details>

### F25 Q11 — Detecting live DataNodes
How does a NameNode determine which DataNodes are live in the cluster?
- (A) gossip  (B) leader election  (C) heartbeats  (D) pipelines  (E) checksums

<details>
<summary><b>Show answer</b></summary>

**(C) heartbeats.** DataNodes send periodic heartbeats to the NameNode. Missing heartbeats → NameNode marks the node dead and re-replicates affected blocks.

</details>

### F25 Q24 — Hot file
You have an HDFS file, F, that rarely changes, but it is very popular: clients read F so often that DataNodes storing blocks of the file cannot keep up with requests. It will not be catastrophic if F is lost, because you can just execute a MapReduce job to regenerate the contents of F as needed. What would be better, from a performance perspective?
- (A) decrease replication factor for F
- (B) increase replication factor for F

<details>
<summary><b>Show answer</b></summary>

**(B) increase replication factor.** More replicas = more DataNodes that can serve reads in parallel. Loss isn't catastrophic, so the durability tradeoff is fine.

</details>

### F24 Q9 — Popular file
A single HDFS file is being read by many different clients, and HDFS is having trouble keeping up. What is most likely to help?
- (A) disable pipelined writes
- (B) increase the replication factor
- (C) decrease the replication factor

<details>
<summary><b>Show answer</b></summary>

**(B) increase the replication factor.** Same logic — more replicas distributes read traffic. Pipelined writes are about writes, not reads.

</details>

### F24 Q11 — Pipelined writes
Which system uses pipelined writes to send data to all the workers that will store a new piece of data?
- (A) HDFS  (B) Spark  (C) Cassandra  (D) Kafka

<details>
<summary><b>Show answer</b></summary>

**(A) HDFS.** Client → DN1 → DN2 → DN3 in a chain. Cassandra writes go to a coordinator which fans out. Kafka uses leader-fetch (followers pull). Spark doesn't replicate.

</details>

### F24 Q24 — Failure detection
What technique does HDFS use to DETECT DataNode failures?
- (A) partitioning  (B) replication  (C) heartbeats  (D) block maps  (E) hashing

<details>
<summary><b>Show answer</b></summary>

**(C) heartbeats.** Replication is what HDFS uses to *recover* from failures, but heartbeats are how it *detects* them.

</details>

### F24 Q29 — Leader/follower replication
Which system(s) have a leader/follower approach to replication?
- (A) only HDFS  (B) only Spark  (C) only Cassandra  (D) only Kafka  (E) both HDFS and Cassandra

<details>
<summary><b>Show answer</b></summary>

**(D) only Kafka.** Kafka has explicit partition leaders + followers. HDFS uses pipelined writes (peer-to-peer through the pipeline, not leader/follower). Cassandra is leaderless / peer-to-peer with quorum-based consistency.

</details>

### Sp24 Q8 — Pipelined writes math
A client is writing 5 MB of data to a 4x replicated HDFS file. Assuming pipelined writes, how much data does the client send over the network?
- (A) 4 MB  (B) 5 MB  (C) 9 MB  (D) 20 MB

<details>
<summary><b>Show answer</b></summary>

**(B) 5 MB.** The client sends to one DataNode; subsequent forwards happen between DataNodes, not from the client. Replication factor doesn't change client network bytes.

</details>

### Sp24 Q26 — Similar system
Which of the following is most similar to HDFS?
- (A) Colossus  (B) BigTable  (C) BigQuery  (D) HBase  (E) Dynamo

<details>
<summary><b>Show answer</b></summary>

**(A) Colossus.** Both are distributed file systems for large block storage. Colossus is Google's successor to GFS; HDFS is open-source and modeled directly on the GFS paper.

</details>

### Sp26-E3 Q5 — 2x replication I/O
You write 5 MB to a 2x replicated file in HDFS, then later read it back. How much data will be read and written to disks across the cluster?
- (A) 5 MB written, 10 MB read
- (B) 10 MB written, 10 MB read
- (C) 10 MB written, 5 MB read
- (D) 5 MB written, 5 MB read

<details>
<summary><b>Show answer</b></summary>

**(C) 10 MB written, 5 MB read.** 2x replication → each byte written twice (10 MB total disk writes). Reads only need one replica → 5 MB read.

</details>

### Sp26-E3 Q6 — Network bytes (2x replication, 11 MB / 8 MB blocks)
A client writes a 11 MB file to HDFS with 2x replication. The block size is 8 MB. How much data does the client send over the network?
- (A) 8 MB  (B) 11 MB  (C) 16 MB  (D) 22 MB  (E) 32 MB

<details>
<summary><b>Show answer</b></summary>

**(B) 11 MB.** Pipelined writes: client sends each byte exactly once over the network. Replication factor and block size don't change client-side network bytes — they're handled DataNode-to-DataNode in the pipeline.

</details>

---

## Spark

### S25 Q3 — Hash partitioning columns
For the below Spark SQL query, over which column(s) will hash values be calculated for hash partitioning?
```
SELECT F, MAX(R) FROM mytable GROUP BY H, F;
```
- (A) F  (B) R  (C) F and H  (D) H  (E) R and F

<details>
<summary><b>Show answer</b></summary>

**(C) F and H.** Hash partitioning for aggregation uses the GROUP BY columns so that all rows with the same group key land on the same machine. The SELECT list doesn't affect partitioning.

</details>

### S25 Q14 — Join algorithm with hash partitioning
Which join algorithm uses hash partitioning to bring rows from each table that could potentially match with each other together on the same machine?
- (A) SMJ  (B) BHJ

<details>
<summary><b>Show answer</b></summary>

**(A) SMJ (Shuffle Sort Merge Join).** SMJ shuffles both tables by hash of the join key so matching rows co-locate, then sorts and merges. BHJ broadcasts the small table — no partitioning needed.

</details>

### S25 Q17 — Streaming partition mismatch
Say you want to run a streaming Spark query over a Kafka topic. The topic is partitioned by column X, but the query is grouping by a different column, Y. What will happen?
- (A) Spark will refuse to run the query
- (B) Spark will produce incorrect outputs
- (C) Spark will be able to group correctly by column Y

<details>
<summary><b>Show answer</b></summary>

**(C) Spark will group correctly.** Spark performs its own shuffle to repartition by Y. Kafka's partitioning is just an input distribution — Spark doesn't depend on it for correctness, only performance.

</details>

### S25 Q19 — Caching levels using JVM types
Which Spark caching levels will use the JVM types to represent data?
- (A) MEMORY_ONLY and MEMORY_ONLY_2
- (B) MEMORY_ONLY_SER and MEMORY_ONLY_SER_2
- (C) MEMORY_ONLY AND MEMORY_ONLY_SER
- (D) MEMORY_ONLY_2 AND MEMORY_ONLY_SER_2

<details>
<summary><b>Show answer</b></summary>

**(A) MEMORY_ONLY and MEMORY_ONLY_2.** The non-`_SER` levels store deserialized JVM objects (fast access, more memory). The `_SER` levels serialize the data (compact, slower). The `_2` suffix means 2x replicated for fault tolerance — orthogonal to serialization.

</details>

### S25 Q20 — Watermark discard time
A Spark streaming query is maintaining a count for an interval starting at 3pm. At what time could Spark reasonably discard the running count for events occuring in this interval?
```python
(animals.withWatermark("timestamp", "2 hours")
        .groupBy(window("timestamp", "4 hours"))
        .count())
```
- (A) 11pm  (B) 10pm  (C) 9pm  (D) 7pm  (E) 5pm

<details>
<summary><b>Show answer</b></summary>

**(C) 9pm.** Window: 3pm → 7pm (4-hour window). Watermark allows 2 more hours of late data after window close. So Spark holds state until 7pm + 2hr = 9pm.

</details>

### F25 Q3 — `.cache` default
In Spark, `.cache` is a convenience method that calls `.persist(...)` with what setting?
- (A) MEMORY_ONLY  (B) MEMORY_ONLY_SER  (C) DISK_ONLY  (D) DISK_ONLY_2

<details>
<summary><b>Show answer</b></summary>

**(A) MEMORY_ONLY.** `cache()` is shorthand for `persist(StorageLevel.MEMORY_ONLY)`.

</details>

### F25 Q14 — Choosing join
You have a Spark cluster with 50 machines, each with 64 GB of memory. You need to join two tables. Smaller table: 9.6 GB. Bigger table: 31655.5 GB. As long as you don't run out of memory, your goal should be to minimize network I/O.
- (A) SMJ (Shuffle Sort Merge Join)
- (B) BHJ (Broadcast Hash Join)

<details>
<summary><b>Show answer</b></summary>

**(B) BHJ.** 9.6 GB easily fits in 64 GB per machine, so broadcast it. Otherwise SMJ would shuffle the 31 TB table — catastrophic. BHJ broadcasts the small table (9.6 GB × 50 machines = 480 GB total network) vs SMJ shuffling 31655 GB.

</details>

### F24 Q3 — Task fundamentals
A single Spark task typically runs on _________ and operates on __________.
- (A) one core, one partition
- (B) one core, many partitions
- (C) multiple cores, one partition
- (D) multiple cores, many partitions

<details>
<summary><b>Show answer</b></summary>

**(A) one core, one partition.** This is the basic unit of Spark parallelism. Number of tasks ≈ number of partitions; concurrency is limited by total cores in the cluster.

</details>

### F24 Q4 — When BHJ helps
When is BHJ most beneficial?
- (A) both tables are small
- (B) one table is small and one is large
- (C) both tables are large

<details>
<summary><b>Show answer</b></summary>

**(B) one small, one large.** Broadcast the small table; the large table stays put. Two small tables don't need a special algorithm. Two large tables can't fit broadcasts.

</details>

### F24 Q17 — Streaming stateless
In Spark streaming, is the following stateless?
```sql
SELECT MAX(x) AS total FROM mystream;
```
- (A) yes  (B) no

<details>
<summary><b>Show answer</b></summary>

**(B) no.** MAX is an aggregation requiring running state across all events. A truly stateless query operates row-by-row (e.g. `SELECT 1/x FROM stream`).

</details>

### F24 Q19 — Immutability
Which of the following are immutable in Spark?
- (A) only Pipeline  (B) only PipelineModel  (C) both Pipeline and PipelineModel

<details>
<summary><b>Show answer</b></summary>

**(C) both.** Pipeline (estimator) is immutable; calling `.fit(data)` returns a new PipelineModel. PipelineModel is also immutable; transforms produce new DataFrames.

</details>

### F24 Q20 — Streaming partition mismatch (repeat pattern)
Say you want to run a streaming Spark query over a Kafka topic. The topic is partitioned by column X, but the query is grouping by a different column, Y. What will happen?
- (A) Spark will refuse to run the query
- (B) Spark will produce incorrect outputs
- (C) Spark will be able to group correctly by column Y

<details>
<summary><b>Show answer</b></summary>

**(C) Spark will group correctly.** Same reasoning as S25 Q17 — Spark reshuffles internally.

</details>

### Sp24 Q2 — Unsupported streaming op
Which of the following operations is not supported by Spark streaming?
- (A) group by  (B) group by time intervals  (C) inner join  (D) watermarks  (E) pivots

<details>
<summary><b>Show answer</b></summary>

**(E) pivots.** Pivots require knowing all distinct values up front; not feasible on an unbounded stream.

</details>

### Sp24 Q3 — Hive table method
Which of the following methods enables us to create a HIVE table?
- (A) createTempView  (B) createOrReplaceTempView  (C) createGlobalTempView  (D) saveAsTable

<details>
<summary><b>Show answer</b></summary>

**(D) saveAsTable.** This persists to the Hive metastore as a managed table. The other three create in-memory views that disappear when the session ends.

</details>

### Sp24 Q7 — Small fits-in-memory join
Suppose we want to use Spark to join two tables using a small number of worker machines. If one of those tables fits entirely into memory, which of the following join algorithms should we pick?
- (A) Broadcast Hash Join  (B) Shuffle Sort Merge Join

<details>
<summary><b>Show answer</b></summary>

**(A) Broadcast Hash Join.** When the small table fits in memory, broadcasting eliminates shuffling entirely.

</details>

### Sp24 Q9 — Fundamental data structure
Which of the following is the fundamental data structure of Spark?
- (A) DataFrame  (B) table  (C) view  (D) protocol buffer  (E) RDD

<details>
<summary><b>Show answer</b></summary>

**(E) RDD.** DataFrames and Datasets are higher-level abstractions built on top of RDDs.

</details>

### Sp24 Q11 — Unfit decision tree type
Which Spark type is used for a decision tree model that has NOT been fit to the data yet?
- (A) DecisionTreeRegressor  (B) DecisionTreeRegressionModel

<details>
<summary><b>Show answer</b></summary>

**(A) DecisionTreeRegressor.** The "Regressor" is the estimator (unfit). Calling `.fit()` returns a `DecisionTreeRegressionModel`. General pattern: `XxxRegressor` / `XxxClassifier` are unfit; `XxxModel` is fit.

</details>

### Sp24 Q16 — Filter lambda
Suppose `banks_df` is a Spark DataFrame containing a column called "name". Which of the following lambda expressions enables us to filter all rows that contain "First" as part of the bank names?
- (A) `lambda banks_df: "First" == banks_df["name"]`
- (B) `lambda b: "First" in b["name"]`
- (C) `lambda banks_df: "First" in banks_df["name"]`
- (D) `lambda b: "First" == b["name"]`

<details>
<summary><b>Show answer</b></summary>

**(B) `lambda b: "First" in b["name"]`.** The lambda takes one row at a time (conventionally `b`, not `banks_df`), and `in` checks for substring containment. `==` would require exact match.

</details>

### Sp24 Q17 — ML predictions
How do you make predictions using Spark ML implementations?
- (A) invoke transform method on unfit model
- (B) invoke predict method on unfit model
- (C) invoke fit method on fitted model
- (D) invoke predict method on fitted model
- (E) invoke transform method on fitted model

<details>
<summary><b>Show answer</b></summary>

**(E) transform on fitted model.** Spark ML uses `transform()` for prediction (unlike scikit-learn's `predict()`). And you must use the fitted model.

</details>

### Sp24 Q29 — Similar system
Which of the following systems is most similar to Spark?
- (A) Colossus  (B) BigTable  (C) BigQuery  (D) HBase  (E) Dynamo

<details>
<summary><b>Show answer</b></summary>

**(C) BigQuery.** Both are distributed analytical query/processing engines. The others (Colossus, BigTable, HBase, Dynamo) are storage systems.

</details>

### Sp24 Q30 — Not a transformation
Which of the following is NOT a Spark transformation operation?
- (A) filter  (B) parallelize  (C) mean  (D) map

<details>
<summary><b>Show answer</b></summary>

**(C) mean.** mean is an action — returns a value to the driver and triggers execution. filter and map return RDDs (lazy transformations). parallelize creates an RDD from a Python collection.

</details>

### Sp24 Q32 — Streaming stateless
Consider Spark streaming. Is the following query stateless?
```sql
SELECT 1/x AS inverse FROM some_stream;
```
- (A) yes  (B) no

<details>
<summary><b>Show answer</b></summary>

**(A) yes.** Each row is computed independently. No aggregation, no state to maintain.

</details>

### Sp26-E3 Q4 — Caching speed with lots of RAM
If you have lots of RAM, which caching level will generally be fastest?
- (A) MEMORY_ONLY  (B) MEMORY_ONLY_SER  (C) DISK_ONLY

<details>
<summary><b>Show answer</b></summary>

**(A) MEMORY_ONLY.** Stores deserialized JVM objects — no serialization overhead on access. `MEMORY_ONLY_SER` saves space at the cost of CPU. `DISK_ONLY` is much slower.

</details>

### Sp26-E3 Q8 — Choosing join (BHJ vs SMJ)
You have a Spark cluster with 20 machines, each with 8 GB of memory. You need to join two tables. Smaller table: 1.5 GB. Bigger table: 2560.8 GB. Goal: minimize network I/O.
- (A) SMJ (Shuffle Sort Merge Join)
- (B) BHJ (Broadcast Hash Join)

<details>
<summary><b>Show answer</b></summary>

**(B) BHJ.** 1.5 GB fits comfortably in 8 GB per machine. BHJ network cost ≈ 1.5 × 20 = 30 GB; SMJ would shuffle 2560+ GB.

</details>

### Sp26-E3 Q9 — RDD acronym (fill-in)
RDD stands for ____________

<details>
<summary><b>Show answer</b></summary>

**Resilient Distributed Dataset.** Resilient = lineage-based fault recovery. Distributed = partitioned across cluster. Dataset = collection of records.

</details>

### Sp26-E3 Q12 — Lazy operation returning an RDD (fill-in)
A lazy Spark operation that returns an RDD is called a(n) ____________

<details>
<summary><b>Show answer</b></summary>

**transformation.** Transformations (map, filter, etc.) build the lineage graph but don't execute. Actions (collect, count, mean) trigger execution and return values to the driver.

</details>

### Sp26-E3 Q13 — One-to-one partition mapping (fill-in)
A Spark transformation where each input partition maps to exactly one output partition is called a ____________ transformation

<details>
<summary><b>Show answer</b></summary>

**narrow.** Narrow transformations (map, filter, union) don't shuffle. Wide transformations (groupBy, join, repartition) do.

</details>

### Sp26-E3 Q15 — Stop caching a DataFrame (fill-in)
To stop caching a Spark DataFrame df, call df.____________()

<details>
<summary><b>Show answer</b></summary>

**unpersist.** `df.unpersist()` removes the DataFrame from memory and disk caches.

</details>

### Sp26-E3 Q17 — RDD lineage (drawing)
Given:
```python
A = ... # RDD of all ints from 0 to 1 million, in 10 partitions
B = A.sample(True, 0.01, 544)
C = B.map(lambda x: x * 2).repartition(5)
D = B.mean()
E = C.filter(lambda x: x < 10)
```
Draw a directed graph (A through E). Boxes for RDDs (label partition counts beneath), circles for materialized values. Label edges as transformation (T) or action (A).

<details>
<summary><b>Show answer</b></summary>

**Diagram:**
- **A** (box, 10 partitions) — sample is a narrow transformation, preserves partitions
- **A → B** labeled **T** (sample)
- **B** (box, 10 partitions) — map is narrow but `.repartition(5)` shuffles to 5
- **B → C** labeled **T** (map + repartition)
- **C** (box, 5 partitions) — filter is narrow, partition count preserved
- **B → D** labeled **A** (mean is an action)
- **D** (circle, materialized float — the mean value)
- **C → E** labeled **T** (filter)
- **E** (box, 5 partitions)

Key gotchas: `sample` keeps original partition count (10). `repartition(5)` is what changes C to 5 partitions. `mean` is the only action — returns a number, drawn as a circle.

</details>

---

## Cassandra

### S25 Q2 — Design inspiration
What are two systems that inspired the design of Cassandra?
- (A) BigTable and Dynamo
- (B) BigTable and MapReduce
- (C) BigQuery and Dynamo
- (D) BigQuery and MapReduce

<details>
<summary><b>Show answer</b></summary>

**(A) BigTable and Dynamo.** Cassandra borrowed the data model (column families, sparse rows) from Google's BigTable paper and the peer-to-peer / consistent-hashing / quorum architecture from Amazon's Dynamo paper.

</details>

### S25 Q21 — Token map (2x replication)
Assuming 2x replication, which node(s) are responsible a new row being inserted?
Row: `x="red", y="green", z="blue"`. The primary key is `("x", "y")`.
Hashes: `hash("red")=6, hash("green")=-4, hash("blue")=-8, hash(<"red","green">)=2, hash(<"red","green","blue">)=-6`.
Token map: `token(n1) = [3, 4, -2], token(n2) = [-5, -3, -1], token(n3) = [-8]`
Reference: `-8 | -7 | -6 | -5 | -4 | -3 | -2 | -1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

<details>
<summary><b>Show answer</b></summary>

**(E) n2+n3.** Primary key `(x, y)` written this way means **x is partition key, y is cluster key**. Hash of partition key = hash("red") = 6. Sort all tokens: -8(n3), -5(n2), -3(n2), -2(n1), -1(n2), 3(n1), 4(n1). Walk clockwise from 6: nothing higher than 6, so wrap to -8 (n3). For RF=2, advance to next *distinct node*: -5 belongs to n2. Replicas: n3 + n2.

</details>

### S25 Q22 — Quorums (R, W given)
For Cassandra, R=5 and W=2. Readers are guaranteed to see previous writes. What can we infer about RF? Choose the answer that provides the tightest bound on RF.
- (A) RF >= 7  (B) RF >= 8  (C) RF < 7  (D) RF > 7  (E) RF < 8

<details>
<summary><b>Show answer</b></summary>

**(C) RF < 7.** Strong consistency requires R + W > RF. So 5 + 2 > RF → RF < 7. (E) RF < 8 is also true but looser. (C) is the tightest.

</details>

### F25 Q2 — Primary key uniqueness
A Cassandra table has three columns: X (first column, partition key), Y (second column, cluster key), Z (third column, regular). You insert these rows:
```
(2,2,4)
(3,2,2)
(3,2,4)
(2,1,5)
(2,1,2)
```
How many rows will be in the table?
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

<details>
<summary><b>Show answer</b></summary>

**(D) 3.** Uniqueness in Cassandra is determined by the primary key (X, Y). Distinct (X,Y) combos: (2,2), (3,2), (2,1). Repeat inserts overwrite Z. Final rows: (2,2,4), (3,2,4), (2,1,2).

</details>

### F25 Q13 — Quorums (R, RF given)
Cassandra Quorums: Given R=5 and RF=10, what should W be to make sure readers see successful writes? If multiple satisfy this, choose the smallest correct.
- (A) 1  (B) 4  (C) 6  (D) 8

<details>
<summary><b>Show answer</b></summary>

**(C) 6.** R + W > RF → 5 + W > 10 → W > 5 → smallest W = 6.

</details>

### F25 Q21 — CAP priority
What characteristic does Cassandra's design prioritize?
- (A) availability  (B) atomicity  (C) consistency  (D) isolation

<details>
<summary><b>Show answer</b></summary>

**(A) availability.** Cassandra is "AP" in CAP — Availability + Partition tolerance, sacrificing strong consistency. It will keep accepting writes even during partitions, with eventual consistency via tunable quorums and read repair.

</details>

### F25 Q22 — Token map (2x replication)
Assuming 2x replication, which node(s) are responsible for a new row being inserted?
Row: `x="red", y="green", z="blue"`. Primary key: `("x", "y")`.
Hashes: `hash("red")=2, hash("green")=-2, hash("blue")=-5, hash(<"red","green">)=4, hash(<"red","green","blue">)=-8`.
Token map: `token(n1) = [4, -4], token(n2) = [-1, -5], token(n3) = [6]`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

<details>
<summary><b>Show answer</b></summary>

**(D) n1+n3.** Partition key is x → hash("red") = 2. Sort tokens: -5(n2), -4(n1), -1(n2), 4(n1), 6(n3). Clockwise from 2: next is 4 (n1). For RF=2, next *distinct* node: 6 (n3). Replicas: n1 + n3.

</details>

### F24 Q2 — Token map (2x replication, given token)
Assuming 2x replication, which node(s) are responsible for row token 3?
Token map: `token(n1) = [-7], token(n2) = [7], token(n3) = [4]`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

<details>
<summary><b>Show answer</b></summary>

**(E) n2+n3.** Sort tokens: -7(n1), 4(n3), 7(n2). Clockwise from 3: next is 4 (n3). Next distinct node: 7 (n2). Replicas: n3 + n2.

</details>

### F24 Q21 — Gossip protocol
Which of the following uses a gossip protocol for updating information about cluster membership?
- (A) HDFS  (B) Spark  (C) Cassandra  (D) Kafka

<details>
<summary><b>Show answer</b></summary>

**(C) Cassandra.** Peer-to-peer architecture, no central master, so nodes gossip state to each other (alive/dead, schema versions, token assignments).

</details>

### F24 Q22 — Quorums (W, RF given)
Cassandra Quorums: Given W=4 and RF=9, what should R be to make sure readers see successful writes? Smallest correct.
- (A) 2  (B) 4  (C) 5  (D) 6

<details>
<summary><b>Show answer</b></summary>

**(D) 6.** R + W > RF → R + 4 > 9 → R > 5 → smallest R = 6.

</details>

### Sp24 Q10 — Read deduplication
Which of the following techniques is used to avoid reading identical copies of the same data when Cassandra read quorum R > 0?
- (A) pipelined reads  (B) caching  (C) checksum  (D) compression

<details>
<summary><b>Show answer</b></summary>

**(C) checksum.** Cassandra fetches the full data from one replica and a digest (checksum) from the others. If checksums match, no need to transfer duplicate full data — saves network. Mismatch triggers read repair.

</details>

### Sp24 Q20 — Token map (RF=3)
Keyspace replication_factor=3. Token map:
```
token(n1) = {5, 8}
token(n2) = {11, 15}
token(n3) = {-2, 12}
token(n4) = {-5, 20}
```
Partition key hashes to -1. Which vnodes are responsible?
- (A) n1
- (B) n1 and n3
- (C) n1 and n2
- (D) n1, n2, and n3
- (E) n1, n2, n3, and n4

<details>
<summary><b>Show answer</b></summary>

**(D) n1, n2, and n3.** Sort tokens: -5(n4), -2(n3), 5(n1), 8(n1), 11(n2), 12(n3), 15(n2), 20(n4). Clockwise from -1: 5(n1) → 8(n1, same node skip) → 11(n2) → 12(n3). Three distinct nodes: n1, n2, n3.

</details>

### Sp24 Q21 — New node joining
Same token map as above. Node n5 joins with vnodes 9 and -4. Which existing node(s) will pass off some data to this new node?
- (A) n1 and n2
- (B) n2 and n3
- (C) n2 and n4
- (D) n1 and n4
- (E) n4 and n5

<details>
<summary><b>Show answer</b></summary>

**(B) n2 and n3.** A new vnode steals the range from the previous token up to itself.
- vnode 9: previous token is 8 (n1). Range (8, 11] previously belonged to n2 (next existing token). n5 takes (8, 9] → from n2.
- vnode -4: previous token is -5 (n4). Range (-5, -2] previously belonged to n3. n5 takes (-5, -4] → from n3.

So n2 and n3 give up data.

</details>

### Sp24 Q28 — Unique values under primary key
```sql
CREATE TABLE sample(X INT, Y INT, Z TEXT, PRIMARY KEY ((X), Y));
INSERT INTO sample (X, Y, Z) VALUES (1, 1, 'a');
INSERT INTO sample (X, Y, Z) VALUES (2, 1, 'b');
INSERT INTO sample (X, Y, Z) VALUES (1, 1, 'c');
INSERT INTO sample (X, Y, Z) VALUES (2, 1, 'd');
INSERT INTO sample (X, Y, Z) VALUES (3, 1, 'e');
```
How many unique values will be in the Z column of the sample table?
- (A) 1  (B) 2  (C) 3  (D) 4  (E) 5

<details>
<summary><b>Show answer</b></summary>

**(C) 3.** Primary key is (X, Y). Distinct (X,Y): (1,1) → overwritten with 'c'; (2,1) → overwritten with 'd'; (3,1) → 'e'. Final Z values: c, d, e.

</details>

### Sp24 Q31 — Wrapping range
Same token map as Q20. What is the wrapping range?
- (A) > 15 <= 20  (B) > 20  (C) >= 20  (D) <= -5  (E) < -5

<details>
<summary><b>Show answer</b></summary>

**(B) > 20.** The wrapping range is the segment that spans the "top" of the ring. Largest token is 20; anything > 20 wraps clockwise to the smallest token's owner. (No data hashes to exactly 20 — tokens themselves don't own their exact value, the range is what matters.)

</details>

### Sp24 Q33 — Quorums (RF, R given)
Considering Cassandra quorums, suppose RF=10 and R=5, what should W be to make sure that we read the latest successful write?
- (A) 1  (B) 2  (C) 4  (D) 5  (E) 6

<details>
<summary><b>Show answer</b></summary>

**(E) 6.** R + W > RF → 5 + W > 10 → W > 5 → smallest W = 6.

</details>

---

## Kafka

### S25 Q10 — Leader election eligibility
A Kafka partition leader fails, and there are three followers. Which are eligible to become the new leader?
- Follower 1: in-sync, and has all messages that the old leader had
- Follower 2: in-sync, but is missing 10 messages that the old leader had
- Follower 3: lagging, but is missing 1 message that the old leader had
- (A) only 1  (B) 1 or 2  (C) 1 or 3  (D) 1, 2, or 3

<details>
<summary><b>Show answer</b></summary>

**(B) 1 or 2.** Only in-sync replicas (ISR members) are eligible for leader election by default. Follower 3 is lagging (out of ISR) → ineligible despite being closer to the leader's data. Electing F2 would lose 10 messages but is still permitted; that's an availability vs durability trade-off Kafka allows. (Unclean leader election would let F3 in, but the default is ISR-only.)

</details>

### S25 Q13 — Replication write path
A Kafka topic has a replication factor of 3. How will new data be written to the replicas?
- (A) The client will write the message directly to the leader and both followers.
- (B) The client will write the message to the leader, and the followers will later fetch it.
- (C) The client will write the message to the leader, which will actively send it to both followers.
- (D) The client will send the data to the leader, the leader will send it to the first follower, and the first follower will send it to the second follower.
- (E) The client will send the data to the first follower, the first follower will send it to the second follower, and the second follower will send it to the leader, at which point it will be committed.

<details>
<summary><b>Show answer</b></summary>

**(B) Followers fetch from the leader.** Kafka uses a *pull* replication model — followers issue fetch requests to the leader, just like consumers. The leader doesn't push to followers (unlike HDFS which uses pipelined push).

</details>

### S25 Q16 — Consumer groups
There are 4 Kafka groups, each with 5 consumer(s). All groups are subscribed to topic T. A new message in T will be consumed how many times?
- (A) 20  (B) 7  (C) 5  (D) 4  (E) 1

<details>
<summary><b>Show answer</b></summary>

**(D) 4.** Each consumer *group* sees every message exactly once (delivered to one consumer in the group). 4 groups → 4 deliveries. Number of consumers per group affects parallelism, not delivery count.

</details>

### S25 Q29 — Same partition guarantees
What can we guarantee about which messages will go to the same partition?
1. `topic="purple", key="green", value="red"`
2. `topic="red", key="green", value="red"`
3. `topic="green", key="red", value="red"`
- (A) 1 and 2 will go to the same partition
- (B) 1 and 3 will go to the same partition
- (C) 2 and 3 will go to the same partition
- (D) We can't guarantee anything

<details>
<summary><b>Show answer</b></summary>

**(D) We can't guarantee anything.** Same-partition guarantees only apply *within a single topic* with the same key. All three messages have different topics, so they go to different topic+partition spaces entirely.

</details>

### F25 Q10 — Consumer groups (repeat)
There are 4 Kafka groups, each with 5 consumer(s). All groups are subscribed to topic T. A new message in T will be consumed how many times?
- (A) 1  (B) 3  (C) 4  (D) 5  (E) 20

<details>
<summary><b>Show answer</b></summary>

**(C) 4.** One delivery per group; 4 groups.

</details>

### F25 Q16 — Same partition guarantees
1. `topic="green", key="purple", value="red"`
2. `topic="red", key="blue", value="purple"`
3. `topic="green", key="green", value="red"`
- (A) 1 and 2  (B) 1 and 3  (C) 2 and 3  (D) Can't guarantee

<details>
<summary><b>Show answer</b></summary>

**(D) Can't guarantee.** 1 and 3 share a topic but have different keys → different partitions (likely). 2 has a different topic. No same-key+same-topic pair exists.

</details>

### F25 Q20 — Committed?
Given the following Kafka partition state, is message E committed?
- Leader: A, B, C, D, E
- Follower 1 (lagging): A, B
- Follower 2 (in-sync): A, B, C, D, E
- Follower 3 (in-sync): A, B
- (A) Yes  (B) No

<details>
<summary><b>Show answer</b></summary>

**(B) No.** A message is committed only when *all in-sync replicas* (ISR) have it. F3 is labeled in-sync but only has A, B — so E hasn't propagated to all ISR members yet. Not committed.

</details>

### F25 Q29 — No custom code
For which one do you NOT usually need to write custom code when using Kafka?
- (A) producers  (B) brokers  (C) consumers

<details>
<summary><b>Show answer</b></summary>

**(B) brokers.** Brokers are Kafka's built-in server processes — you configure them, you don't code them. Producers and consumers are application code.

</details>

### F24 Q13 — Same partition guarantees
1. `topic="W", key="Z", value="X"`
2. `topic="W", key="X", value="X"`
3. `topic="X", key="Z", value="Z"`
- (A) 1 and 2  (B) 1 and 3  (C) 2 and 3  (D) Can't guarantee

<details>
<summary><b>Show answer</b></summary>

**(D) Can't guarantee.** 1 and 2 share topic W but have different keys. 3 is a different topic. No same-topic+same-key pair.

</details>

### F24 Q16 — High volume bottleneck
You have a high-volume Kafka topic. The brokers can keep up, but the consumers cannot. What is most likely to help?
- (A) use more topic partitions
- (B) use fewer topic partitions
- (C) start more consumer groups
- (D) start more consumers per consumer group

<details>
<summary><b>Show answer</b></summary>

**(D) more consumers per consumer group.** Within a group, partitions are assigned to consumers; more consumers = more parallel processing. (A) helps only if you also add consumers; (C) makes things worse — more groups means each message gets processed *more* times. The first lever is adding consumers, then scaling partitions if you've maxed out.

</details>

### Sp24 Q5 — Delivery semantics
Producer produced 10, 20, 30 to topic "nums". A Spark streaming query computing the product produces 600. What semantics?
- (A) at-most-once  (B) at-least-once  (C) exactly-once

<details>
<summary><b>Show answer</b></summary>

**(C) exactly-once.** 10 × 20 × 30 = 6000... wait, that's 6000, not 600. Let me recheck — actually 10 × 20 × 30 = 6000. But the question says output is 600. Hmm. If the product is 600, that means one of the messages was missed (e.g. 10 × 30 × 2 doesn't work, but 20 × 30 = 600 → only 20 and 30 processed). Actually 20 × 30 = 600 — so the 10 was dropped → at-most-once.

Reconsidering: if product = 600 and inputs are {10, 20, 30}, then we have 20 × 30 (skipped 10), suggesting **at-most-once**. Going with **(A) at-most-once**.

</details>

### Sp24 Q13 — Order guarantees
What can we guarantee about consumption order?
1. `topic="sports", key="A", value="hello"`
2. `topic="sports", key="B", value="sports fans"`
3. `topic="weather", key="B", value="sunny"`
4. `topic="weather", key="B", value="and bright"`
5. `topic="international", key="A", value="overseas"`
6. `topic="international", value=10`
- (A) msg 1 before msg 5
- (B) msg 2 before msg 3
- (C) msg 3 before msg 4
- (D) msg 4 before msg 3
- (E) msg 5 before msg 6

<details>
<summary><b>Show answer</b></summary>

**(C) msg 3 before msg 4.** Order is guaranteed only within a single partition. Same topic + same key → same partition. Msgs 3 and 4 share `topic="weather", key="B"` → same partition → ordered by produce order (3 then 4). Cross-topic or different-key messages have no order guarantee.

</details>

### Sp24 Q14 — Uncommitted fetches
In Kafka, both consumers and followers send fetch requests to the leader. Who can fetch uncommitted messages in Kafka?
- (A) only consumers
- (B) only followers
- (C) both consumers and followers
- (D) neither consumers nor followers

<details>
<summary><b>Show answer</b></summary>

**(B) only followers.** Followers must see uncommitted messages — that's how they catch up and *make* messages committed. Consumers only see committed data (this is the normal isolation guarantee).

</details>

### Sp24 Q18 — Min commits with min.insync.replicas
RF=5, min.insync.replicas=3. There are 4 in-sync replicas and 1 lagging. Minimum number of replicas a message must be written to in order to be considered committed?
- (A) 1  (B) 2  (C) 3  (D) 4  (E) 5

<details>
<summary><b>Show answer</b></summary>

**(D) 4.** Commit requires writing to *all in-sync replicas* (not just min.insync.replicas). The min.insync.replicas setting (3) is the *minimum required to accept writes at all*; if fewer ISR than that, writes are rejected. With 4 ISR currently, all 4 must acknowledge for commit.

</details>

---

## BigQuery

### S25 Q1 — ML clause
Which clause related to machine-learning does BigQuery add to SQL?
- (A) TEST  (B) TRAIN  (C) TRANSFORM  (D) TRANSPOSE

<details>
<summary><b>Show answer</b></summary>

**(C) TRANSFORM.** BigQuery ML adds a TRANSFORM clause for declaring feature engineering steps that are applied identically at training and prediction time.

</details>

### S25 Q5 — Billing using leftovers
Which BigQuery billing model uses "leftover" CPU and memory resources?
- (A) capacity  (B) on-demand  (C) rollover  (D) spare

<details>
<summary><b>Show answer</b></summary>

**(D) spare.** Spare capacity / flex slots use otherwise-idle compute. Capacity is reserved slots you pay for regardless. On-demand bills per byte scanned.

</details>

### S25 Q26 — Internal engine
The query engine for BigQuery is internally based on what system?
- (A) GFS  (B) Dremel  (C) Spark  (D) MapReduce

<details>
<summary><b>Show answer</b></summary>

**(B) Dremel.** Google's Dremel paper (2010) is the basis for BigQuery's serving tree execution model. GFS/Colossus is storage. MapReduce is batch (different paradigm).

</details>

### S25 Q27 — Correlated cross join unnest
If you do a correlated cross join between columns y and z (after unnesting each), how many rows will you get?
```
x, y, z
1, [2, 3], [4]
5, [6, 7], [8, 9, 10]
```
- (A) 2  (B) 4  (C) 7  (D) 8  (E) 16

<details>
<summary><b>Show answer</b></summary>

**(D) 8.** Correlated = per-row cartesian product within each row. Row 1: |y|×|z| = 2×1 = 2. Row 2: 2×3 = 6. Total: 2 + 6 = 8.

</details>

### S25 Q28 — Parquet inspiration
Which format inspired Parquet?
- (A) Arrow  (B) Capacitor  (C) ColumnIO  (D) Protocol Buffers

<details>
<summary><b>Show answer</b></summary>

**(C) ColumnIO.** ColumnIO is the column format used inside Dremel (described in the Dremel paper). Parquet was directly inspired by it. Arrow is in-memory, Capacitor is BigQuery's internal evolution.

</details>

### F25 Q1 — Storage I/O rounding
When BigQuery computes query cost based on bytes of storage I/O, how does it round I/O?
- (A) rounds up  (B) rounds down

<details>
<summary><b>Show answer</b></summary>

**(A) rounds up.** Always rounds up to a minimum unit (typically 10 MB per query, then per-MB rounding up). You're billed for at least the unit.

</details>

### F25 Q5 — Non-cloud analog
Which non-cloud platform is most similar to Google's BigQuery?
- (A) Spark  (B) Cassandra  (C) Kafka  (D) HBase  (E) BigTable

<details>
<summary><b>Show answer</b></summary>

**(A) Spark.** Both are distributed analytical query engines. Spark SQL gives the closest open-source analog. The others (Cassandra, Kafka, HBase, BigTable) are storage / messaging systems, not analytical engines.

</details>

### F25 Q19 — PLANET split points
What split points does PLANET consider?
- (A) between any unique values
- (B) thresholds between bins in an equi-width histogram
- (C) thresholds between bins in an equi-depth histogram

<details>
<summary><b>Show answer</b></summary>

**(B) equi-width histogram bins.** PLANET (Parallel Learner for Assembling Numerous Ensemble Trees) approximates split-point search by using equi-width bin thresholds, avoiding the cost of considering every unique value at scale.

</details>

### F25 Q26 — SQLX origin
SQLX is an extension of SQL provided by which of the following?
- (A) Arrow  (B) BigQuery  (C) Cassandra  (D) Dataform  (E) GCS

<details>
<summary><b>Show answer</b></summary>

**(D) Dataform.** Dataform (a GCP tool, formerly third-party) provides SQLX for templated, dependency-aware SQL workflows on top of BigQuery.

</details>

### F25 Q27 — Correlated cross join unnest
```
x , y , z
5 , [6, 7], [8, 9]
10, [11] , [12, 13]
```
- (A) 2  (B) 6  (C) 7  (D) 8  (E) 9

<details>
<summary><b>Show answer</b></summary>

**(B) 6.** Row 1: 2×2 = 4. Row 2: 1×2 = 2. Total: 6.

</details>

### F24 Q1 — Capacity billing free
What is something that capacity billing gives BigQuery users for free?
- (A) CPU  (B) memory  (C) Colossus I/O  (D) Colossus Storage

<details>
<summary><b>Show answer</b></summary>

**(C) Colossus I/O.** Capacity (slot-based) billing covers compute (CPU + memory bundled in slots). Storage is always billed separately. Bytes scanned (Colossus I/O) is not separately metered under capacity — that's the on-demand model.

</details>

### F24 Q5 — Capacitor format access
If you want to run BigQuery over data in the Capacitor format, how should you add tables to your dataset?
- (A) load job  (B) external table

<details>
<summary><b>Show answer</b></summary>

**(A) load job.** Capacitor is BigQuery's *internal* storage format. Data ends up in Capacitor only after being loaded into BigQuery storage (via a load job). External tables reference data in other formats (Parquet, CSV) at their original location.

</details>

### F24 Q26 — Centroid output volume
You run `SELECT FUNC(geom) FROM geotable` in BigQuery. Which FUNC will generally result in more output rows?
- (A) ST_CENTROID  (B) ST_CENTROID_AGG  (C) tie

<details>
<summary><b>Show answer</b></summary>

**(A) ST_CENTROID.** ST_CENTROID is a per-row scalar function — one output row per input row. ST_CENTROID_AGG is an aggregate — one output row per group (or one total without GROUP BY).

</details>

### F24 Q27 — Correlated cross join unnest
```
x, y, z
1, [2, 3], [4, 5]
6, [7], [8, 9, 10]
```
- (A) 0  (B) 2  (C) 4  (D) 7  (E) 15

<details>
<summary><b>Show answer</b></summary>

**(D) 7.** Row 1: 2×2 = 4. Row 2: 1×3 = 3. Total: 7.

</details>

### Sp24 Q23 — ML coefficients
Which of the following will enable you to determine the coefficients used to multiply features?
- (A) ML.PREDICT  (B) ML.OPTIONS  (C) ML.EVALUATE  (D) ML.WEIGHTS

<details>
<summary><b>Show answer</b></summary>

**(D) ML.WEIGHTS.** Returns the trained model's feature coefficients/weights. PREDICT runs inference; EVALUATE gives metrics; OPTIONS isn't a function.

</details>

### Sp24 Q24 — Not column-oriented
Which of the following is not a column-oriented format?
- (A) Capacitor  (B) CSV  (C) ColumnIO  (D) Parquet

<details>
<summary><b>Show answer</b></summary>

**(B) CSV.** CSV stores rows as full lines (row-oriented). The other three are column-oriented analytical formats.

</details>

### Sp24 Q25 — UNNEST row count
Given the language column contains REPEATED RECORDS:
```sql
SELECT repo_name, ARRAY_LENGTH(language) as total_languages
FROM `bigquery-public-data.github_repos.languages`
WHERE ARRAY_LENGTH(language) > 200
```
returns:
```
repo_name           total_languages
polyrabbit/polyglot   216
```
How many rows in df after:
```sql
SELECT * FROM `bigquery-public-data.github_repos.languages`, UNNEST(language)
WHERE repo_name = "polyrabbit/polyglot"
```
- (A) 0  (B) 1  (C) 2  (D) 200  (E) 216

<details>
<summary><b>Show answer</b></summary>

**(E) 216.** UNNEST flattens the array — one row per array element, repeating the parent row's columns. Array has 216 elements → 216 rows.

</details>

### Sp24 Q27 — Lat/long to geo
In BigQuery, which function converts floating-point longitude/latitude into geographic data?
- (A) ST_LATLONG  (B) ST_GEOGPOINT  (C) ST_CENTROID  (D) ST_MAKEPOINT

<details>
<summary><b>Show answer</b></summary>

**(B) ST_GEOGPOINT.** Signature: `ST_GEOGPOINT(longitude, latitude)`. Note longitude comes first (a frequent gotcha).

</details>

### Sp26-E2 Q2 — Layout orientation (storage)
Is the below data layout "column oriented" or "row oriented"?
Table:
```
6, 4, 3
5, 1, 2
```
Disk layout: `6, 4, 3, 5, 1, 2`
- (A) column oriented  (B) row oriented

<details>
<summary><b>Show answer</b></summary>

**(B) row oriented.** Reading left to right: row 1 (6, 4, 3) then row 2 (5, 1, 2). Column-oriented would be: 6, 5, 4, 1, 3, 2 (all of column 1, then column 2, then column 3).

</details>

### Sp26-E2 Q6 — OLTP layout
An OLTP database typically uses what orientation for its on-disk layout?
- (A) row-oriented  (B) column-oriented

<details>
<summary><b>Show answer</b></summary>

**(A) row-oriented.** OLTP workloads read/write whole rows (e.g. "fetch this customer"). Row-oriented co-locates a row's columns. Column-oriented is for OLAP — fast scans across one column for analytics.

</details>

### Sp26-E2 Q17 — Arrow representation (drawing)
Consider the Python list of strings: `["AB", "C", "", "DEF"]`. Illustrate how Arrow would represent this in a cache-friendly way:
1. Write letters where they appear in the value buffer (single byte per letter).
2. Write values in the offset buffer to support efficient indexing over the strings.
3. Fill in 1s or 0s in the validity bitmap (rightmost bits = smallest indexes).

<details>
<summary><b>Show answer</b></summary>

**Value buffer:** `A B C D E F` (offsets 0 1 2 3 4 5)

**Offset buffer:** `[0, 2, 3, 3, 6]` — n+1 offsets for n strings.
- String 0 "AB" → bytes [0, 2)
- String 1 "C"  → bytes [2, 3)
- String 2 ""   → bytes [3, 3)  ← empty: same start and end
- String 3 "DEF" → bytes [3, 6)

**Validity bitmap:** `0000 1111` (rightmost 4 bits are valid). Empty string is *valid* in Arrow (length 0); only `null` flips a bit to 0. Since none of the strings are null, all 4 bits are 1. Bits beyond index 3 are unused.

</details>

---

## Other Cloud

### S25 Q8 — Free tiers
How do "free tiers" usually work for cloud services?
- (A) you are not charged for initial operations up to some limit
- (B) you are not charged for additional operations after exceeding some limit

<details>
<summary><b>Show answer</b></summary>

**(A) initial operations up to some limit.** Free tier = a usage allowance (e.g. first 1 GB egress free, first X queries free). Once you exceed the allowance, you start paying.

</details>

### S25 Q12 — EC2 service kind
What kind of service is EC2?
- (A) IaaS  (B) PaaS

<details>
<summary><b>Show answer</b></summary>

**(A) IaaS (Infrastructure as a Service).** EC2 gives you raw VMs. PaaS would manage the runtime for you (App Engine, Heroku).

</details>

### S25 Q15 — GFLOPS calc
You have 6 billion floating point operations to do on a device capable of 3 GFLOPS. How many seconds will it take?
- (A) 0.0005  (B) 0.5  (C) 1  (D) 2.0  (E) 500.0

<details>
<summary><b>Show answer</b></summary>

**(D) 2.0.** 6×10⁹ ops ÷ 3×10⁹ ops/sec = 2 sec.

</details>

### F25 Q8 — Spot vs on-demand
You need to run a once-a-day batch job that can wait if the VM is temporarily unavailable, and it is not critical if it gets interrupted. Which type of VM instance is most suitable?
- (A) on-demand instances  (B) spot instances

<details>
<summary><b>Show answer</b></summary>

**(B) spot instances.** Spot instances are deeply discounted but can be reclaimed by the cloud at any time. Perfect for non-urgent, fault-tolerant batch jobs.

</details>

### F25 Q12 — Powered-off costs
When you power off a cloud VM, what do you usually still pay for while it is off?
- (A) memory only  (B) CPU and memory  (C) CPU only  (D) disk capacity

<details>
<summary><b>Show answer</b></summary>

**(D) disk capacity.** Persistent disks are billed for storage as long as they exist. CPU and memory are only billed when the VM is running.

</details>

### F25 Q23 — SSD challenge
Which I/O pattern is most challenging for SSDs?
- (A) random reads  (B) random writes  (C) sequential reads  (D) sequential writes

<details>
<summary><b>Show answer</b></summary>

**(B) random writes.** SSDs erase in large blocks but write in pages. Random writes scatter changes → write amplification + garbage collection. Reads don't cause erase cycles.

</details>

### F25 Q28 — Network I/O cost
What usually costs more for cloud network I/O?
- (A) ingress  (B) egress

<details>
<summary><b>Show answer</b></summary>

**(B) egress.** Cloud providers want to attract data in (free ingress) but charge to take data out. Major source of revenue and a soft form of vendor lock-in.

</details>

### F24 Q8 — Kubernetes vs Compose
What is something that Kubernetes does that Compose does not do?
- (A) bin packing
- (B) use cgroups to isolate performance
- (C) deploy multiple replicas from the same Docker image

<details>
<summary><b>Show answer</b></summary>

**(A) bin packing.** Kubernetes schedules pods across a multi-node cluster, fitting them onto nodes by resource requests. Compose is single-host. Both use cgroups (via Docker) and can run multiple containers from an image.

</details>

### F24 Q12 — VM cost split
What generally costs more when deploying on a cloud? Option 1: one VM for 100 hours. Option 2: 100 VMs for 1 hour.
- (A) option 1 costs more  (B) option 2 costs more  (C) the costs are similar

<details>
<summary><b>Show answer</b></summary>

**(C) costs are similar.** Cloud bills per VM-hour (or vCPU-hour). 1 × 100 = 100 × 1 = 100 VM-hours. Total compute = total cost (modulo small per-instance overhead).

</details>

### F24 Q28 — Cloud organization
What best describes cloud organization?
- (A) zones contain regions  (B) regions contain zones  (C) clusters contain regions

<details>
<summary><b>Show answer</b></summary>

**(B) regions contain zones.** A region is a geographic area (e.g. us-east-1). Each region contains several isolated zones (separate buildings/power/network) for fault tolerance.

</details>

### Sp26-E1 Q2 — GFLOPS calc (Version A)
You have 4 billion floating point operations to do on a device capable of 8 GFLOPS. How many seconds will it take?
- (A) 0.002  (B) 0.5  (C) 1  (D) 2.0  (E) 2000.0

<details>
<summary><b>Show answer</b></summary>

**(B) 0.5.** 4×10⁹ ÷ 8×10⁹ = 0.5 sec.

**Variant answers:**
- V-B (4 million / 2 GFLOPS): 4×10⁶ ÷ 2×10⁹ = 0.002 → **(A)**
- V-C (8 million / 4 GFLOPS): 8×10⁶ ÷ 4×10⁹ = 0.002 → **(A)**
- V-D (8 million / 16 GFLOPS): 8×10⁶ ÷ 16×10⁹ = 0.0005 → **(A)**

Watch the units: "billion" = 10⁹, "million" = 10⁶, GFLOPS = 10⁹ ops/sec.

</details>

> Variations across versions: B uses 4 million ops / 2 GFLOPS, C uses 8 million / 4 GFLOPS, D uses 8 million / 16 GFLOPS.

### Sp26-E2 Q8 — SSD garbage collection
What access pattern creates the most garbage collection work for SSDs?
- (A) random reads  (B) sequential reads  (C) random writes  (D) sequential writes

<details>
<summary><b>Show answer</b></summary>

**(C) random writes.** Random writes maximally fragment SSD blocks, requiring more GC cycles to consolidate live pages and erase blocks for reuse.

</details>

---

## Minor systems (MySQL / MapReduce / HBase)

### F24 Q14 — HBase RegionServers
How does HBase assign data to RegionServers? Assume we are using 3x replication.
- (A) each column will belong to one RegionServer
- (B) each column will belong to three RegionServers
- (C) each region will belong to one RegionServer
- (D) each region will belong to three RegionServers

<details>
<summary><b>Show answer</b></summary>

**(C) each region will belong to one RegionServer.** A region is owned by exactly one RegionServer at a time for serving traffic. The 3x replication is at the *HDFS* layer underneath (each HFile block is replicated 3 times on disk), which is independent of region ownership.

</details>

### Sp26-E3 Q7 — MapReduce: how many map() calls
For a MapReduce job, you have 1000 input key/value pairs, 138 intermediate key/value pairs, and 43 output key/value pairs. Among the 1000 inputs, there are 4 unique keys. How many times will map(...) be called?
- (A) 4  (B) 43  (C) 138  (D) 1000

<details>
<summary><b>Show answer</b></summary>

**(D) 1000.** map() is called once per *input pair*. The 4 unique keys matter for *reduce* (reduce is called once per unique intermediate key, after the shuffle groups them — but here intermediate uniques aren't given). The numbers 138 and 43 are red herrings for this part.

</details>

> Note: Direct HBase / MySQL / MapReduce questions are sparse in the past Exam 3 papers since those topics were typically tested earlier in the semester. Expect new final-exam questions covering: HBase region assignment and master node, full MapReduce phases (map / shuffle / reduce) and intermediate I/O behavior, and MySQL replication / consistency tradeoffs vs Cassandra.

---

## Review (Caching, Concurrency, Linux, Docker, Networking, SQL, Idempotency, Memory)

### Caching

#### S25 Q23 — FIFO size 4
How many hits are there for a FIFO cache of size 4 for the workload `1, 2, 3, 1, 4, 2, 1, 7`?
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

<details>
<summary><b>Show answer</b></summary>

**(D) 3.** Walk through (FIFO doesn't reorder on hit):
| step | access | cache | result |
|---|---|---|---|
| 1 | 1 | [1] | miss |
| 2 | 2 | [1,2] | miss |
| 3 | 3 | [1,2,3] | miss |
| 4 | 1 | [1,2,3] | **HIT** |
| 5 | 4 | [1,2,3,4] | miss |
| 6 | 2 | [1,2,3,4] | **HIT** |
| 7 | 1 | [1,2,3,4] | **HIT** |
| 8 | 7 | [2,3,4,7] | miss (evict 1, oldest) |

3 hits.

</details>

#### F25 Q9 — LRU size 3
How many cache hits for `D, D, A, D, B, A, A, C` with LRU eviction and cache size 3?
- (A) 4  (B) 5  (C) 6  (D) 7  (E) 8

<details>
<summary><b>Show answer</b></summary>

**(A) 4.** LRU reorders on hit (most recently used moves to "front"):
| step | access | cache (LRU → MRU) | result |
|---|---|---|---|
| 1 | D | [D] | miss |
| 2 | D | [D] | **HIT** |
| 3 | A | [D, A] | miss |
| 4 | D | [A, D] | **HIT** |
| 5 | B | [A, D, B] | miss |
| 6 | A | [D, B, A] | **HIT** |
| 7 | A | [D, B, A] | **HIT** |
| 8 | C | [B, A, C] | miss (evict D, the LRU) |

4 hits.

</details>

#### F24 Q15 — FIFO size 3
How many hits for FIFO cache size 3 with workload `6, 4, 2, 3, 4, 3, 6, 6`?
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

<details>
<summary><b>Show answer</b></summary>

**(D) 3.**
| step | access | cache | result |
|---|---|---|---|
| 1 | 6 | [6] | miss |
| 2 | 4 | [6,4] | miss |
| 3 | 2 | [6,4,2] | miss |
| 4 | 3 | [4,2,3] | miss (evict 6) |
| 5 | 4 | [4,2,3] | **HIT** |
| 6 | 3 | [4,2,3] | **HIT** |
| 7 | 6 | [2,3,6] | miss (evict 4) |
| 8 | 6 | [2,3,6] | **HIT** |

3 hits.

</details>

#### Sp24 Q1 — LRU hit rate
LRU cache size 4. Hit rate for `P, Q, P, R, S, Q, T, P` (cache empty initially).
- (A) 0  (B) 0.25  (C) 0.375  (D) 0.75  (E) 1

<details>
<summary><b>Show answer</b></summary>

**(B) 0.25.**
| step | access | cache (LRU → MRU) | result |
|---|---|---|---|
| 1 | P | [P] | miss |
| 2 | Q | [P,Q] | miss |
| 3 | P | [Q,P] | **HIT** |
| 4 | R | [Q,P,R] | miss |
| 5 | S | [Q,P,R,S] | miss |
| 6 | Q | [P,R,S,Q] | **HIT** |
| 7 | T | [R,S,Q,T] | miss (evict P, LRU) |
| 8 | P | [S,Q,T,P] | miss |

2 hits / 8 accesses = **0.25**.

</details>

#### Sp24 Q34 — FIFO hit rate
FIFO cache size 4. Hit rate for `P, Q, P, R, S, Q, T, R` (cache empty initially).
- (A) 0  (B) 0.25  (C) 0.375  (D) 0.75  (E) 1

<details>
<summary><b>Show answer</b></summary>

**(C) 0.375.**
| step | access | cache | result |
|---|---|---|---|
| 1 | P | [P] | miss |
| 2 | Q | [P,Q] | miss |
| 3 | P | [P,Q] | **HIT** |
| 4 | R | [P,Q,R] | miss |
| 5 | S | [P,Q,R,S] | miss |
| 6 | Q | [P,Q,R,S] | **HIT** |
| 7 | T | [Q,R,S,T] | miss (evict P, oldest) |
| 8 | R | [Q,R,S,T] | **HIT** |

3 hits / 8 = **0.375**.

</details>

#### Sp26-E2 Q4 — FIFO size 3 (count)
How many hits will there be? FIFO cache, capacity=3. Workload: `1, 2, 3, 1, 4, 1`
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

<details>
<summary><b>Show answer</b></summary>

**(B) 1.**
| step | access | cache | result |
|---|---|---|---|
| 1 | 1 | [1] | miss |
| 2 | 2 | [1,2] | miss |
| 3 | 3 | [1,2,3] | miss |
| 4 | 1 | [1,2,3] | **HIT** |
| 5 | 4 | [2,3,4] | miss (evict 1, oldest) |
| 6 | 1 | [3,4,1] | miss (1 was just evicted!) |

1 hit. The trick: under FIFO, accessing 1 in step 4 doesn't refresh its position — it gets evicted next.

</details>

### Concurrency / Threading

#### S25 Q6 — Single thread + global
What value(s) could possibly be printed?
```python
x = 5
def task():
    global x
    x += 2
t = threading.Thread(target=task)
t.start()
t.join()
print(x)
```
- (A) 5 or 7  (B) 5 or 2  (C) only 5  (D) only 7  (E) only 2

<details>
<summary><b>Show answer</b></summary>

**(D) only 7.** Single thread, plus `t.join()` *before* the print, means the thread always finishes its `x += 2` before main reads x. Deterministic: 5 + 2 = 7.

</details>

#### F25 Q7 — Which functions need the lock
The library uses a lock for `f` and `g` only. Which other functions must acquire the lock to avoid race conditions?
```python
lock = threading.Lock()
x = 2; y = 1; z = 2
def f():
    global y, x
    with lock:
        y += z
        x += 1
def g():
    global z
    with lock:
        z *= x
def A(): print(x)
def B(): print(y)
def C(): print(z)
```
- (A) A and C  (B) C only  (C) A and B  (D) A only  (E) A, B, and C

<details>
<summary><b>Show answer</b></summary>

**(E) A, B, and C.** Each reader touches a variable that f or g modifies under the lock:
- A reads x → modified by f (and read by g while held)
- B reads y → modified by f
- C reads z → modified by g (and read by f while held)

Without the lock, a reader could observe a torn / inconsistent state mid-update.

</details>

#### F25 Q30 — Lock-protected sequence
```python
x = 4
lock = threading.Lock()
def task():
    global x
    with lock:
        x = x - 3
t = threading.Thread(target=task)
t.start()
with lock:
    x = x * 3
t.join()
print(x)
```
- (A) only 9  (B) only 1  (C) only 3  (D) 3 or 9  (E) only 12

<details>
<summary><b>Show answer</b></summary>

**(D) 3 or 9.** Both blocks acquire the lock, so the order is non-deterministic depending on which one wins:
- Main first: 4 × 3 = 12, then thread: 12 - 3 = **9**
- Thread first: 4 - 3 = 1, then main: 1 × 3 = **3**

So 3 or 9.

</details>

#### F24 Q6 — Two threads with lock
X starts at 5, two threads run concurrently with a global lock. What are the possible final values?
```python
# thread 1
with lock: X *= 2
# thread 2
with lock: X += 1
```
- (A) only 11  (B) only 12  (C) 11 or 12  (D) 6, 10, 11, or 12  (E) 5, 6, 10, 11, or 12

<details>
<summary><b>Show answer</b></summary>

**(C) 11 or 12.**
- Thread 1 first: 5 × 2 = 10, then 10 + 1 = **11**
- Thread 2 first: 5 + 1 = 6, then 6 × 2 = **12**

The lock prevents intermediate values from being observed.

</details>

#### Sp24 Q19 — Critical section term
A portion of code we don't want to be interrupted by another thread is called a ?
- (A) context switch  (B) critical section  (C) lock  (D) collision

<details>
<summary><b>Show answer</b></summary>

**(B) critical section.** The *region of code* is the critical section. The *mechanism* protecting it is the lock.

</details>

#### Sp26-E2 Q1 — Lock + main-thread lock
What value(s) could possibly be printed?
```python
x = 3
lock = threading.Lock()
def task():
    global x
    with lock:
        x = x * 3
t = threading.Thread(target=task)
t.start()
with lock:
    x = x + 5
t.join()
print(x)
```
- (A) only 24  (B) only 14  (C) only 8  (D) 14 or 24  (E) only 9

<details>
<summary><b>Show answer</b></summary>

**(D) 14 or 24.**
- Thread first: 3 × 3 = 9, then 9 + 5 = **14**
- Main first: 3 + 5 = 8, then 8 × 3 = **24**

</details>

#### Sp26-E3 Q2 — Lock and context switching (T/F)
True/False: when a thread is holding a lock during a critical section, the scheduler WILL NEVER context switch to another thread in the same process.
- (A) True  (B) False

<details>
<summary><b>Show answer</b></summary>

**(B) False.** The OS scheduler doesn't know about user-level locks. A thread holding a lock can absolutely be preempted. The lock just prevents *other threads* from entering the critical section while it's held — not from being scheduled in general. (This is exactly why locks must be held only briefly.)

</details>

#### Sp26-E3 Q16 — Per-thread storage (fill-in)
Each thread has its own ____________ for storing local variables.

<details>
<summary><b>Show answer</b></summary>

**stack.** Each thread has a private stack for local variables and call frames. The heap and code segments are shared across threads in a process.

</details>

### Idempotency

#### F25 Q17 — Indexed assignment
Is `do_it` idempotent?
```python
data = [3, 8, 2]
def do_it(val, d):
    data[d] = val
```
- (A) Yes  (B) No

<details>
<summary><b>Show answer</b></summary>

**(A) Yes.** Calling `do_it(5, 1)` once or ten times leaves `data[1] = 5`. Repeated calls with the same args produce the same final state.

</details>

#### F24 Q10 — Squaring
Is the following function idempotent?
```python
def set_square():
    global x
    x = x ** 2
```
- (A) Yes  (B) No

<details>
<summary><b>Show answer</b></summary>

**(B) No.** First call: x → x². Second call: x² → x⁴. Different state each call (unless x ∈ {0, 1}, but that's not the general case). Not idempotent.

</details>

#### Sp24 Q4 — Inverse
Idempotent?
```python
x = 10
def inverse_x():
    global x
    x = 1 / x
```
- (A) yes  (B) no

<details>
<summary><b>Show answer</b></summary>

**(B) no.** First call: 10 → 0.1. Second call: 0.1 → 10. State flips. Idempotency requires that *repeated* invocation has the same effect as one invocation; this oscillates.

</details>

### Linux / Shell

#### S25 Q9 — Documentation command
What Linux command provides documentation about how to use a given program?
- (A) wget  (B) which  (C) cat  (D) du  (E) man

<details>
<summary><b>Show answer</b></summary>

**(E) man.** `man <program>` opens the manual page. `which` finds the binary path. `cat` prints files. `du` shows disk usage. `wget` downloads.

</details>

#### F25 Q15 — Redirect stdout only
How do you redirect ONLY the standard output from program X to file Y?
- (A) X > Y  (B) X -> Y  (C) X | Y  (D) X & Y  (E) X &> Y

<details>
<summary><b>Show answer</b></summary>

**(A) X > Y.** `>` redirects stdout (overwriting). `&>` redirects both stdout *and* stderr. `|` pipes to another command. `&` runs in background.

</details>

#### F24 Q23 — Port to process
What Linux tool can help you see what process is using a port?
- (A) ls  (B) ns  (C) os  (D) ps  (E) ss

<details>
<summary><b>Show answer</b></summary>

**(E) ss.** `ss -tlnp` shows listening TCP sockets with the owning process. (Older alternative: `netstat`.) `ps` shows processes but not their ports. `ls` is for files.

</details>

#### Sp24 Q35 — Pipe to grep
Search for "UN" inside output.txt. What replaces ???
```
cat output.txt ??? grep "UN"
```
- (A) &  (B) >  (C) &>  (D) >>  (E) |

<details>
<summary><b>Show answer</b></summary>

**(E) |.** Pipe sends cat's stdout to grep's stdin. (Of course, you could also just run `grep "UN" output.txt` directly — this is the "useless use of cat" pattern.)

</details>

#### Sp26-E1 (V-A,B) Q11 / (V-C,D) Q12 — List files (fill-in)
The ____________ command lists the files and directories in a location.

<details>
<summary><b>Show answer</b></summary>

**ls.** Common flags: `-l` long format, `-a` show hidden, `-h` human-readable sizes.

</details>

#### Sp26-E1 (V-A,B) Q14 — Run as root (fill-in)
To run a command with root privileges, prefix it with ____________.

<details>
<summary><b>Show answer</b></summary>

**sudo.** "Substitute user do" — temporarily elevates privileges for a single command.

</details>

#### Sp26-E1 (V-A,B) Q16 / (V-C,D) Q13 — Word count (fill-in)
The ____________ program counts lines, words, and characters in its stdin.

<details>
<summary><b>Show answer</b></summary>

**wc.** `wc -l` lines, `-w` words, `-c` bytes, `-m` characters.

</details>

#### Sp26-E1 (V-C,D) Q12 — Permissions (fill-in)
File access permissions in Linux are changed using the ____________ command.

<details>
<summary><b>Show answer</b></summary>

**chmod.** Change mode. Symbolic (`chmod u+x file`) or octal (`chmod 755 file`).

</details>

#### Sp26-E1 (V-C,D) Q13 / Q9 — Download file (fill-in)
The w____________ command downloads a file from a URL, saving it locally.

<details>
<summary><b>Show answer</b></summary>

**wget.** Downloads via HTTP/HTTPS/FTP. (`curl` is the alternative — slightly different defaults.)

</details>

#### Sp26-E1 (V-C,D) Q16 — Current directory (fill-in)
Running ____________ displays the full path of your current working directory.

<details>
<summary><b>Show answer</b></summary>

**pwd.** "Print working directory."

</details>

#### Sp26-E2 Q14 — Unbuffered Python (fill-in)
The -u flag in `python3 -u server.py` forces stdout and stderr to be ____________.

<details>
<summary><b>Show answer</b></summary>

**unbuffered.** Without `-u`, Python buffers stdout when not connected to a terminal (e.g. when piped or redirected to a Docker log). Buffering can hide output for a long time. `-u` forces immediate flushing — essential for container logs.

</details>

#### Sp26-E3 Q14 — Pattern filter (fill-in)
The ____________ program filters lines of input, keeping only those that match a pattern.

<details>
<summary><b>Show answer</b></summary>

**grep.** "Global regular expression print." `grep "pattern" file` or `cmd | grep "pattern"`.

</details>

#### Sp26-E1 Q17 / Sp26-E1 (V-C,D) Q17 — Shell pipeline diagrams (drawing)
Sp26-E1 (V-A,B): Draw the result of running:
```
A | B
C > E
D >> E
```

Sp26-E1 (V-C,D): Draw the result of:
```
A > B
C > B
D | E
```

In each, A is already drawn. Represent processes as boxes (with stdin/stdout labeled), files as boxes labeled with their final contents, and use arrows to connect.

<details>
<summary><b>Show answer</b></summary>

**Version A,B (`A | B`, `C > E`, `D >> E`):**

Processes: A (already drawn), B, C, D — each prints its name to stdout.

- `A | B`: A's stdout → B's stdin (pipe). B's stdout goes to terminal. So A doesn't reach the terminal directly; B does. B reads "A" from stdin (and ignores it), then prints "B" to stdout.
- `C > E`: C's stdout overwrites file E. After this, file E contains: `C`
- `D >> E`: D's stdout appends to file E. After this, file E contains: `C\nD` (i.e., "C" then "D" on the next line).

**Final layout:**
- Processes box: A (stdin from terminal, stdout → B's stdin), B (stdin from A, stdout → terminal), C (stdin terminal, stdout → file E), D (stdin terminal, stdout → file E)
- File `E` contains: `C` then `D`
- Terminal output: `B`

**Version C,D (`A > B`, `C > B`, `D | E`):**

- `A > B`: A's stdout → file B (overwrites). File B contains: `A`
- `C > B`: C's stdout → file B (overwrites again). File B now contains: `C`. The `A` from earlier is gone.
- `D | E`: D's stdout → E's stdin. E reads "D" (ignores), prints "E" to terminal.

**Final layout:**
- File `B` contains: `C`
- Terminal output: `E`

The trick on V-C,D is that two `>` redirections overwrite each other — only the last write to the file survives.

</details>

### Docker / Containers

#### S25 Q18 — Mount flag
In Docker, if you want a file/directory location on the host machine to be visible within a container, what flag should you pass to run?
- (A) -d  (B) -f  (C) -p  (D) -v

<details>
<summary><b>Show answer</b></summary>

**(D) -v.** `-v /host/path:/container/path` mounts a volume. `-d` is detached mode. `-p` is port forwarding. `-f` doesn't apply to `run` typically.

</details>

#### S25 Q30 — SSH tunnel + Docker port (Spring 2025 chain)
Steps:
1. Dockerfile launches Jupyter on port 2856
2. SSH tunnel: `-L localhost:4702:localhost:3973`
3. `docker run ... -p ????:2856`
4. Browser: `http://localhost:4702/`

What should ???? be in step 3?
- (A) 8888  (B) 3973  (C) 5000  (D) 2856  (E) 4702

<details>
<summary><b>Show answer</b></summary>

**(B) 3973.** Trace the chain backwards: browser hits laptop:4702 → SSH tunnel forwards to VM:3973 → that's the *host* port on the VM, which `docker -p` maps to container:2856. So `???? = 3973`.

The recipe: `????` always equals the right-hand inner port of the SSH `-L` flag.

</details>

#### F25 Q6 — `docker ps`
What does `docker ps` show?
- (A) what images Docker has locally
- (B) what containers are running
- (C) what processes are running within a container

<details>
<summary><b>Show answer</b></summary>

**(B) what containers are running.** `ps` here mirrors the Linux `ps` for processes, but lists containers. `docker images` lists images. `docker exec ... ps` would list processes inside a container.

</details>

#### F25 Q18 — Memory limit
A container launched with `-m 2g` reads `big.file` (3 GB) on a VM with 4 GB free RAM. No swap. Will memory constraints prevent successful execution?
- (A) yes  (B) no

<details>
<summary><b>Show answer</b></summary>

**(A) yes.** `-m 2g` is the *container's* memory cap (enforced via cgroups), regardless of how much free RAM the VM has. Loading 3 GB into a process inside a 2 GB cap → OOM kill. Note: simple sequential reads might not require holding the whole file in memory, but if the program does load it (e.g., `df = pd.read_csv(...)`), it dies.

</details>

#### F24 Q25 — SSH tunnel + Docker port (Fall 2024 chain)
Steps:
1. Dockerfile launches Jupyter on port 2241
2. `docker run ... -p 3752:2241`
3. SSH tunnel: `-L localhost:4432:localhost:3752`
4. Browser: `http://localhost:????/`

What should ???? be in step 4?
- (A) 2241  (B) 5000  (C) 8888  (D) 4432  (E) 3752

<details>
<summary><b>Show answer</b></summary>

**(D) 4432.** The browser hits the *laptop* port, which is the left-hand value of the SSH `-L` flag. The browser doesn't know about the VM or container — it just talks to localhost on the laptop, and SSH does the forwarding.

</details>

#### F24 Q30 — Detached mode output
A Docker container `myapp` is running in detached mode. How can you see what the process started by CMD is printing?
- (A) `docker ps myapp`
- (B) `docker logs myapp`
- (C) `docker exec -it myapp`
- (D) `docker exec myapp stdout`

<details>
<summary><b>Show answer</b></summary>

**(B) `docker logs myapp`.** Detached mode (`-d`) doesn't show stdout/stderr in the terminal. `docker logs` retrieves the captured output. (`docker logs -f` follows new output.)

</details>

#### Sp24 Q12 — File classification
What is the following an example of?
```
FROM ubuntu:23.10
RUN apt-get update && apt-get install -y unzip python3 python3-pip
RUN pip3 install pandas===2.1.0 --break-system-packages
```
- (A) yml file  (B) protocol buffer  (C) Dockerfile  (D) nodetool

<details>
<summary><b>Show answer</b></summary>

**(C) Dockerfile.** Dead giveaways: `FROM <image>` and `RUN <command>` are the two most common Dockerfile directives.

</details>

#### Sp26-E1 Q1 — Containers vs VMs (cross-OS) [V-A, V-B]
If you need to sandbox processes, some of which run on Linux and others on Windows, which is better?
- (A) containers  (B) virtual machines

<details>
<summary><b>Show answer</b></summary>

**(B) virtual machines.** Containers share the *host kernel*. You can't run Windows programs (which need the Windows kernel) in a container on a Linux host (or vice versa). VMs each have their own kernel, so they can run different operating systems on the same physical host.

</details>

#### Sp26-E1 Q1 — Containers vs VMs (memory) [V-C, V-D]
If you need to sandbox processes and your main concern is minimizing memory usage, which is better?
- (A) containers  (B) virtual machines

<details>
<summary><b>Show answer</b></summary>

**(A) containers.** Containers share the host kernel and don't carry per-instance OS overhead. VMs each have a full OS in memory, costing hundreds of MB to GB per VM.

</details>

#### Sp26-E1 Q4 / Sp26-E3 Q1 — Docker port chain (Spring 2026)
Step 1: Dockerfile launches Jupyter on port 2446
Step 2: SSH tunnel `-L localhost:4369:localhost:3035`
Step 3: `docker run ... -p ????:2446`
Step 4: Browser `http://localhost:4369/`

What should ???? be?
- (A) 5000  (B) 4369  (C) 2446  (D) 3035  (E) 8888

<details>
<summary><b>Show answer</b></summary>

**(D) 3035.** Same recipe: `???? = right-hand inner port of the SSH -L flag`. The chain: browser → laptop:4369 → (SSH) → VM:3035 → (docker -p) → container:2446.

**Variant answers** (same logic, different numbers):
- V-B (SSH `-L 4290:localhost:3254`, container 2505): `???? = 3254` → **(B)**
- V-C (SSH `-L 4747:localhost:3572`, container 2003): `???? = 3572` → **(C)**
- V-D (SSH `-L 4211:localhost:3488`, container 2550): `???? = 3488` → **(C)**

</details>

> Variations: V-B uses 2505/4290/3254, V-C uses 2003/4747/3572, V-D uses 2550/4211/3488. The principle is the same: ???? is the host port that should match the SSH tunnel's right-hand inner port.

#### Sp26-E1 Q7 — Build-time directive [V-A, V-B]
In a Dockerfile, how could you specify that "apt update" should execute during build?
- (A) EXEC apt update  (B) DO apt update  (C) CMD apt update  (D) RUN apt update

<details>
<summary><b>Show answer</b></summary>

**(D) RUN apt update.** RUN executes during the *build* (creating a new image layer). CMD specifies the *default command run when the container starts*. EXEC and DO aren't valid Dockerfile directives.

</details>

#### Sp26-E1 Q7 — Default startup program [V-C, V-D]
In a Dockerfile, how do you specify the program that should launch (by default) when a container starts?
- (A) EXEC  (B) RUN  (C) CMD  (D) DO

<details>
<summary><b>Show answer</b></summary>

**(C) CMD.** CMD is the default command at *container start*. Can be overridden by `docker run <image> <cmd>`. `ENTRYPOINT` is similar but harder to override.

</details>

#### Sp26-E1 Q9 — Delete image (fill-in) [V-A, V-B] / Q10 [V-A, V-B]
To delete a Docker image X, run `docker ____________ X`.

<details>
<summary><b>Show answer</b></summary>

**rmi** (`docker rmi X`). "Remove image." For containers, it's `docker rm`.

</details>

#### Sp26-E1 Q9 — Running containers (fill-in) [V-C, V-D] / Q14 [V-D]
To see currently running Docker containers, run `docker ____________`.

<details>
<summary><b>Show answer</b></summary>

**ps.** Add `-a` to also see stopped containers (`docker ps -a`).

</details>

#### Sp26-E2 Q3 — Rebuild speed
If you build an image from a Dockerfile, change the Dockerfile, then build again, what change will usually result in a SLOWER rebuild?
- (A) changing a line near the beginning
- (B) changing a line near the end

<details>
<summary><b>Show answer</b></summary>

**(A) changing a line near the beginning.** Docker caches each layer top-down. When a line changes, that layer's cache is invalidated and *every layer after it* must rebuild. Changing the last line invalidates only one layer; changing the first invalidates all of them. Best practice: put rarely-changing things (apt installs, dependencies) early, frequently-changing things (your code) late.

</details>

#### Sp26-E2 Q11 — Run a command inside a container (fill-in)
To run a command inside a running container, use `docker ____________`.

<details>
<summary><b>Show answer</b></summary>

**exec.** `docker exec -it <container> bash` is the canonical "shell into a container" incantation. `-it` makes it interactive with a TTY.

</details>

### Networking / Memory / Misc

#### S25 Q7 — Truly unique ID
You don't want any chance of different computers using the same library to produce the same ID. What information about the machine would be most helpful?
- (A) IP address  (B) MAC address  (C) port number of current process  (D) free disk space

<details>
<summary><b>Show answer</b></summary>

**(B) MAC address.** MAC addresses are assigned at hardware-manufacturing time and (in principle) globally unique. IPs change between networks. Port numbers are reused. Disk space is not unique.

</details>

#### S25 Q25 — RAM granularity
For RAM, what is the finest granularity at which every piece of memory has its own address?
- (A) bit  (B) byte  (C) cacheline  (D) page  (E) block

<details>
<summary><b>Show answer</b></summary>

**(B) byte.** RAM is byte-addressable. Cachelines and pages are units of *transfer* / *management*, not addressing.

</details>

#### F25 Q25 — URL component
You have a URL `someprotocol://someaddr:someport/someresource`. Which part determines the specific running process on a machine that will receive the request?
- (A) someprotocol  (B) someaddr  (C) someport  (D) someresource

<details>
<summary><b>Show answer</b></summary>

**(C) someport.** Address picks the machine. Port picks the listening process on that machine. Resource is then handled by that process at the application (HTTP) layer.

</details>

#### F24 Q7 — gRPC serialization
What does gRPC use to serialize messages?
- (A) ColumnIO  (B) JSON  (C) Parquet  (D) Protocol Buffers

<details>
<summary><b>Show answer</b></summary>

**(D) Protocol Buffers.** Compact binary serialization with schema-based forward/backward compatibility. JSON is too slow/large; ColumnIO and Parquet are columnar storage formats, not RPC serialization.

</details>

#### Sp24 Q15 — Encoding identification
A column has values: `apple, apple, apple, banana, banana, orange, orange`. Represented as `{3: 1, 2: 2, 2: 3}` and `{"apple": 1, "banana": 2, "orange": 3}`. What technique(s)?
- (A) only dictionary encoding
- (B) only run-length encoding
- (C) both dictionary encoding and run-length encoding

<details>
<summary><b>Show answer</b></summary>

**(C) both.** The first map `{3: 1, 2: 2, 2: 3}` is run-length encoding: "3 of code 1, 2 of code 2, 2 of code 3." The second `{"apple": 1, "banana": 2, "orange": 3}` is the dictionary that decodes those numeric codes back to strings. Together: dictionary + RLE.

</details>

#### Sp24 Q22 — PyTorch tensor bytes
A 20x5 PyTorch tensor stores double precision floats. How many bytes (excluding Python object overhead)?
- (A) 64  (B) 200  (C) 400  (D) 800  (E) 51200

<details>
<summary><b>Show answer</b></summary>

**(D) 800.** 20 × 5 = 100 elements. Double precision = 8 bytes. 100 × 8 = 800 bytes.

</details>

#### Sp26-E1 Q3 — URL HTTP layer
You have a URL `http://someaddr:someport/someresource`. Which part is processed by the HTTP layer of the network stack, specifically?
- (A) someaddr  (B) someport  (C) someresource

<details>
<summary><b>Show answer</b></summary>

**(C) someresource.** Address is handled by IP routing (network layer). Port is handled by TCP (transport layer). The path/resource is part of the HTTP request line — handled by HTTP (application layer).

</details>

#### Sp26-E1 Q8 — Latency measurement [V-A, V-B]
Which one is an example of a latency measurement?
- (A) 6 MB  (B) 2 seconds  (C) 3 MB/s

<details>
<summary><b>Show answer</b></summary>

**(B) 2 seconds.** Latency = time. Throughput = data / time (e.g. MB/s). Just data (MB) is a size, not a rate or duration.

</details>

#### Sp26-E1 Q8 — Throughput measurement [V-C, V-D]
Which one is an example of a throughput measurement?
- (A) 6 MB  (B) 2 seconds  (C) 3 MB/s

<details>
<summary><b>Show answer</b></summary>

**(C) 3 MB/s.** Throughput is data per unit time.

</details>

#### Sp26-E1 Q12 — IPv4 size (fill-in)
An IPv4 address contains ____________ bits.

<details>
<summary><b>Show answer</b></summary>

**32.** Four octets × 8 bits each = 32 bits. (IPv6 is 128 bits.)

</details>

#### Sp26-E1 Q13 — Address translation (fill-in)
A N____________ is often used to forward from public to private IP addresses.

<details>
<summary><b>Show answer</b></summary>

**NAT (Network Address Translation).** Lets many private IPs share one public IP. Maps connections back to the right internal host using port numbers.

</details>

#### Sp26-E1 Q15 — Transport protocol (fill-in)
Name either of the transport protocols we covered in class that provides port numbers: ____________.

<details>
<summary><b>Show answer</b></summary>

**TCP** or **UDP**. Either is acceptable. TCP is reliable, ordered, connection-oriented. UDP is connectionless, unreliable, lower overhead.

</details>

#### Sp26-E1 (V-C, V-D) Q15 — Loopback (fill-in)
The loopback device uses IP address ____________.

<details>
<summary><b>Show answer</b></summary>

**127.0.0.1** (also `localhost`). Routes traffic back to the same host. The full block 127.0.0.0/8 is reserved for loopback.

</details>

#### Sp26-E2 Q7 — Best RAM throughput
What access pattern typically provides the best throughput when reading integers from RAM?
- (A) random  (B) sequential

<details>
<summary><b>Show answer</b></summary>

**(B) sequential.** Sequential accesses benefit from CPU prefetchers, cache line utilization (one fetch brings in 64 bytes = many ints), and DRAM row-buffer hits.

</details>

#### Sp26-E2 Q9 — Virtual address space (fill-in)
The three most important things stored in a virtual address space are code, stack, and the ____________.

<details>
<summary><b>Show answer</b></summary>

**heap.** Code (text segment, read-only program instructions), stack (local variables, call frames — grows down), and heap (dynamic allocations — grows up).

</details>

#### Sp26-E2 Q12 — Fastest cache level (fill-in)
The fastest CPU cache level is L____________ (assume a 3-level cache).

<details>
<summary><b>Show answer</b></summary>

**1 (L1).** Smallest, fastest, closest to the core. L2 is bigger and slower. L3 (LLC) is shared and slowest of the three. RAM is much slower still.

</details>

#### Sp26-E2 Q15 — Cache miss term (fill-in)
When a lookup to a cache CANNOT find a value, the lookup is called a ____________.

<details>
<summary><b>Show answer</b></summary>

**miss.** Found = hit, not found = miss. Hit rate = hits / (hits + misses).

</details>

#### Sp26-E2 Q16 — Python's GC lock (fill-in)
The lock Python uses internally to protect counters related to garbage collection is called the ____________.

<details>
<summary><b>Show answer</b></summary>

**GIL (Global Interpreter Lock).** CPython's GIL serializes bytecode execution to protect reference counts (the GC mechanism). It's why Python threads don't help with CPU-bound parallelism. Python 3.13+ has experimental "no-GIL" builds.

</details>

#### Sp26-E3 Q10 — Replication term (fill-in)
Having multiple copies of data on different machines is called ____________.

<details>
<summary><b>Show answer</b></summary>

**replication.** (As opposed to *partitioning*, which is splitting data across machines.)

</details>

#### Sp26-E3 Q11 — Random access (fill-in)
Jumping around and reading from many different locations in a file is called ____________ access.

<details>
<summary><b>Show answer</b></summary>

**random.** Opposite of sequential access. Generally slower because it defeats prefetching, caching, and (for spinning disks) requires seeks.

</details>

### gRPC / Protocol Buffers (new section, mostly Sp26)

#### Sp26-E1 Q5 — Server-side generated class [V-A, V-B]
With gRPC, what generated class does a server override?
- (A) a servicer class  (B) a stub class

<details>
<summary><b>Show answer</b></summary>

**(A) a servicer class.** Mnemonic: **S**ervicer = **S**erver. The servicer is what protoc generates from your .proto's service definition; you subclass it on the server and implement each RPC method.

</details>

#### Sp26-E1 Q5 — Client-side generated code [V-C, V-D]
With gRPC, what generated code does a client use to make calls?
- (A) servicer code  (B) stub code

<details>
<summary><b>Show answer</b></summary>

**(B) stub code.** The stub is the client-side proxy. It looks like a local object but each method call gets serialized and shipped to the server.

</details>

#### Sp26-E1 Q6 — Older client missing field [V-A, V-B]
An older gRPC client does NOT send a field the gRPC server was expecting. What happens?
- (A) the server crashes
- (B) the server ignores the request
- (C) the server returns an error to the client
- (D) the server uses a default value for missing field

<details>
<summary><b>Show answer</b></summary>

**(D) the server uses a default value for missing field.** Protocol Buffers handle backward compatibility by giving missing fields their default value (0 for ints, "" for strings, etc.). This lets you add new fields to the protocol without breaking older clients.

</details>

#### Sp26-E1 Q6 — Newer client extra field [V-C, V-D]
A newer gRPC client sends a field the gRPC server was NOT expecting. What happens?
- (A) the server crashes
- (B) the server ignores the field
- (C) the server returns an error to the client
- (D) the server uses a default value for missing field

<details>
<summary><b>Show answer</b></summary>

**(B) the server ignores the field.** Forward compatibility: unknown fields are silently skipped during deserialization. The server processes the rest as normal. (Technically the field bytes are preserved in the message — useful for proxy/relay scenarios.)

</details>

#### Sp26-E1 Q10 — .proto: messages and ____ (fill-in) [V-A, V-B]
The two main constructs defined in a .proto file are messages and ____________.

<details>
<summary><b>Show answer</b></summary>

**services.** Messages define data types (request/response payloads). Services define RPC interfaces (the methods the server exposes).

</details>

#### Sp26-E1 (V-C, V-D) Q14 — .proto: services and ____ (fill-in)
The two main constructs defined in a .proto file are services and ____________.

<details>
<summary><b>Show answer</b></summary>

**messages.** Same answer flipped — services + messages are the two big constructs.

</details>

#### Sp26-E1 (V-C, V-D) Q10 / Q11 — gRPC serialization (fill-in)
gRPC uses ____________ for serialization/deserialization.

<details>
<summary><b>Show answer</b></summary>

**Protocol Buffers** (or **protobuf**). The same Protocol Buffers language is used for both schema definition (.proto files) and wire format.

</details>

#### Sp26-E2 Q5 — NOT a gRPC feature
What is NOT a feature built into gRPC?
- (A) for a failed call, it will automatically retry to a different server
- (B) for small integers, it will use variable length encoding to save space
- (C) it allows clients and servers to be written in different programming languages
- (D) it allows clients and servers to have different versions of a protocol in some cases

<details>
<summary><b>Show answer</b></summary>

**(A) automatic retry to a different server.** gRPC has built-in retries (configurable), but they retry the *same* connection — it doesn't natively know about other servers or do failover routing. You'd add a load balancer / service mesh for that. The other three (varint encoding, polyglot interop, version compatibility) are core gRPC/protobuf features.

</details>

#### F24 Q7 (also relevant here) — gRPC serialization
What does gRPC use to serialize messages?
- (A) ColumnIO  (B) JSON  (C) Parquet  (D) Protocol Buffers

<details>
<summary><b>Show answer</b></summary>

**(D) Protocol Buffers.** Repeated for emphasis since this also fits the Networking/Misc category.

</details>

### Databases / ACID (new section, Sp26)

#### Sp26-E2 Q10 — ETL (fill-in)
ETL stands for ____________.

<details>
<summary><b>Show answer</b></summary>

**Extract, Transform, Load.** Pull data from sources (extract), reshape/clean it (transform), then write into the target (load). ELT swaps the order — load first into the warehouse, transform there.

</details>

#### Sp26-E2 Q13 — ACID acronym (fill-in)
ACID stands for ____________, Consistency, Isolation, and Durability.

<details>
<summary><b>Show answer</b></summary>

**Atomicity.** A transaction is all-or-nothing — either every change commits or none does.

</details>

#### Sp26-E3 Q3 — Durability scenario
You don't want your ACID database to lose committed data if your server suddenly loses power. What guarantee is this?
- (A) Atomicity  (B) Consistency  (C) Isolation  (D) Durability

<details>
<summary><b>Show answer</b></summary>

**(D) Durability.** Once a transaction commits, its effects survive crashes, power loss, and restarts. Typically achieved via write-ahead logs flushed (fsync'd) to non-volatile storage before acknowledging the commit.

</details>

### SQL

#### F24 Q18 — Filter before group
If you want to filter rows before they are grouped, what do you use in SQL?
- (A) HAVING  (B) LIMIT  (C) WHERE

<details>
<summary><b>Show answer</b></summary>

**(C) WHERE.** Filter order: FROM → WHERE (row-level filter) → GROUP BY → HAVING (group-level filter) → SELECT → ORDER BY → LIMIT. So WHERE filters before grouping, HAVING after.

</details>

#### Sp24 Q6 — Projection
Which SQL clause enables us to perform projection?
- (A) SELECT  (B) FROM  (C) WHERE  (D) GROUP BY  (E) ORDER BY

<details>
<summary><b>Show answer</b></summary>

**(A) SELECT.** Projection = "pick which columns appear in the output." That's what SELECT does. WHERE is selection (rows). FROM specifies the source. GROUP BY/ORDER BY do other things.

</details>

---

## Question Counts by Topic

| Topic | Total Questions Collected |
|-------|---------------------------|
| HDFS | 14 |
| Spark | 28 |
| Cassandra | 16 |
| Kafka | 13 |
| BigQuery | 20 |
| Other Cloud | 12 |
| Minor systems | 2 (HBase + MapReduce) |
| Review (caching) | 6 |
| Review (concurrency) | 9 |
| Review (idempotency) | 3 |
| Review (Linux) | 13 |
| Review (Docker) | 17 |
| Review (networking/memory/misc) | 21 |
| Review (gRPC / Protocol Buffers) | 9 |
| Review (Databases / ACID) | 3 |
| Review (SQL) | 2 |

## Patterns That Repeat (high-yield for the final)

1. **Cassandra token map (2x replication)** appeared in S25, F25, F24, Sp24. Basically guaranteed.
2. **Cassandra quorums** (R + W > RF) appeared in S25, F25, F24, Sp24.
3. **BigQuery correlated cross join after UNNEST** appeared in S25, F25, F24.
4. **Kafka same-partition guarantees** (key-based hashing) appeared in S25, F25, F24.
5. **Spark streaming partition-by-X but group-by-Y** appeared in S25 and F24 (identical wording).
6. **Spark streaming watermark / stateless detection** appeared in S25, F24, Sp24.
7. **FIFO/LRU cache hit counting** appeared in S25, F25, F24, Sp24, Sp26-E2.
8. **Threading with global + lock** appeared in S25, F25, F24, Sp24, Sp26-E2, Sp26-E3.
9. **Docker SSH tunnel + `-p` port chain** appeared in S25, F24, Sp26-E1, Sp26-E3 (the same chain has now appeared in 4 different exams; expect it on the final).
10. **Idempotency check** appeared in F25, F24, Sp24.
11. **HDFS replication I/O accounting** (write-then-read totals) appeared in S25, Sp26-E3. Variation: client-network bytes with block size appeared in F25 and Sp26-E3.
12. **BHJ vs SMJ choice** with smaller-table-fits-in-memory phrasing appeared in F25 and Sp26-E3 (near-identical wording).
13. **gRPC version compatibility** (older client missing a field, newer client adding a field) is brand new in Sp26-E1 and likely on the final.
14. **MEMORY_ONLY caching levels** appeared in S25, F25, Sp26-E3.

## Topics from the Image Distribution Worth Reviewing Even If Few Questions Above

- **Minor systems** (MySQL, MapReduce, HBase): the final allocates 3 questions here. The Sp26-E3 MapReduce question (counting map calls = 1 per input pair) is the freshest signal. Review HBase region/RegionServer/master architecture, MapReduce phases (map → shuffle → reduce, intermediate disk I/O between phases, 1 map call per input pair, combiners optional), and MySQL leader-follower replication.
- **Other Cloud**: review IaaS vs PaaS vs SaaS, regions/zones/clusters, spot vs on-demand, free tier mechanics, network I/O (ingress free, egress costs), SSD I/O patterns including garbage collection (random writes are worst because of erase-block dynamics), GFLOPS calculations.
- **gRPC / Protocol Buffers**: the Sp26 exams introduce this as a recurring topic. Memorize: servicer = server side, stub = client side; missing field → default value; unknown field → ignored; gRPC does NOT auto-retry; .proto defines messages and services; serialization uses Protocol Buffers with variable-length encoding.
- **ACID guarantees**: Atomicity (all or nothing), Consistency (valid state transitions), Isolation (concurrent txns appear serial), Durability (committed data survives crashes / power loss).
- **Memory hierarchy**: L1 is fastest, sequential RAM access beats random access for throughput, cache miss is the term for failed lookup, virtual address space contains code + stack + heap.
- **Python concurrency specifics**: GIL protects garbage-collection counters, each thread has its own stack for local variables, holding a lock does NOT prevent context switches (a thread holding a lock can still be preempted; the lock just prevents OTHER threads from entering the critical section).
