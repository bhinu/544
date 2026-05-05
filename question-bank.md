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

### S25 Q11 — Load balancing
In an HDFS cluster, load is poorly balanced across DataNodes. What is most likely to help improve balance?
- (A) using smaller blocks
- (B) using bigger blocks

### S25 Q24 — Replication I/O
You write 50 MB to a 3x replicated file in HDFS, then later read it back. How much data will be read and written to disks across the cluster?
- (A) 50 MB written, 150 MB read
- (B) 150 MB written, 50 MB read
- (C) 50 MB written, 50 MB read
- (D) 150 MB written, 150 MB read

### F25 Q4 — Client network bytes
A client writes a 80 MB file to HDFS with 1x replication. The block size is 16 MB. How much data does the client send over the network?
- (A) 5 MB  (B) 16 MB  (C) 80 MB  (D) 112 MB  (E) 400 MB

### F25 Q11 — Detecting live DataNodes
How does a NameNode determine which DataNodes are live in the cluster?
- (A) gossip  (B) leader election  (C) heartbeats  (D) pipelines  (E) checksums

### F25 Q24 — Hot file
You have an HDFS file, F, that rarely changes, but it is very popular: clients read F so often that DataNodes storing blocks of the file cannot keep up with requests. It will not be catastrophic if F is lost, because you can just execute a MapReduce job to regenerate the contents of F as needed. What would be better, from a performance perspective?
- (A) decrease replication factor for F
- (B) increase replication factor for F

### F24 Q9 — Popular file
A single HDFS file is being read by many different clients, and HDFS is having trouble keeping up. What is most likely to help?
- (A) disable pipelined writes
- (B) increase the replication factor
- (C) decrease the replication factor

### F24 Q11 — Pipelined writes
Which system uses pipelined writes to send data to all the workers that will store a new piece of data?
- (A) HDFS  (B) Spark  (C) Cassandra  (D) Kafka

### F24 Q24 — Failure detection
What technique does HDFS use to DETECT DataNode failures?
- (A) partitioning  (B) replication  (C) heartbeats  (D) block maps  (E) hashing

### F24 Q29 — Leader/follower replication
Which system(s) have a leader/follower approach to replication?
- (A) only HDFS  (B) only Spark  (C) only Cassandra  (D) only Kafka  (E) both HDFS and Cassandra

### Sp24 Q8 — Pipelined writes math
A client is writing 5 MB of data to a 4x replicated HDFS file. Assuming pipelined writes, how much data does the client send over the network?
- (A) 4 MB  (B) 5 MB  (C) 9 MB  (D) 20 MB

### Sp24 Q26 — Similar system
Which of the following is most similar to HDFS?
- (A) Colossus  (B) BigTable  (C) BigQuery  (D) HBase  (E) Dynamo

### Sp26-E3 Q5 — 2x replication I/O
You write 5 MB to a 2x replicated file in HDFS, then later read it back. How much data will be read and written to disks across the cluster?
- (A) 5 MB written, 10 MB read
- (B) 10 MB written, 10 MB read
- (C) 10 MB written, 5 MB read
- (D) 5 MB written, 5 MB read

### Sp26-E3 Q6 — Network bytes (2x replication, 11 MB / 8 MB blocks)
A client writes a 11 MB file to HDFS with 2x replication. The block size is 8 MB. How much data does the client send over the network?
- (A) 8 MB  (B) 11 MB  (C) 16 MB  (D) 22 MB  (E) 32 MB

---

## Spark

### S25 Q3 — Hash partitioning columns
For the below Spark SQL query, over which column(s) will hash values be calculated for hash partitioning?
```
SELECT F, MAX(R) FROM mytable GROUP BY H, F;
```
- (A) F  (B) R  (C) F and H  (D) H  (E) R and F

### S25 Q14 — Join algorithm with hash partitioning
Which join algorithm uses hash partitioning to bring rows from each table that could potentially match with each other together on the same machine?
- (A) SMJ  (B) BHJ

### S25 Q17 — Streaming partition mismatch
Say you want to run a streaming Spark query over a Kafka topic. The topic is partitioned by column X, but the query is grouping by a different column, Y. What will happen?
- (A) Spark will refuse to run the query
- (B) Spark will produce incorrect outputs
- (C) Spark will be able to group correctly by column Y

### S25 Q19 — Caching levels using JVM types
Which Spark caching levels will use the JVM types to represent data?
- (A) MEMORY_ONLY and MEMORY_ONLY_2
- (B) MEMORY_ONLY_SER and MEMORY_ONLY_SER_2
- (C) MEMORY_ONLY AND MEMORY_ONLY_SER
- (D) MEMORY_ONLY_2 AND MEMORY_ONLY_SER_2

### S25 Q20 — Watermark discard time
A Spark streaming query is maintaining a count for an interval starting at 3pm. At what time could Spark reasonably discard the running count for events occuring in this interval?
```python
(animals.withWatermark("timestamp", "2 hours")
        .groupBy(window("timestamp", "4 hours"))
        .count())
```
- (A) 11pm  (B) 10pm  (C) 9pm  (D) 7pm  (E) 5pm

### F25 Q3 — `.cache` default
In Spark, `.cache` is a convenience method that calls `.persist(...)` with what setting?
- (A) MEMORY_ONLY  (B) MEMORY_ONLY_SER  (C) DISK_ONLY  (D) DISK_ONLY_2

### F25 Q14 — Choosing join
You have a Spark cluster with 50 machines, each with 64 GB of memory. You need to join two tables. Smaller table: 9.6 GB. Bigger table: 31655.5 GB. As long as you don't run out of memory, your goal should be to minimize network I/O.
- (A) SMJ (Shuffle Sort Merge Join)
- (B) BHJ (Broadcast Hash Join)

### F24 Q3 — Task fundamentals
A single Spark task typically runs on _________ and operates on __________.
- (A) one core, one partition
- (B) one core, many partitions
- (C) multiple cores, one partition
- (D) multiple cores, many partitions

### F24 Q4 — When BHJ helps
When is BHJ most beneficial?
- (A) both tables are small
- (B) one table is small and one is large
- (C) both tables are large

### F24 Q17 — Streaming stateless
In Spark streaming, is the following stateless?
```sql
SELECT MAX(x) AS total FROM mystream;
```
- (A) yes  (B) no

### F24 Q19 — Immutability
Which of the following are immutable in Spark?
- (A) only Pipeline  (B) only PipelineModel  (C) both Pipeline and PipelineModel

### F24 Q20 — Streaming partition mismatch (repeat pattern)
Say you want to run a streaming Spark query over a Kafka topic. The topic is partitioned by column X, but the query is grouping by a different column, Y. What will happen?
- (A) Spark will refuse to run the query
- (B) Spark will produce incorrect outputs
- (C) Spark will be able to group correctly by column Y

### Sp24 Q2 — Unsupported streaming op
Which of the following operations is not supported by Spark streaming?
- (A) group by  (B) group by time intervals  (C) inner join  (D) watermarks  (E) pivots

### Sp24 Q3 — Hive table method
Which of the following methods enables us to create a HIVE table?
- (A) createTempView  (B) createOrReplaceTempView  (C) createGlobalTempView  (D) saveAsTable

### Sp24 Q7 — Small fits-in-memory join
Suppose we want to use Spark to join two tables using a small number of worker machines. If one of those tables fits entirely into memory, which of the following join algorithms should we pick?
- (A) Broadcast Hash Join  (B) Shuffle Sort Merge Join

### Sp24 Q9 — Fundamental data structure
Which of the following is the fundamental data structure of Spark?
- (A) DataFrame  (B) table  (C) view  (D) protocol buffer  (E) RDD

### Sp24 Q11 — Unfit decision tree type
Which Spark type is used for a decision tree model that has NOT been fit to the data yet?
- (A) DecisionTreeRegressor  (B) DecisionTreeRegressionModel

### Sp24 Q16 — Filter lambda
Suppose `banks_df` is a Spark DataFrame containing a column called "name". Which of the following lambda expressions enables us to filter all rows that contain "First" as part of the bank names?
- (A) `lambda banks_df: "First" == banks_df["name"]`
- (B) `lambda b: "First" in b["name"]`
- (C) `lambda banks_df: "First" in banks_df["name"]`
- (D) `lambda b: "First" == b["name"]`

### Sp24 Q17 — ML predictions
How do you make predictions using Spark ML implementations?
- (A) invoke transform method on unfit model
- (B) invoke predict method on unfit model
- (C) invoke fit method on fitted model
- (D) invoke predict method on fitted model
- (E) invoke transform method on fitted model

### Sp24 Q29 — Similar system
Which of the following systems is most similar to Spark?
- (A) Colossus  (B) BigTable  (C) BigQuery  (D) HBase  (E) Dynamo

### Sp24 Q30 — Not a transformation
Which of the following is NOT a Spark transformation operation?
- (A) filter  (B) parallelize  (C) mean  (D) map

### Sp24 Q32 — Streaming stateless
Consider Spark streaming. Is the following query stateless?
```sql
SELECT 1/x AS inverse FROM some_stream;
```
- (A) yes  (B) no

### Sp26-E3 Q4 — Caching speed with lots of RAM
If you have lots of RAM, which caching level will generally be fastest?
- (A) MEMORY_ONLY  (B) MEMORY_ONLY_SER  (C) DISK_ONLY

### Sp26-E3 Q8 — Choosing join (BHJ vs SMJ)
You have a Spark cluster with 20 machines, each with 8 GB of memory. You need to join two tables. Smaller table: 1.5 GB. Bigger table: 2560.8 GB. Goal: minimize network I/O.
- (A) SMJ (Shuffle Sort Merge Join)
- (B) BHJ (Broadcast Hash Join)

### Sp26-E3 Q9 — RDD acronym (fill-in)
RDD stands for ____________

### Sp26-E3 Q12 — Lazy operation returning an RDD (fill-in)
A lazy Spark operation that returns an RDD is called a(n) ____________

### Sp26-E3 Q13 — One-to-one partition mapping (fill-in)
A Spark transformation where each input partition maps to exactly one output partition is called a ____________ transformation

### Sp26-E3 Q15 — Stop caching a DataFrame (fill-in)
To stop caching a Spark DataFrame df, call df.____________()

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

---

## Cassandra

### S25 Q2 — Design inspiration
What are two systems that inspired the design of Cassandra?
- (A) BigTable and Dynamo
- (B) BigTable and MapReduce
- (C) BigQuery and Dynamo
- (D) BigQuery and MapReduce

### S25 Q21 — Token map (2x replication)
Assuming 2x replication, which node(s) are responsible a new row being inserted?
Row: `x="red", y="green", z="blue"`. The primary key is `("x", "y")`.
Hashes: `hash("red")=6, hash("green")=-4, hash("blue")=-8, hash(<"red","green">)=2, hash(<"red","green","blue">)=-6`.
Token map: `token(n1) = [3, 4, -2], token(n2) = [-5, -3, -1], token(n3) = [-8]`
Reference: `-8 | -7 | -6 | -5 | -4 | -3 | -2 | -1 | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

### S25 Q22 — Quorums (R, W given)
For Cassandra, R=5 and W=2. Readers are guaranteed to see previous writes. What can we infer about RF? Choose the answer that provides the tightest bound on RF.
- (A) RF >= 7  (B) RF >= 8  (C) RF < 7  (D) RF > 7  (E) RF < 8

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

### F25 Q13 — Quorums (R, RF given)
Cassandra Quorums: Given R=5 and RF=10, what should W be to make sure readers see successful writes? If multiple satisfy this, choose the smallest correct.
- (A) 1  (B) 4  (C) 6  (D) 8

### F25 Q21 — CAP priority
What characteristic does Cassandra's design prioritize?
- (A) availability  (B) atomicity  (C) consistency  (D) isolation

### F25 Q22 — Token map (2x replication)
Assuming 2x replication, which node(s) are responsible for a new row being inserted?
Row: `x="red", y="green", z="blue"`. Primary key: `("x", "y")`.
Hashes: `hash("red")=2, hash("green")=-2, hash("blue")=-5, hash(<"red","green">)=4, hash(<"red","green","blue">)=-8`.
Token map: `token(n1) = [4, -4], token(n2) = [-1, -5], token(n3) = [6]`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

### F24 Q2 — Token map (2x replication, given token)
Assuming 2x replication, which node(s) are responsible for row token 3?
Token map: `token(n1) = [-7], token(n2) = [7], token(n3) = [4]`
- (A) only n1  (B) only n2  (C) n1+n2  (D) n1+n3  (E) n2+n3

### F24 Q21 — Gossip protocol
Which of the following uses a gossip protocol for updating information about cluster membership?
- (A) HDFS  (B) Spark  (C) Cassandra  (D) Kafka

### F24 Q22 — Quorums (W, RF given)
Cassandra Quorums: Given W=4 and RF=9, what should R be to make sure readers see successful writes? Smallest correct.
- (A) 2  (B) 4  (C) 5  (D) 6

### Sp24 Q10 — Read deduplication
Which of the following techniques is used to avoid reading identical copies of the same data when Cassandra read quorum R > 0?
- (A) pipelined reads  (B) caching  (C) checksum  (D) compression

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

### Sp24 Q21 — New node joining
Same token map as above. Node n5 joins with vnodes 9 and -4. Which existing node(s) will pass off some data to this new node?
- (A) n1 and n2
- (B) n2 and n3
- (C) n2 and n4
- (D) n1 and n4
- (E) n4 and n5

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

### Sp24 Q31 — Wrapping range
Same token map as Q20. What is the wrapping range?
- (A) > 15 <= 20  (B) > 20  (C) >= 20  (D) <= -5  (E) < -5

### Sp24 Q33 — Quorums (RF, R given)
Considering Cassandra quorums, suppose RF=10 and R=5, what should W be to make sure that we read the latest successful write?
- (A) 1  (B) 2  (C) 4  (D) 5  (E) 6

---

## Kafka

### S25 Q10 — Leader election eligibility
A Kafka partition leader fails, and there are three followers. Which are eligible to become the new leader?
- Follower 1: in-sync, and has all messages that the old leader had
- Follower 2: in-sync, but is missing 10 messages that the old leader had
- Follower 3: lagging, but is missing 1 message that the old leader had
- (A) only 1  (B) 1 or 2  (C) 1 or 3  (D) 1, 2, or 3

### S25 Q13 — Replication write path
A Kafka topic has a replication factor of 3. How will new data be written to the replicas?
- (A) The client will write the message directly to the leader and both followers.
- (B) The client will write the message to the leader, and the followers will later fetch it.
- (C) The client will write the message to the leader, which will actively send it to both followers.
- (D) The client will send the data to the leader, the leader will send it to the first follower, and the first follower will send it to the second follower.
- (E) The client will send the data to the first follower, the first follower will send it to the second follower, and the second follower will send it to the leader, at which point it will be committed.

### S25 Q16 — Consumer groups
There are 4 Kafka groups, each with 5 consumer(s). All groups are subscribed to topic T. A new message in T will be consumed how many times?
- (A) 20  (B) 7  (C) 5  (D) 4  (E) 1

### S25 Q29 — Same partition guarantees
What can we guarantee about which messages will go to the same partition?
1. `topic="purple", key="green", value="red"`
2. `topic="red", key="green", value="red"`
3. `topic="green", key="red", value="red"`
- (A) 1 and 2 will go to the same partition
- (B) 1 and 3 will go to the same partition
- (C) 2 and 3 will go to the same partition
- (D) We can't guarantee anything

### F25 Q10 — Consumer groups (repeat)
There are 4 Kafka groups, each with 5 consumer(s). All groups are subscribed to topic T. A new message in T will be consumed how many times?
- (A) 1  (B) 3  (C) 4  (D) 5  (E) 20

### F25 Q16 — Same partition guarantees
1. `topic="green", key="purple", value="red"`
2. `topic="red", key="blue", value="purple"`
3. `topic="green", key="green", value="red"`
- (A) 1 and 2  (B) 1 and 3  (C) 2 and 3  (D) Can't guarantee

### F25 Q20 — Committed?
Given the following Kafka partition state, is message E committed?
- Leader: A, B, C, D, E
- Follower 1 (lagging): A, B
- Follower 2 (in-sync): A, B, C, D, E
- Follower 3 (in-sync): A, B
- (A) Yes  (B) No

### F25 Q29 — No custom code
For which one do you NOT usually need to write custom code when using Kafka?
- (A) producers  (B) brokers  (C) consumers

### F24 Q13 — Same partition guarantees
1. `topic="W", key="Z", value="X"`
2. `topic="W", key="X", value="X"`
3. `topic="X", key="Z", value="Z"`
- (A) 1 and 2  (B) 1 and 3  (C) 2 and 3  (D) Can't guarantee

### F24 Q16 — High volume bottleneck
You have a high-volume Kafka topic. The brokers can keep up, but the consumers cannot. What is most likely to help?
- (A) use more topic partitions
- (B) use fewer topic partitions
- (C) start more consumer groups
- (D) start more consumers per consumer group

### Sp24 Q5 — Delivery semantics
Producer produced 10, 20, 30 to topic "nums". A Spark streaming query computing the product produces 600. What semantics?
- (A) at-most-once  (B) at-least-once  (C) exactly-once

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

### Sp24 Q14 — Uncommitted fetches
In Kafka, both consumers and followers send fetch requests to the leader. Who can fetch uncommitted messages in Kafka?
- (A) only consumers
- (B) only followers
- (C) both consumers and followers
- (D) neither consumers nor followers

### Sp24 Q18 — Min commits with min.insync.replicas
RF=5, min.insync.replicas=3. There are 4 in-sync replicas and 1 lagging. Minimum number of replicas a message must be written to in order to be considered committed?
- (A) 1  (B) 2  (C) 3  (D) 4  (E) 5

---

## BigQuery

### S25 Q1 — ML clause
Which clause related to machine-learning does BigQuery add to SQL?
- (A) TEST  (B) TRAIN  (C) TRANSFORM  (D) TRANSPOSE

### S25 Q5 — Billing using leftovers
Which BigQuery billing model uses "leftover" CPU and memory resources?
- (A) capacity  (B) on-demand  (C) rollover  (D) spare

### S25 Q26 — Internal engine
The query engine for BigQuery is internally based on what system?
- (A) GFS  (B) Dremel  (C) Spark  (D) MapReduce

### S25 Q27 — Correlated cross join unnest
If you do a correlated cross join between columns y and z (after unnesting each), how many rows will you get?
```
x, y, z
1, [2, 3], [4]
5, [6, 7], [8, 9, 10]
```
- (A) 2  (B) 4  (C) 7  (D) 8  (E) 16

### S25 Q28 — Parquet inspiration
Which format inspired Parquet?
- (A) Arrow  (B) Capacitor  (C) ColumnIO  (D) Protocol Buffers

### F25 Q1 — Storage I/O rounding
When BigQuery computes query cost based on bytes of storage I/O, how does it round I/O?
- (A) rounds up  (B) rounds down

### F25 Q5 — Non-cloud analog
Which non-cloud platform is most similar to Google's BigQuery?
- (A) Spark  (B) Cassandra  (C) Kafka  (D) HBase  (E) BigTable

### F25 Q19 — PLANET split points
What split points does PLANET consider?
- (A) between any unique values
- (B) thresholds between bins in an equi-width histogram
- (C) thresholds between bins in an equi-depth histogram

### F25 Q26 — SQLX origin
SQLX is an extension of SQL provided by which of the following?
- (A) Arrow  (B) BigQuery  (C) Cassandra  (D) Dataform  (E) GCS

### F25 Q27 — Correlated cross join unnest
```
x , y , z
5 , [6, 7], [8, 9]
10, [11] , [12, 13]
```
- (A) 2  (B) 6  (C) 7  (D) 8  (E) 9

### F24 Q1 — Capacity billing free
What is something that capacity billing gives BigQuery users for free?
- (A) CPU  (B) memory  (C) Colossus I/O  (D) Colossus Storage

### F24 Q5 — Capacitor format access
If you want to run BigQuery over data in the Capacitor format, how should you add tables to your dataset?
- (A) load job  (B) external table

### F24 Q26 — Centroid output volume
You run `SELECT FUNC(geom) FROM geotable` in BigQuery. Which FUNC will generally result in more output rows?
- (A) ST_CENTROID  (B) ST_CENTROID_AGG  (C) tie

### F24 Q27 — Correlated cross join unnest
```
x, y, z
1, [2, 3], [4, 5]
6, [7], [8, 9, 10]
```
- (A) 0  (B) 2  (C) 4  (D) 7  (E) 15

### Sp24 Q23 — ML coefficients
Which of the following will enable you to determine the coefficients used to multiply features?
- (A) ML.PREDICT  (B) ML.OPTIONS  (C) ML.EVALUATE  (D) ML.WEIGHTS

### Sp24 Q24 — Not column-oriented
Which of the following is not a column-oriented format?
- (A) Capacitor  (B) CSV  (C) ColumnIO  (D) Parquet

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

### Sp24 Q27 — Lat/long to geo
In BigQuery, which function converts floating-point longitude/latitude into geographic data?
- (A) ST_LATLONG  (B) ST_GEOGPOINT  (C) ST_CENTROID  (D) ST_MAKEPOINT

### Sp26-E2 Q2 — Layout orientation (storage)
Is the below data layout "column oriented" or "row oriented"?
Table:
```
6, 4, 3
5, 1, 2
```
Disk layout: `6, 4, 3, 5, 1, 2`
- (A) column oriented  (B) row oriented

### Sp26-E2 Q6 — OLTP layout
An OLTP database typically uses what orientation for its on-disk layout?
- (A) row-oriented  (B) column-oriented

### Sp26-E2 Q17 — Arrow representation (drawing)
Consider the Python list of strings: `["AB", "C", "", "DEF"]`. Illustrate how Arrow would represent this in a cache-friendly way:
1. Write letters where they appear in the value buffer (single byte per letter).
2. Write values in the offset buffer to support efficient indexing over the strings.
3. Fill in 1s or 0s in the validity bitmap (rightmost bits = smallest indexes).

Empty boxes are ignored during grading.

---

## Other Cloud

### S25 Q8 — Free tiers
How do "free tiers" usually work for cloud services?
- (A) you are not charged for initial operations up to some limit
- (B) you are not charged for additional operations after exceeding some limit

### S25 Q12 — EC2 service kind
What kind of service is EC2?
- (A) IaaS  (B) PaaS

### S25 Q15 — GFLOPS calc
You have 6 billion floating point operations to do on a device capable of 3 GFLOPS. How many seconds will it take?
- (A) 0.0005  (B) 0.5  (C) 1  (D) 2.0  (E) 500.0

### F25 Q8 — Spot vs on-demand
You need to run a once-a-day batch job that can wait if the VM is temporarily unavailable, and it is not critical if it gets interrupted. Which type of VM instance is most suitable?
- (A) on-demand instances  (B) spot instances

### F25 Q12 — Powered-off costs
When you power off a cloud VM, what do you usually still pay for while it is off?
- (A) memory only  (B) CPU and memory  (C) CPU only  (D) disk capacity

### F25 Q23 — SSD challenge
Which I/O pattern is most challenging for SSDs?
- (A) random reads  (B) random writes  (C) sequential reads  (D) sequential writes

### F25 Q28 — Network I/O cost
What usually costs more for cloud network I/O?
- (A) ingress  (B) egress

### F24 Q8 — Kubernetes vs Compose
What is something that Kubernetes does that Compose does not do?
- (A) bin packing
- (B) use cgroups to isolate performance
- (C) deploy multiple replicas from the same Docker image

### F24 Q12 — VM cost split
What generally costs more when deploying on a cloud? Option 1: one VM for 100 hours. Option 2: 100 VMs for 1 hour.
- (A) option 1 costs more  (B) option 2 costs more  (C) the costs are similar

### F24 Q28 — Cloud organization
What best describes cloud organization?
- (A) zones contain regions  (B) regions contain zones  (C) clusters contain regions

### Sp26-E1 Q2 — GFLOPS calc (Version A)
You have 4 billion floating point operations to do on a device capable of 8 GFLOPS. How many seconds will it take?
- (A) 0.002  (B) 0.5  (C) 1  (D) 2.0  (E) 2000.0

> Variations across versions: B uses 4 million ops / 2 GFLOPS, C uses 8 million / 4 GFLOPS, D uses 8 million / 16 GFLOPS.

### Sp26-E2 Q8 — SSD garbage collection
What access pattern creates the most garbage collection work for SSDs?
- (A) random reads  (B) sequential reads  (C) random writes  (D) sequential writes

---

## Minor systems (MySQL / MapReduce / HBase)

### F24 Q14 — HBase RegionServers
How does HBase assign data to RegionServers? Assume we are using 3x replication.
- (A) each column will belong to one RegionServer
- (B) each column will belong to three RegionServers
- (C) each region will belong to one RegionServer
- (D) each region will belong to three RegionServers

### Sp26-E3 Q7 — MapReduce: how many map() calls
For a MapReduce job, you have 1000 input key/value pairs, 138 intermediate key/value pairs, and 43 output key/value pairs. Among the 1000 inputs, there are 4 unique keys. How many times will map(...) be called?
- (A) 4  (B) 43  (C) 138  (D) 1000

> Note: Direct HBase / MySQL / MapReduce questions are sparse in the past Exam 3 papers since those topics were typically tested earlier in the semester. The Sp26-E3 MapReduce question above is the only fresh one. Expect new final-exam questions covering: HBase region assignment and master node, full MapReduce phases (map / shuffle / reduce) and intermediate I/O behavior, and MySQL replication / consistency tradeoffs vs Cassandra.

---

## Review (Caching, Concurrency, Linux, Docker, Networking, SQL, Idempotency, Memory)

### Caching

#### S25 Q23 — FIFO size 4
How many hits are there for a FIFO cache of size 4 for the workload `1, 2, 3, 1, 4, 2, 1, 7`?
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

#### F25 Q9 — LRU size 3
How many cache hits for `D, D, A, D, B, A, A, C` with LRU eviction and cache size 3?
- (A) 4  (B) 5  (C) 6  (D) 7  (E) 8

#### F24 Q15 — FIFO size 3
How many hits for FIFO cache size 3 with workload `6, 4, 2, 3, 4, 3, 6, 6`?
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

#### Sp24 Q1 — LRU hit rate
LRU cache size 4. Hit rate for `P, Q, P, R, S, Q, T, P` (cache empty initially).
- (A) 0  (B) 0.25  (C) 0.375  (D) 0.75  (E) 1

#### Sp24 Q34 — FIFO hit rate
FIFO cache size 4. Hit rate for `P, Q, P, R, S, Q, T, R` (cache empty initially).
- (A) 0  (B) 0.25  (C) 0.375  (D) 0.75  (E) 1

#### Sp26-E2 Q4 — FIFO size 3 (count)
How many hits will there be? FIFO cache, capacity=3. Workload: `1, 2, 3, 1, 4, 1`
- (A) 0  (B) 1  (C) 2  (D) 3  (E) 4

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

#### F24 Q6 — Two threads with lock
X starts at 5, two threads run concurrently with a global lock. What are the possible final values?
```python
# thread 1
with lock: X *= 2
# thread 2
with lock: X += 1
```
- (A) only 11  (B) only 12  (C) 11 or 12  (D) 6, 10, 11, or 12  (E) 5, 6, 10, 11, or 12

#### Sp24 Q19 — Critical section term
A portion of code we don't want to be interrupted by another thread is called a ?
- (A) context switch  (B) critical section  (C) lock  (D) collision

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

#### Sp26-E3 Q2 — Lock and context switching (T/F)
True/False: when a thread is holding a lock during a critical section, the scheduler WILL NEVER context switch to another thread in the same process.
- (A) True  (B) False

#### Sp26-E3 Q16 — Per-thread storage (fill-in)
Each thread has its own ____________ for storing local variables.

### Idempotency

#### F25 Q17 — Indexed assignment
Is `do_it` idempotent?
```python
data = [3, 8, 2]
def do_it(val, d):
    data[d] = val
```
- (A) Yes  (B) No

#### F24 Q10 — Squaring
Is the following function idempotent?
```python
def set_square():
    global x
    x = x ** 2
```
- (A) Yes  (B) No

#### Sp24 Q4 — Inverse
Idempotent?
```python
x = 10
def inverse_x():
    global x
    x = 1 / x
```
- (A) yes  (B) no

### Linux / Shell

#### S25 Q9 — Documentation command
What Linux command provides documentation about how to use a given program?
- (A) wget  (B) which  (C) cat  (D) du  (E) man

#### F25 Q15 — Redirect stdout only
How do you redirect ONLY the standard output from program X to file Y?
- (A) X > Y  (B) X -> Y  (C) X | Y  (D) X & Y  (E) X &> Y

#### F24 Q23 — Port to process
What Linux tool can help you see what process is using a port?
- (A) ls  (B) ns  (C) os  (D) ps  (E) ss

#### Sp24 Q35 — Pipe to grep
Search for "UN" inside output.txt. What replaces ???
```
cat output.txt ??? grep "UN"
```
- (A) &  (B) >  (C) &>  (D) >>  (E) |

#### Sp26-E1 (V-A,B) Q11 / (V-C,D) Q12 — List files (fill-in)
The ____________ command lists the files and directories in a location.

#### Sp26-E1 (V-A,B) Q14 — Run as root (fill-in)
To run a command with root privileges, prefix it with ____________.

#### Sp26-E1 (V-A,B) Q16 / (V-C,D) Q13 — Word count (fill-in)
The ____________ program counts lines, words, and characters in its stdin.

#### Sp26-E1 (V-C,D) Q12 — Permissions (fill-in)
File access permissions in Linux are changed using the ____________ command.

#### Sp26-E1 (V-C,D) Q13 / Q9 — Download file (fill-in)
The w____________ command downloads a file from a URL, saving it locally.

#### Sp26-E1 (V-C,D) Q16 — Current directory (fill-in)
Running ____________ displays the full path of your current working directory.

#### Sp26-E2 Q14 — Unbuffered Python (fill-in)
The -u flag in `python3 -u server.py` forces stdout and stderr to be ____________.

#### Sp26-E3 Q14 — Pattern filter (fill-in)
The ____________ program filters lines of input, keeping only those that match a pattern.

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

### Docker / Containers

#### S25 Q18 — Mount flag
In Docker, if you want a file/directory location on the host machine to be visible within a container, what flag should you pass to run?
- (A) -d  (B) -f  (C) -p  (D) -v

#### S25 Q30 — SSH tunnel + Docker port (Spring 2025 chain)
Steps:
1. Dockerfile launches Jupyter on port 2856
2. SSH tunnel: `-L localhost:4702:localhost:3973`
3. `docker run ... -p ????:2856`
4. Browser: `http://localhost:4702/`

What should ???? be in step 3?
- (A) 8888  (B) 3973  (C) 5000  (D) 2856  (E) 4702

#### F25 Q6 — `docker ps`
What does `docker ps` show?
- (A) what images Docker has locally
- (B) what containers are running
- (C) what processes are running within a container

#### F25 Q18 — Memory limit
A container launched with `-m 2g` reads `big.file` (3 GB) on a VM with 4 GB free RAM. No swap. Will memory constraints prevent successful execution?
- (A) yes  (B) no

#### F24 Q25 — SSH tunnel + Docker port (Fall 2024 chain)
Steps:
1. Dockerfile launches Jupyter on port 2241
2. `docker run ... -p 3752:2241`
3. SSH tunnel: `-L localhost:4432:localhost:3752`
4. Browser: `http://localhost:????/`

What should ???? be in step 4?
- (A) 2241  (B) 5000  (C) 8888  (D) 4432  (E) 3752

#### F24 Q30 — Detached mode output
A Docker container `myapp` is running in detached mode. How can you see what the process started by CMD is printing?
- (A) `docker ps myapp`
- (B) `docker logs myapp`
- (C) `docker exec -it myapp`
- (D) `docker exec myapp stdout`

#### Sp24 Q12 — File classification
What is the following an example of?
```
FROM ubuntu:23.10
RUN apt-get update && apt-get install -y unzip python3 python3-pip
RUN pip3 install pandas===2.1.0 --break-system-packages
```
- (A) yml file  (B) protocol buffer  (C) Dockerfile  (D) nodetool

#### Sp26-E1 Q1 — Containers vs VMs (cross-OS) [V-A, V-B]
If you need to sandbox processes, some of which run on Linux and others on Windows, which is better?
- (A) containers  (B) virtual machines

#### Sp26-E1 Q1 — Containers vs VMs (memory) [V-C, V-D]
If you need to sandbox processes and your main concern is minimizing memory usage, which is better?
- (A) containers  (B) virtual machines

#### Sp26-E1 Q4 / Sp26-E3 Q1 — Docker port chain (Spring 2026)
Step 1: Dockerfile launches Jupyter on port 2446
Step 2: SSH tunnel `-L localhost:4369:localhost:3035`
Step 3: `docker run ... -p ????:2446`
Step 4: Browser `http://localhost:4369/`

What should ???? be?
- (A) 5000  (B) 4369  (C) 2446  (D) 3035  (E) 8888

> Variations: V-B uses 2505/4290/3254, V-C uses 2003/4747/3572, V-D uses 2550/4211/3488. The principle is the same: ???? is the host port that should match the SSH tunnel's right-hand inner port.

#### Sp26-E1 Q7 — Build-time directive [V-A, V-B]
In a Dockerfile, how could you specify that "apt update" should execute during build?
- (A) EXEC apt update  (B) DO apt update  (C) CMD apt update  (D) RUN apt update

#### Sp26-E1 Q7 — Default startup program [V-C, V-D]
In a Dockerfile, how do you specify the program that should launch (by default) when a container starts?
- (A) EXEC  (B) RUN  (C) CMD  (D) DO

#### Sp26-E1 Q9 — Delete image (fill-in) [V-A, V-B] / Q10 [V-A, V-B]
To delete a Docker image X, run `docker ____________ X`.

#### Sp26-E1 Q9 — Running containers (fill-in) [V-C, V-D] / Q14 [V-D]
To see currently running Docker containers, run `docker ____________`.

#### Sp26-E2 Q3 — Rebuild speed
If you build an image from a Dockerfile, change the Dockerfile, then build again, what change will usually result in a SLOWER rebuild?
- (A) changing a line near the beginning
- (B) changing a line near the end

#### Sp26-E2 Q11 — Run a command inside a container (fill-in)
To run a command inside a running container, use `docker ____________`.

### Networking / Memory / Misc

#### S25 Q7 — Truly unique ID
You don't want any chance of different computers using the same library to produce the same ID. What information about the machine would be most helpful?
- (A) IP address  (B) MAC address  (C) port number of current process  (D) free disk space

#### S25 Q25 — RAM granularity
For RAM, what is the finest granularity at which every piece of memory has its own address?
- (A) bit  (B) byte  (C) cacheline  (D) page  (E) block

#### F25 Q25 — URL component
You have a URL `someprotocol://someaddr:someport/someresource`. Which part determines the specific running process on a machine that will receive the request?
- (A) someprotocol  (B) someaddr  (C) someport  (D) someresource

#### F24 Q7 — gRPC serialization
What does gRPC use to serialize messages?
- (A) ColumnIO  (B) JSON  (C) Parquet  (D) Protocol Buffers

#### Sp24 Q15 — Encoding identification
A column has values: `apple, apple, apple, banana, banana, orange, orange`. Represented as `{3: 1, 2: 2, 2: 3}` and `{"apple": 1, "banana": 2, "orange": 3}`. What technique(s)?
- (A) only dictionary encoding
- (B) only run-length encoding
- (C) both dictionary encoding and run-length encoding

#### Sp24 Q22 — PyTorch tensor bytes
A 20x5 PyTorch tensor stores double precision floats. How many bytes (excluding Python object overhead)?
- (A) 64  (B) 200  (C) 400  (D) 800  (E) 51200

#### Sp26-E1 Q3 — URL HTTP layer
You have a URL `http://someaddr:someport/someresource`. Which part is processed by the HTTP layer of the network stack, specifically?
- (A) someaddr  (B) someport  (C) someresource

#### Sp26-E1 Q8 — Latency measurement [V-A, V-B]
Which one is an example of a latency measurement?
- (A) 6 MB  (B) 2 seconds  (C) 3 MB/s

#### Sp26-E1 Q8 — Throughput measurement [V-C, V-D]
Which one is an example of a throughput measurement?
- (A) 6 MB  (B) 2 seconds  (C) 3 MB/s

#### Sp26-E1 Q12 — IPv4 size (fill-in)
An IPv4 address contains ____________ bits.

#### Sp26-E1 Q13 — Address translation (fill-in)
A N____________ is often used to forward from public to private IP addresses.

#### Sp26-E1 Q15 — Transport protocol (fill-in)
Name either of the transport protocols we covered in class that provides port numbers: ____________.

#### Sp26-E1 (V-C, V-D) Q15 — Loopback (fill-in)
The loopback device uses IP address ____________.

#### Sp26-E2 Q7 — Best RAM throughput
What access pattern typically provides the best throughput when reading integers from RAM?
- (A) random  (B) sequential

#### Sp26-E2 Q9 — Virtual address space (fill-in)
The three most important things stored in a virtual address space are code, stack, and the ____________.

#### Sp26-E2 Q12 — Fastest cache level (fill-in)
The fastest CPU cache level is L____________ (assume a 3-level cache).

#### Sp26-E2 Q15 — Cache miss term (fill-in)
When a lookup to a cache CANNOT find a value, the lookup is called a ____________.

#### Sp26-E2 Q16 — Python's GC lock (fill-in)
The lock Python uses internally to protect counters related to garbage collection is called the ____________.

#### Sp26-E3 Q10 — Replication term (fill-in)
Having multiple copies of data on different machines is called ____________.

#### Sp26-E3 Q11 — Random access (fill-in)
Jumping around and reading from many different locations in a file is called ____________ access.

### gRPC / Protocol Buffers (new section, mostly Sp26)

#### Sp26-E1 Q5 — Server-side generated class [V-A, V-B]
With gRPC, what generated class does a server override?
- (A) a servicer class  (B) a stub class

#### Sp26-E1 Q5 — Client-side generated code [V-C, V-D]
With gRPC, what generated code does a client use to make calls?
- (A) servicer code  (B) stub code

#### Sp26-E1 Q6 — Older client missing field [V-A, V-B]
An older gRPC client does NOT send a field the gRPC server was expecting. What happens?
- (A) the server crashes
- (B) the server ignores the request
- (C) the server returns an error to the client
- (D) the server uses a default value for missing field

#### Sp26-E1 Q6 — Newer client extra field [V-C, V-D]
A newer gRPC client sends a field the gRPC server was NOT expecting. What happens?
- (A) the server crashes
- (B) the server ignores the field
- (C) the server returns an error to the client
- (D) the server uses a default value for missing field

#### Sp26-E1 Q10 — .proto: messages and ____ (fill-in) [V-A, V-B]
The two main constructs defined in a .proto file are messages and ____________.

#### Sp26-E1 (V-C, V-D) Q14 — .proto: services and ____ (fill-in)
The two main constructs defined in a .proto file are services and ____________.

#### Sp26-E1 (V-C, V-D) Q10 / Q11 — gRPC serialization (fill-in)
gRPC uses ____________ for serialization/deserialization.

#### Sp26-E2 Q5 — NOT a gRPC feature
What is NOT a feature built into gRPC?
- (A) for a failed call, it will automatically retry to a different server
- (B) for small integers, it will use variable length encoding to save space
- (C) it allows clients and servers to be written in different programming languages
- (D) it allows clients and servers to have different versions of a protocol in some cases

#### F24 Q7 (also relevant here) — gRPC serialization
What does gRPC use to serialize messages?
- (A) ColumnIO  (B) JSON  (C) Parquet  (D) Protocol Buffers

### Databases / ACID (new section, Sp26)

#### Sp26-E2 Q10 — ETL (fill-in)
ETL stands for ____________.

#### Sp26-E2 Q13 — ACID acronym (fill-in)
ACID stands for ____________, Consistency, Isolation, and Durability.

#### Sp26-E3 Q3 — Durability scenario
You don't want your ACID database to lose committed data if your server suddenly loses power. What guarantee is this?
- (A) Atomicity  (B) Consistency  (C) Isolation  (D) Durability

### SQL

#### F24 Q18 — Filter before group
If you want to filter rows before they are grouped, what do you use in SQL?
- (A) HAVING  (B) LIMIT  (C) WHERE

#### Sp24 Q6 — Projection
Which SQL clause enables us to perform projection?
- (A) SELECT  (B) FROM  (C) WHERE  (D) GROUP BY  (E) ORDER BY

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
