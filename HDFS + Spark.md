<!-- hdfs-mapreduce-spark-study-guide.md -->

# CS544 Study Guide: HDFS, MapReduce, Spark

This guide is built from the question bank + uploaded slides. Focus on the repeated exam patterns: **HDFS I/O math, NameNode/DataNode roles, MapReduce shuffle, Spark RDD lineage, caching, joins, streaming, and MLlib APIs**.

---

# 0. Big picture: why these systems exist

Traditional single-server databases are good for:
- SQL
- joins
- transactions
- ACID guarantees

But when data becomes too large for one machine, systems need to prioritize:

```text
scalability
elasticity
fault tolerance
availability
```

The Hadoop ecosystem mirrors Google’s internal systems:

| Google paper/system | Hadoop/open-source equivalent | Purpose |
|---|---|---|
| **GFS** | **HDFS** | distributed file system |
| **MapReduce** | **Hadoop MapReduce / Spark** | distributed analytics |
| **BigTable** | **HBase / Cassandra-ish data model influence** | distributed database |

The core idea is building systems on many cheap commodity machines, then handling partitioning, replication, and failure in software.

---

# 1. HDFS

## 1.1 What HDFS is

HDFS = Hadoop Distributed File System.

It stores large files across many machines.

Main components:

| Component | Role |
|---|---|
| **Client** | asks NameNode for metadata, then reads/writes data from/to DataNodes |
| **NameNode** | stores metadata: file → blocks → DataNode locations |
| **DataNode** | stores actual block data as local files |

Exam answer:

```text
To connect to HDFS, client needs the NameNode address.
```

Why: the NameNode tells the client where file blocks live.

The client does **not** need all DataNode addresses upfront.

---

## 1.2 HDFS blocks

HDFS breaks files into blocks.

Example:

```text
File F2 = "EFGHIJKL"
block size = 4 chars

F2.1 = "EFGH"
F2.2 = "IJKL"
```

Large files are partitioned across multiple DataNodes.

Small files may fit in one block.

A block is stored as a local file on a DataNode’s local filesystem.

Example:

```text
40 MB file
32 MB block size

Block 1 = 32 MB
Block 2 = 8 MB
```

The second block does not waste the full 32 MB.

Key idea:

```text
block size is a maximum chunk size, not guaranteed wasted space
```

---

## 1.3 Logical blocks vs physical blocks

A logical block is the abstract HDFS block.

A physical block is an actual replica stored on a DataNode.

If block `F1.1` has replication factor 3:

```text
logical block: F1.1
physical replicas: F1.1 on DN1, F1.1 on DN2, F1.1 on DN3
```

---

## 1.4 Replication

Replication means storing multiple copies of each block.

Example:

```text
replication factor = 3
50 MB file written
disk writes across cluster = 150 MB
```

Because each byte is stored three times.

But reading the file only needs one replica:

```text
read 50 MB file
disk reads across cluster = 50 MB
```

Question-bank formula:

```text
disk writes = file size × replication factor
disk reads = file size
```

Example:

```text
Write 50 MB to 3x replicated HDFS file, then read it.
Written = 150 MB
Read = 50 MB
```

---

## 1.5 Rack awareness

HDFS tries to spread replicas across racks.

Why?

If all replicas are in one rack, a rack failure can lose the block even if replication factor is high.

Exam wording:

```text
HDFS replicates across DataNodes and tries to distribute replicas across racks.
```

---

## 1.6 Detecting DataNode failure

DataNodes send **heartbeats** to the NameNode.

If no heartbeat arrives for long enough:
- node becomes stale
- eventually node is considered dead
- blocks on that node may be under-replicated
- HDFS creates new replicas elsewhere

Question-bank answer:

```text
HDFS detects DataNode failures using heartbeats.
```

Not:
- gossip
- leader election
- checksum
- pipelines

---

## 1.7 HDFS reads

Read flow:

```text
client → NameNode: where are blocks for file F?
NameNode → client: block locations
client → DataNode(s): read block data
```

The NameNode does **not** store file data. It stores metadata.

Exam trap:

```text
NameNode is used to locate blocks.
DataNodes serve the actual bytes.
```

---

## 1.8 Data locality

If computation can run anywhere, run it near the data.

Exam phrase:

```text
co-locate compute with storage
```

Meaning:

```text
run mapper/task on the machine that already has the HDFS block
```

This avoids network I/O.

---

## 1.9 HDFS writes and pipelined writes

For a replicated write, HDFS uses **pipelined writes**.

For replication factor 3:

```text
client → DataNode 1 → DataNode 2 → DataNode 3
```

The client sends each byte only once into the pipeline.

Question-bank formula:

```text
client network bytes = file size
```

Regardless of:
- replication factor
- block size

Example:

```text
Client writes 80 MB to HDFS with 1x replication and 16 MB blocks.
Client sends = 80 MB
```

Example:

```text
Client writes 5 MB to 4x replicated HDFS file.
Client sends = 5 MB
```

The DataNodes forward data between themselves; the client does not send separate copies.

Important contrast:

| System | Replication write style |
|---|---|
| **HDFS** | pipelined writes |
| **Kafka** | leader/follower; followers fetch |
| **Cassandra** | coordinator sends to replicas |

Question-bank trap:

```text
Which system uses pipelined writes?
Answer: HDFS
```

---

## 1.10 HDFS load balancing

There are two different balance problems.

### I/O balance

Some DataNodes may be overloaded because popular blocks live there.

Fix for hot file:

```text
increase replication factor
```

More replicas means more DataNodes can serve reads.

Question-bank pattern:

```text
popular file, rarely changes, regenerated if lost
better performance choice = increase replication factor
```

Why: more replicas spread read traffic.

### Storage balance

Some DataNodes may store much more data than others.

Strategies:
1. Prefer underutilized nodes when writing new data.
2. Run a balancer tool to move data after the fact.

Rebalancing after the fact can cause high network I/O.

---

## 1.11 Block size and load balance

Question-bank pattern:

```text
Load poorly balanced across DataNodes.
What helps?
A) smaller blocks
B) bigger blocks
```

Answer:

```text
smaller blocks
```

Why:

```text
smaller blocks = more blocks = finer-grained distribution across machines
```

Bigger blocks concentrate a large file onto fewer DataNodes, which can make balance worse.

---

## 1.12 HDFS exam formulas

Memorize:

```text
disk writes = file size × replication factor
disk reads = file size
client network send for pipelined write = file size
```

Examples:

```text
5 MB file, RF=2:
disk written = 10 MB
disk read = 5 MB
client sends = 5 MB
```

```text
11 MB file, RF=2, block size=8 MB:
client sends = 11 MB
```

Block size affects number of blocks, not total client bytes.

---

## 1.13 HDFS must-know answers

```text
Client needs NameNode address.
NameNode stores metadata, not file bytes.
DataNodes store actual blocks.
DataNodes send heartbeats.
Replication handles failure.
Pipelined writes reduce client network cost.
Reads use one replica.
Writes store all replicas.
Increase RF for hot popular files.
Smaller blocks can improve balance.
HDFS tries to place replicas across racks.
HDFS is most similar to Colossus/GFS.
```

---

# 2. MapReduce

## 2.1 What MapReduce is

MapReduce is a distributed analytics model.

Instead of writing SQL, you provide:

```text
map function
reduce function
number of mappers/reducers
input files
```

Input/output files are generally in HDFS.

---

## 2.2 Mapper role

A mapper processes input records and emits zero or more key-value pairs.

Example input:

```text
color, shape, size
red, circle, 3
red, square, 5
blue, oval, 1
green, square, 3
```

Question:

```sql
SELECT color FROM table WHERE shape = "square";
```

Mapper:

```python
def map(key, value):
    if value.shape == "square":
        emit(key, value.color)
```

Output:

```text
1 red
3 green
```

Key point:

```text
mapper can emit zero, one, or many key-value pairs per input row
```

---

## 2.3 Reducer role

A reducer receives:

```text
one key + all values for that key
```

MapReduce shuffles intermediate data so that all values with the same key are brought together.

All data with the same key is passed to a single reduce call.

A reduce task generally calls `reduce` many times for different keys.

Example:

```python
def map(key, value):
    emit(value.color, value)

def reduce(key, values):
    count = 0
    for row in values:
        count += 1
    emit(key, count)
```

Input:

```text
red circle
red square
blue oval
green square
```

Intermediate grouped data:

```text
blue → [blue oval]
green → [green square]
red → [red circle, red square]
```

Output:

```text
blue 1
green 1
red 2
```

SQL equivalent:

```sql
SELECT color, COUNT(*)
FROM table
GROUP BY color;
```

---

## 2.4 Shuffle phase

The shuffle phase brings related data together.

In MapReduce:
- mapper emits key-value pairs
- shuffle groups/sorts by key
- reducer processes each key group

Exam phrase:

```text
all intermediate rows with the same key go to the same reducer
```

This matters for:
- `GROUP BY`
- joins
- aggregations
- ordering

---

## 2.5 Multiple reducers

If there are multiple reducers:
- each reduce task produces one output file
- one reducer task may process multiple keys
- a single reduce call handles one key and all corresponding values
- same key cannot be split across reducers

Exam wording:

```text
A reduce task can handle multiple keys, but a reduce call handles one key.
```

---

## 2.6 SQL → MapReduce mental mapping

| SQL operation | MapReduce phase |
|---|---|
| `SELECT` | map and/or reduce |
| `WHERE` | map |
| `GROUP BY` | map + shuffle + reduce |
| `ORDER BY` | shuffle |
| `JOIN` | map + shuffle + reduce |
| aggregates | reduce |
| `HAVING` | reduce |

---

## 2.7 MapReduce with HDFS data locality

MapReduce workers often run on the same machines as HDFS DataNodes.

Goal:

```text
run mapper where the needed HDFS block already exists
```

Then the mapper uses local disk instead of network.

---

## 2.8 MapReduce pipeline weakness

A pipeline of MapReduce jobs often writes intermediate results to HDFS between stages.

Problem:
- writing intermediate data to HDFS is expensive
- replication of recomputable intermediate data can be wasteful
- treating each stage independently prevents whole-pipeline optimization

This motivates Spark: Spark keeps **lineage** instead of always materializing intermediate HDFS files.

---

# 3. Spark fundamentals

## 3.1 What Spark improves over MapReduce

MapReduce:

```text
actual intermediate bytes written to HDFS between jobs
```

Spark:

```text
lineage graph of operations needed to produce data
```

Spark avoids materializing intermediate data unless needed.

Spark RDD ideas:
- data lineage
- lazy evaluation
- immutability

---

## 3.2 RDD

RDD stands for:

```text
Resilient Distributed Dataset
```

Meaning:
- **Resilient**: can recover lost partitions using lineage
- **Distributed**: split across machines
- **Dataset**: collection of records

Question-bank exact fill-in:

```text
RDD = Resilient Distributed Dataset
```

---

## 3.3 Spark lineage

Spark records how to compute an RDD from previous RDDs.

Example:

```python
table = sc.parallelize(data)
double = table.map(mult2)
doubleA = double.filter(onlyA)
doubleA.collect()
```

Here:
- `parallelize` creates an RDD
- `map` creates a new RDD
- `filter` creates a new RDD
- `collect` triggers execution and returns actual data

Operation types:
- transformations
- actions

---

## 3.4 Transformations vs actions

### Transformation

A transformation returns a new RDD/DataFrame and is lazy.

Examples:

```text
map
filter
sample
repartition
where
select
groupBy before action
```

Question-bank fill-in:

```text
A lazy Spark operation that returns an RDD is a transformation.
```

### Action

An action triggers execution and returns/writes actual results.

Examples:

```text
collect
count
mean
save/write
toPandas
```

Question-bank trap:

```text
Which is NOT a Spark transformation?
mean
```

Because `mean` returns a value to the driver and triggers execution.

---

## 3.5 Immutability

RDDs/DataFrames are immutable.

Meaning:

```text
you do not change an RDD in place
you define a new RDD/DataFrame based on an old one
```

Question-bank pattern:

```text
Pipeline and PipelineModel are both immutable.
```

Calling `.fit()` returns a new `PipelineModel`; it does not mutate the original pipeline.

---

## 3.6 Partitions and tasks

Spark data is split into partitions.

Exam rule:

```text
one Spark task typically runs on one core and processes one partition
```

Question-bank answer:

```text
one core, one partition
```

Consequences:
- number of tasks is related to number of partitions
- total parallelism is limited by total cluster cores
- too few partitions → not enough parallelism
- too many partitions → overhead

---

## 3.7 Narrow vs wide transformations

### Narrow transformation

Each input partition maps to exactly one output partition.

Examples:

```text
map
filter
sample
simple union-like operations
```

Question-bank fill-in:

```text
A transformation where each input partition maps to exactly one output partition is narrow.
```

### Wide transformation

Data must move across partitions/machines.

Examples:

```text
groupBy
join
repartition
orderBy
distinct
```

Wide transformations require shuffle/network I/O.

Question-bank concept:

```text
wide transformation = shuffle
narrow transformation = no shuffle
```

---

## 3.8 RDD lineage drawing pattern

Question-bank example:

```python
A = ... # RDD of ints from 0 to 1 million, 10 partitions
B = A.sample(True, 0.01, 544)
C = B.map(lambda x: x * 2).repartition(5)
D = B.mean()
E = C.filter(lambda x: x < 10)
```

How to reason:

| Object | Type | Partitions | Why |
|---|---|---|
| `A` | RDD box | 10 | given |
| `B` | RDD box | 10 | `sample` is narrow, preserves partitions |
| `C` | RDD box | 5 | `map` preserves, `repartition(5)` changes to 5 |
| `D` | materialized value/circle | none | `mean` is action |
| `E` | RDD box | 5 | `filter` preserves partitions |

Edges:
- `A → B`: transformation
- `B → C`: transformation
- `B → D`: action
- `C → E`: transformation

---

# 4. Spark DataFrames and SQL

## 4.1 DataFrame vs RDD

RDD is Spark’s fundamental lower-level data structure.

DataFrame is higher-level:
- table-like
- schema-aware
- supports SQL optimization

Question-bank answer:

```text
Spark fundamental data structure = RDD
```

But in practical Spark SQL, you use DataFrames heavily.

---

## 4.2 Tables vs views

Spark SQL example:

```python
df.write.saveAsTable("mytable")
df.createTempView("myview")
```

### Table

```text
physical data stored, usually parquet files in HDFS
```

Pros:
- faster to query later
- persistent

Cons:
- takes space
- slower to create than view

### View

```text
named query/description of how to get data on demand
```

Pros:
- fast to create
- takes less space

Cons:
- recomputes when queried
- may be slower for repeated computation

Question-bank answer:

```text
saveAsTable creates a Hive table.
```

Not:
- `createTempView`
- `createOrReplaceTempView`
- `createGlobalTempView`

---

## 4.3 Distinct, group by, window functions

### `DISTINCT`

Returns unique values.

```sql
SELECT DISTINCT X FROM table;
```

One output row per unique `X`.

### `GROUP BY`

Groups rows and returns one row per group after aggregation.

```sql
SELECT X, SUM(Y)
FROM table
GROUP BY X;
```

GROUP BY collapses groups into one output row per group.

### Window functions

Window functions compute over partitions but can keep multiple rows per partition.

Difference:

```text
GROUP BY collapses rows.
Window function preserves multiple rows.
```

Window calculations like percentage or row number return multiple rows per partition.

---

## 4.4 Nested/chained grouping

Spark DataFrames can chain groupBys.

Example idea:

```text
group by X,Y → count rows
then group by X → count categories
```

SQL often needs nested queries or `WITH` statements.

DataFrames can chain operations directly.

---

## 4.5 Joins in Spark SQL

An equi-join joins rows where keys match.

Example:

```text
visits.day = performances.day
```

If Tuesday has:
- 2 visits
- 2 performances

Then Tuesday output has:

```text
2 × 2 = 4 rows
```

Because join produces every matching combination.

### Inner join

Only matching rows.

### Left join

Keeps all rows from left table.

If no match on right side:

```text
right-side columns = NULL
```

### “Never saw a performance” pattern

Use:
1. left join
2. group by guest
3. count non-null performances
4. having count = 0

---

# 5. Spark performance and internals

## 5.1 Schema inference

Spark CSV reading patterns:

### Manual schema

```python
spark.read.format("csv").schema("...").load(...)
```

Fastest because Spark does not need to inspect the whole file.

### Header only

```python
.option("header", True)
```

Reads only header, but everything may be string.

### Infer schema

```python
.option("inferSchema", True)
```

Slow because Spark reads the whole file to guess types.

### Parquet

```python
spark.read.format("parquet").load(...)
```

Fast schema load because schema is stored in Parquet metadata.

Exam shortcut:

```text
CSV inferSchema is expensive.
Parquet schema is cheap to read.
Manual schema is fastest.
```

---

## 5.2 Collecting data

`collect()` and `toPandas()` bring data back to the driver/application.

Safe:

```text
small result after filtering/aggregation
```

Dangerous:

```text
large DataFrame result
```

Why:

```text
driver may run out of memory
```

Exam shortcut:

```text
Spark workers process distributed data.
collect/toPandas moves result to one machine.
```

---

## 5.3 Caching and persistence

Use caching when:

```text
you will reuse the same filtered/computed DataFrame many times
```

Example:

```python
df2 = df.where(...)
df2.persist(StorageLevel.MEMORY_ONLY)
```

`df.cache()` is shorthand for:

```python
df.persist(StorageLevel.MEMORY_ONLY)
```

Question-bank answer:

```text
.cache() default = MEMORY_ONLY
```

To stop caching:

```python
df.unpersist()
```

Question-bank fill-in:

```text
df.unpersist()
```

---

## 5.4 Storage levels

### `MEMORY_ONLY`

Stores deserialized JVM objects.

Pros:
- fastest access
- no deserialization cost

Cons:
- uses more memory

Question-bank pattern:

```text
If you have lots of RAM, fastest cache level = MEMORY_ONLY
```

### `MEMORY_ONLY_SER`

Stores serialized bytes.

Pros:
- more compact
- uses less memory

Cons:
- slower access because data must be deserialized

Serialized storage stores each RDD partition as one large byte array and reduces memory usage, but access is slower.

### `DISK_ONLY`

Stores on disk.

Pros:
- uses less RAM

Cons:
- much slower

### `_2` suffix

Examples:

```text
MEMORY_ONLY_2
MEMORY_ONLY_SER_2
DISK_ONLY_2
```

Means 2x replication.

Benefits:
- more choices for where to run task without network transfer
- if worker dies, cached data may not need recomputation

Downside:
- uses twice as much space

Question-bank pattern:

```text
Which cache levels use JVM objects?
MEMORY_ONLY and MEMORY_ONLY_2
```

Because non-`SER` means deserialized JVM objects.

---

## 5.5 Hash partitioning for groups

For grouping:

```sql
SELECT F, MAX(R)
FROM mytable
GROUP BY H, F;
```

Spark partitions by the group-by keys:

```text
H and F
```

Not:
- selected aggregate column `R`
- only displayed column `F`

Question-bank answer:

```text
hash partitioning uses GROUP BY columns
```

Why:

```text
all rows with same group key must be on same machine
```

---

## 5.6 Partial aggregation

For `GROUP BY`, Spark can aggregate partially before shuffle.

Instead of sending every row over network, each partition can compute local partial sums/counts first.

Example:

```text
partition 1 has A: 1, A: 3 → local A total 4
partition 2 has A: 7, A: 8 → local A total 15
shuffle partials → final A total 19
```

This reduces network I/O.

High-yield wording:

```text
partial aggregate = combine locally before shuffle
```

---

## 5.7 Partition coalescing

After filtering or partial aggregation, some shuffle partitions may be tiny.

Spark can coalesce partitions:

```text
many small partitions → fewer larger partitions
```

Why:
- fewer tiny tasks
- less scheduling overhead
- better resource use

Do not confuse:
- `repartition`: usually shuffle, can increase/decrease partitions
- `coalesce`: often reduce number of partitions with less shuffle

---

## 5.8 Parquet bucketing

Bucketing stores data pre-partitioned by a column.

Purpose:

```text
avoid/reduce future shuffle for joins/grouping on that bucket key
```

If two tables are bucketed by the same join key, Spark may not need to fully shuffle both tables for the join.

Exam phrase:

```text
bucketing pre-organizes data so related keys are already colocated
```

---

# 6. Spark joins: BHJ vs SMJ

This is one of the most important Spark exam sections.

## 6.1 Why joins are hard

For join:

```sql
A JOIN B ON A.key = B.key
```

Spark must bring rows with the same key together.

Two main algorithms:

```text
BHJ = Broadcast Hash Join
SMJ = Shuffle Sort Merge Join
```

---

## 6.2 Broadcast Hash Join

BHJ is best when one table is small enough to fit in memory on each worker.

How it works in plain English:

```text
copy the small table to every worker
each worker keeps the small table in memory
each worker scans its piece of the large table
for each large row, it looks up matching small rows locally
```

Network cost roughly:

```text
small table size × number of workers
```

Large table does not need to be shuffled.

Use BHJ when:

```text
one table is small
one table is huge
small table fits in memory on every worker
goal is minimizing network I/O
```

Question-bank examples:

```text
50 machines, 64 GB each
small table = 9.6 GB
big table = 31655.5 GB
choose BHJ
```

```text
20 machines, 8 GB each
small table = 1.5 GB
big table = 2560.8 GB
choose BHJ
```

Because broadcasting the small table is much cheaper than shuffling the huge table.

---

## 6.3 Shuffle Sort Merge Join

SMJ is best when both tables are too large to broadcast.

How it works:

```text
hash partition both tables by join key
send same-key rows to same machines
sort rows by key
merge matching sorted rows
```

Question-bank answer:

```text
SMJ uses hash partitioning to bring matching rows from both tables together.
```

Use SMJ when:

```text
both tables are large
small side does not fit in memory on every worker
```

Downside:
- shuffles both tables
- high network I/O

---

## 6.4 BHJ vs SMJ exam shortcut

| Situation | Choose |
|---|---|
| one table small enough to fit in worker memory | **BHJ** |
| both tables large | **SMJ** |
| asked “uses hash partitioning to bring matching rows together” | **SMJ** |
| asked “minimize network I/O and small table fits” | **BHJ** |

Important nuance:

```text
BHJ is less versatile than SMJ.
```

Why:
- BHJ requires one side to fit in memory on every worker.
- SMJ works for large-large joins.

---

# 7. Spark streaming

## 7.1 Stateless vs stateful streaming

### Stateless query

Each row can be processed independently.

Example:

```sql
SELECT 1/x AS inverse
FROM some_stream;
```

Answer:

```text
stateless
```

No memory of previous rows needed.

### Stateful query

Needs memory across rows/events.

Example:

```sql
SELECT MAX(x)
FROM mystream;
```

Answer:

```text
stateful
```

Because Spark must remember the current max.

Question-bank exact patterns:
- `SELECT 1/x` → stateless
- `SELECT MAX(x)` → not stateless

---

## 7.2 Watermarks

Watermarks tell Spark how long to keep state for late-arriving data.

Question-bank example:

```python
animals.withWatermark("timestamp", "2 hours")
       .groupBy(window("timestamp", "4 hours"))
       .count()
```

Question:

```text
count for interval starting at 3pm
when can Spark discard state?
```

Work:

```text
window starts 3pm
window length = 4 hours
window ends 7pm
watermark delay = 2 hours
discard around 9pm
```

Answer:

```text
9pm
```

Formula:

```text
discard time = window end + watermark delay
```

---

## 7.3 Kafka partition mismatch with Spark streaming

Question-bank pattern:

```text
Kafka topic partitioned by X.
Spark query groups by Y.
What happens?
```

Answer:

```text
Spark still computes correctly.
```

Why:

```text
Kafka partitioning is only input layout.
Spark can reshuffle internally by Y.
```

It may cost performance, but correctness is fine.

---

## 7.4 Unsupported streaming operation

Question-bank pattern:

```text
Which is not supported by Spark streaming?
pivots
```

Why:

```text
pivot needs knowing all distinct values up front
stream is unbounded
```

---

# 8. Spark MLlib

## 8.1 ML categories

Classic categories:

| Type | Meaning |
|---|---|
| **Supervised learning** | train on known input/output pairs |
| **Unsupervised learning** | find patterns without labels |
| **Reinforcement learning** | agent takes actions to maximize reward |

Classification vs regression:

```text
classification = categorical output
regression = numeric output
```

---

## 8.2 Train / validation / test

Typical workflow:
1. split data into train/validation/test
2. train models on train data
3. choose model using validation data
4. report final quality on test data
5. deploy model

Overfitting:

```text
model performs well on train data but badly on validation/test data
```

---

## 8.3 Spark MLlib API: estimator vs transformer

Spark MLlib uses DataFrames.

Important difference from scikit-learn:

### Spark estimator

Unfit model object.

```python
unfit_model = DecisionTreeRegressor(...)
fit_model = unfit_model.fit(df)
```

`.fit()` returns a new fitted model object.

Question-bank answer:

```text
DecisionTreeRegressor = unfit estimator
DecisionTreeRegressionModel = fitted model
```

### Spark transformer

Object with `.transform(df)`.

A fitted model is a transformer because it adds predictions to a DataFrame.

Prediction in Spark:

```python
df2 = fit_model.transform(df)
```

Not:

```python
fit_model.predict(...)
```

Question-bank answer:

```text
make predictions using transform on fitted model
```

---

## 8.4 Features column

Spark MLlib usually wants one `features` column.

That column usually contains a vector made from multiple original columns.

Example:

```text
x1 = 2
x2 = 3
features = (2,3)
```

---

## 8.5 Pipeline

A pipeline is a series of stages:
- transformers
- estimators

Spark example:

```text
Pipeline p: T → T → E
m = p.fit(df)
PipelineModel m: T → T → T
m.transform(df)
```

Meaning:
- during fitting, transformers transform data
- estimator gets fit and becomes a transformer
- final `PipelineModel` is all transformers

Question-bank pattern:

```text
Pipeline and PipelineModel are immutable.
```

---

## 8.6 `pyspark.mllib` vs `pyspark.ml`

| Package | Based on |
|---|---|
| `pyspark.mllib` | RDDs |
| `pyspark.ml` | DataFrames |

Question-bank likely wording:

```text
modern Spark ML uses pyspark.ml DataFrame API
```

---

# 9. Decision trees and PLANET

## 9.1 Decision tree prediction

A decision tree is like nested `if/else`.

Example:

```python
if salary < 50000:
    return False
else:
    if commute > 1h:
        return False
    else:
        return True
```

Features and labels can be numeric or categorical.

---

## 9.2 How tree leaves predict

For regression:
- leaf prediction is often average label value in that leaf

For classification:
- leaf prediction is majority class or class probability

Example:
- if a new point lands in a leaf with labels around `5.0` and `5.2`, prediction is about `5.1`
- if it lands in a leaf with labels `8.6` and `8.8`, prediction is about `8.7`

---

## 9.3 Impurity

Impurity measures how mixed/non-uniform labels are inside a node/leaf.

For regression, think:

```text
variance
```

Low impurity = labels in a leaf are similar.

High impurity = labels are mixed.

A good split reduces impurity.

---

## 9.4 Choosing splits

To train a decision tree:
1. start with all data in root node
2. try possible feature split points
3. choose split that reduces impurity most
4. recursively split child nodes
5. stop based on criteria

Stopping criteria:
- max tree height
- minimum rows required to split
- pruning later

---

## 9.5 Overfitting in trees

A very complex tree may fit training data too closely.

Simple tree:
- less exact on training data
- may generalize better

Complex tree:
- very exact on training data
- may do worse on new data

---

## 9.6 Ensembles

Ensemble = many simple models vote/combine.

Examples:
- random forest
- gradient-boosted trees

Random forest:
- many trees
- each trained on subset of rows/features
- predictions combined by voting/averaging

Spark can train many trees in a random forest simultaneously across the cluster.

---

## 9.7 PLANET algorithm

PLANET = Parallel Learner for Assembling Numerous Ensemble Trees.

Spark `DecisionTreeRegressor` and `DecisionTreeClassifier` use it too.

Main idea:
- if node has few rows and fits in worker memory, split in memory
- if node has lots of data, use simplified/distributed approach with fewer split points

Question-bank fact:

```text
PLANET uses split thresholds between equi-width histogram bins.
```

Exam intuition:

```text
PLANET avoids trying every exact split point across huge distributed data.
```

---

# 10. Spark exam-specific answer patterns

## 10.1 Hash partitioning columns

Question:

```sql
SELECT F, MAX(R)
FROM mytable
GROUP BY H, F;
```

Answer:

```text
hash partition on H and F
```

Because group key is `(H, F)`.

---

## 10.2 Join algorithm

Question:

```text
Which join uses hash partitioning to bring matching rows together?
```

Answer:

```text
SMJ
```

Question:

```text
One table small, one table huge, small table fits memory, minimize network I/O.
```

Answer:

```text
BHJ
```

---

## 10.3 Cache level

Question:

```text
.cache() is shorthand for what?
```

Answer:

```text
MEMORY_ONLY
```

Question:

```text
Lots of RAM, fastest cache level?
```

Answer:

```text
MEMORY_ONLY
```

Question:

```text
Which use JVM object representation?
```

Answer:

```text
MEMORY_ONLY and MEMORY_ONLY_2
```

Question:

```text
Stop caching DataFrame df?
```

Answer:

```python
df.unpersist()
```

---

## 10.4 RDD/action/transformation

Question:

```text
RDD stands for?
```

Answer:

```text
Resilient Distributed Dataset
```

Question:

```text
Lazy operation returning RDD?
```

Answer:

```text
transformation
```

Question:

```text
One-to-one partition mapping?
```

Answer:

```text
narrow transformation
```

Question:

```text
Not a transformation: filter, parallelize, mean, map
```

Answer:

```text
mean
```

---

## 10.5 Spark ML

Question:

```text
Unfit decision tree type?
```

Answer:

```text
DecisionTreeRegressor
```

Question:

```text
Fitted decision tree type?
```

Answer:

```text
DecisionTreeRegressionModel
```

Question:

```text
Make predictions using Spark ML?
```

Answer:

```text
transform on fitted model
```

Question:

```text
Are Pipeline and PipelineModel immutable?
```

Answer:

```text
yes, both
```

---

## 10.6 Streaming

Question:

```text
SELECT MAX(x) FROM stream
```

Answer:

```text
stateful
```

Question:

```text
SELECT 1/x FROM stream
```

Answer:

```text
stateless
```

Question:

```text
4-hour window starting 3pm, 2-hour watermark
```

Answer:

```text
discard state at 9pm
```

Question:

```text
Kafka partitioned by X, Spark groups by Y
```

Answer:

```text
Spark still gives correct results by reshuffling
```

Question:

```text
Unsupported Spark streaming op?
```

Answer:

```text
pivot
```

---

# 11. High-yield comparison: HDFS vs MapReduce vs Spark

| Concept | HDFS | MapReduce | Spark |
|---|---|---|---|
| Main role | distributed storage | distributed batch processing | distributed analytics engine |
| Core unit | block | key-value pair | RDD/DataFrame partition |
| Fault tolerance | replication | HDFS files + rerun tasks | lineage recomputation |
| Data movement | read/write blocks | shuffle key groups | shuffle for wide transformations |
| Intermediate data | stored files | writes to HDFS between jobs | lineage, cached if needed |
| Locality | client/task near DataNode | mappers near HDFS blocks | tasks on partitions, locality preferred |
| Exam identity | NameNode/DataNode/blocks | map/shuffle/reduce | RDD/transform/action/cache/join |

---

# 12. Final memorization sheet

Study this last.

```text
HDFS:
Client needs NameNode address.
NameNode stores metadata: file → blocks → DataNodes.
DataNodes store actual block bytes.
Files are split into blocks.
Block size caps max block size; final block can be smaller without wasting full block.
Replication factor = number of physical copies.
Disk writes = file size × RF.
Disk reads = file size.
Client network bytes for pipelined write = file size.
HDFS detects DataNode failure using heartbeats.
Dead DataNode → under-replicated blocks → new replicas.
HDFS tries to spread replicas across racks.
Increase RF for hot/read-heavy files.
Smaller blocks can improve load balance.
HDFS uses pipelined writes.
HDFS is most similar to Colossus/GFS.

MapReduce:
Mapper emits zero or more key-value pairs.
Shuffle groups/sorts intermediate data by key.
Reducer gets one key and all values for that key.
All same-key data goes to same reducer.
One reduce task can process many keys.
Each reduce task writes one output file.
MapReduce input/output usually in HDFS.
Mappers should run near HDFS blocks for data locality.
MapReduce pipelines write intermediate data to HDFS, which is inefficient.
Spark improves by keeping lineage instead of materializing every stage.

Spark:
RDD = Resilient Distributed Dataset.
RDDs are immutable.
Lineage records how to recompute data.
Transformations are lazy and return RDD/DataFrame.
Actions trigger execution and return/write actual results.
mean is an action.
map/filter are transformations.
Narrow transformation = one input partition to one output partition.
Wide transformation = shuffle.
One Spark task usually runs on one core and one partition.
collect/toPandas can OOM driver if result is large.
cache() = persist(MEMORY_ONLY).
MEMORY_ONLY = fastest if enough RAM, deserialized JVM objects.
MEMORY_ONLY_SER = less memory, slower.
_2 means replicated twice.
unpersist removes cached data.
GROUP BY hash partitions by group-by columns.
SMJ shuffles both tables by join key.
BHJ copies small table to all workers.
Choose BHJ when one table fits in memory and other is huge.
Choose SMJ when both are large.
saveAsTable creates Hive table.
Temp view is a named query, cheaper to create, less space.
GroupBy collapses rows; window functions preserve rows.
Spark ML prediction uses transform on fitted model.
Unfit model = estimator; fitted model = transformer.
Decision trees are nested if/else.
Impurity measures label non-uniformity.
PLANET trains decision trees on distributed data using simplified split points for large nodes.
```

---

# 13. Final self-test

## HDFS

1. Client needs what to connect to HDFS?  
**NameNode address.**

2. Who stores actual file bytes?  
**DataNodes.**

3. Who stores metadata?  
**NameNode.**

4. How does HDFS detect DataNode failure?  
**Heartbeats.**

5. Write 20 MB with RF=3. Disk writes?  
**60 MB.**

6. Read that 20 MB file. Disk reads?  
**20 MB.**

7. Client writes 20 MB with RF=3 using pipelined writes. Client network bytes?  
**20 MB.**

8. Popular HDFS file overloaded on reads. Fix?  
**Increase replication factor.**

9. Poor load balance across DataNodes. Smaller or bigger blocks?  
**Smaller blocks.**

10. Which system uses pipelined writes?  
**HDFS.**

---

## MapReduce

11. Mapper emits what?  
**Zero or more key-value pairs.**

12. Shuffle does what?  
**Groups/sorts intermediate data by key and brings same-key data together.**

13. Reducer receives what?  
**One key and all values for that key.**

14. Can one reduce task process multiple keys?  
**Yes.**

15. Can same key go to multiple reducers?  
**No.**

16. What is inefficient about MapReduce pipelines?  
**Intermediate data is repeatedly written to HDFS.**

---

## Spark

17. RDD stands for?  
**Resilient Distributed Dataset.**

18. Lazy operation returning RDD?  
**Transformation.**

19. Operation that triggers execution?  
**Action.**

20. Is `mean` a transformation or action?  
**Action.**

21. One task usually runs on what?  
**One core, one partition.**

22. Narrow transformation means?  
**Each input partition maps to one output partition.**

23. Wide transformation means?  
**Shuffle/network movement.**

24. `.cache()` uses what storage level?  
**MEMORY_ONLY.**

25. Stop caching?  
**`df.unpersist()`**

26. Best cache level with lots of RAM?  
**MEMORY_ONLY.**

27. Which join broadcasts small table?  
**BHJ.**

28. Which join shuffles both by key?  
**SMJ.**

29. Query groups by `H, F`; hash partition on what?  
**H and F.**

30. Spark ML prediction method?  
**`transform()` on fitted model.**

31. Unfit decision tree class?  
**DecisionTreeRegressor.**

32. Fitted decision tree class?  
**DecisionTreeRegressionModel.**

33. Window 3pm–7pm with 2-hour watermark: discard when?  
**9pm.**

34. Kafka partitioned by X, Spark groups by Y. Correct?  
**Yes, Spark reshuffles by Y.**

35. Unsupported streaming operation from bank?  
**Pivot.**
