<!-- hbase-cassandra-kafka-study-guide.md -->

# CS544 Study Guide: HBase, Cassandra, Kafka

This guide covers **HBase + Cassandra + Kafka**. It is meant to be enough to study from directly.

---

# 0. What these systems are

| System | Main idea | Most important exam distinction |
|---|---|---|
| **HBase** | BigTable-style distributed database built on HDFS | Reliability comes from **HDFS replication + RegionServer failover** |
| **Cassandra** | Dynamo + BigTable-inspired distributed database | **Leaderless**, highly available, consistent hashing, quorum reads/writes |
| **Kafka** | Distributed log / streaming system | **Leader/follower replication**, topics, partitions, consumer groups |

Cassandra takes its data model/storage layout ideas from BigTable/HBase and its partitioning/replication ideas from Dynamo. Its goal is high availability even when machines/networks/data centers fail.

---

# 1. HBase

## 1.1 HBase data model

HBase uses **versioned sparse tables**.

Mental model:

```text
table[row : column : version] = value
```

Example:

```text
table["2:y"] = "apple"
table["2:y:v1"] = "apple"
table["2:y:v2"] = "kiwi"
```

Important properties:
- Sparse table: empty cells do **not** waste space.
- Rows can have very different columns.
- Columns can grow over time.
- Multiple versions of a cell can exist.

---

## 1.2 HBase regions and RegionServers

HBase partitions the row space into **regions**.

```text
row ranges = regions
each region is assigned to one RegionServer at a time
one RegionServer can serve multiple regions
```

Rows are never split across regions, so HBase only supports **single-row transactions**.

Design implication:

```text
If you need atomic operations over a user’s data, put that user’s data in one row,
even if the row has millions of columns.
```

---

## 1.3 HBase reliability

RegionServers store region data in **HDFS files**.

If a RegionServer dies:

```text
1. The RegionServer goes offline.
2. The actual region data is still safe in HDFS because HDFS replicates it, often 3x.
3. HBase hands those regions to healthy RegionServers.
```

Exam shortcut:

```text
HBase reliability = HDFS replication + reassign regions after RegionServer failure
```

Do **not** say HBase itself has Cassandra-style quorum replication. It relies heavily on HDFS.

---

## 1.4 HBase storage layout

Problem:

```text
random disk writes are slow
sequential large writes are fast
```

HBase strategy:

```text
1. Store new writes in memory first.
2. Sort them.
3. Flush them as large HDFS files.
```

This produces multiple HDFS files over time. Reads may need to check multiple files to find the latest value.

---

## 1.5 Tombstones and compaction

Deletes are handled with **tombstones**.

Why?

```text
Old HDFS files are finalized; you cannot easily erase one old key-value entry in place.
So deletion is represented by writing a delete marker.
```

Too many files makes reads slow, so HBase performs **compaction**:

```text
many small files → fewer bigger files
```

Compaction also helps clean up overwritten/deleted versions.

---

# 2. HBase vs Cassandra data model

## 2.1 HBase = wide row

HBase has a **wide row** design.

Advantages:
- Whole row is guaranteed to be on one RegionServer.
- Good for single-row transactions.
- Good when you want lots of flexible columns.

Disadvantages:
- No efficient way to know all possible columns.
- SQL-like query languages are hard.

---

## 2.2 Cassandra = wide partition

Cassandra has a **wide partition** design.

Advantages:
- Finite known columns, so CQL/SQL-like querying is easier.
- Related data can be placed on the same machines.
- Rows inside a partition can be sorted by cluster keys.

Disadvantages:
- Big partitions create storage imbalance.
- Hot partitions create load imbalance.

---

## 2.3 Cassandra primary key pieces

Cassandra primary key has:

```text
partition key column(s) + cluster key column(s)
```

Definitions:

| Piece | Meaning |
|---|---|
| **Partition key** | Determines which partition the row belongs to; determines machine placement |
| **Cluster key** | Determines sort order within a partition |
| **Static column** | One value per partition key |
| **Regular column** | Normal row-level value |

Primary key uniquely identifies a row. If you insert another row with the same primary key, it overwrites/updates the old row, not creates a new row.

Example:

```sql
PRIMARY KEY ((station_id), date)
```

Here:
- `station_id` = partition key
- `date` = cluster key
- data is grouped by station
- dates are sorted inside each station partition

This makes queries like “get all dates for station 123” fast.

---

## 2.4 Cassandra schema design rule

Cassandra schema design is query-driven.

You choose:
- partition key based on what data you want stored together
- cluster key based on what range/order queries you need
- static columns for values repeated across all rows in a partition

Bad choices:
- too many partitions: queries hit many nodes
- too few partitions: imbalance
- huge partition: storage imbalance
- popular partition: hot partition

---

# 3. Cassandra partitioning

## 3.1 Three partitioning approaches

| Approach | Used by | How it works |
|---|---|---|
| Mapping data structure | HDFS | NameNode stores block → DataNode locations |
| Hash partitioning | Spark shuffle | `partition = hash(key) % partition_count` |
| Consistent hashing | Cassandra/Dynamo | `token = hash(partition key)` and walk token ring |

Cassandra uses **consistent hashing**, not ordinary modulo hash partitioning.

---

## 3.2 Why consistent hashing matters

With normal hash partitioning:

```text
partition = hash(key) % N
```

If `N` changes, many keys move.

With consistent hashing:
- nodes own token ranges
- adding a node usually only moves data from nearby ranges
- better for elastic scaling

Rule from slides:

```text
Hash partitioning: adding Nth machine can move about (N-1)/N of data
Consistent hashing: only data in the new node’s range moves
```

---

## 3.3 Token ring rule

Cassandra maps rows to tokens:

```text
row token = hash(partition key)
```

Each worker has one or more tokens. A token is the **inclusive end** of a range.

To find the responsible node:

```text
sort all node tokens
start at the row token
walk clockwise to the first node token >= row token
if no token is bigger, wrap to the smallest token
```

The “wrap” is why it is called a **token ring**.

---

## 3.4 Token map example

Suppose:

```text
token(n1) = -7
token(n3) = 4
token(n2) = 7
row token = 3
```

Sorted tokens:

```text
-7(n1), 4(n3), 7(n2)
```

Walk clockwise from 3:
- next token is 4
- owner is `n3`

So with replication factor 1:

```text
row belongs to n3
```

If row token were 8:
- no token >= 8
- wrap to smallest token `-7`
- owner is `n1`

---

## 3.5 Vnodes

A physical Cassandra node can have multiple tokens/vnodes.

Example:

```text
token(n1) = [3, 4, -2]
token(n2) = [-5, -3, -1]
token(n3) = [-8]
```

Treat every vnode token separately when walking the ring, but remember that multiple vnodes can belong to the same physical node.

---

# 4. Cassandra replication

## 4.1 Replication factor

Replication factor, or **RF**, means how many copies of a row Cassandra stores.

Example:

```sql
CREATE KEYSPACE X
WITH replication = {
  'class': 'SimpleStrategy',
  'replication_factor': 2
};
```

RF improves durability because data is not lost if one node dies.

Cassandra lets different keyspaces have different replication factors.

---

## 4.2 Finding replicas on the token ring

For RF > 1:

```text
1. Find first responsible node by walking clockwise from row token.
2. Keep walking clockwise.
3. Collect distinct physical nodes until you have RF nodes.
4. Skip vnodes on the same physical machine.
```

Why skip same machine?

```text
If one physical server dies, all its vnodes die together.
Multiple replicas on the same server provide little safety.
```

This is the same **failure domain**.

Cassandra can skip nodes while walking the ring.

---

## 4.3 SimpleStrategy vs NetworkTopologyStrategy

| Strategy | Behavior |
|---|---|
| **SimpleStrategy** | Treats all nodes equally, skips vnodes on same machine, ignores racks/data centers |
| **NetworkTopologyStrategy** | Considers racks and data centers to avoid correlated failures |

CS544 uses **SimpleStrategy**.

---

## 4.4 Correlated failures

Correlated failure means multiple machines fail together.

Examples:
- same server: all vnodes on that server die
- same rack: top-of-rack switch fails
- same rack: rack power fails
- same data center: larger disaster

NetworkTopologyStrategy tries to place replicas across failure domains.

SimpleStrategy mostly ignores rack/data-center placement.

---

# 5. Cassandra reads/writes and quorums

## 5.1 Coordinator

In Cassandra, the client talks to a **coordinator** node.

The coordinator:
- receives client request
- sends writes/reads to replicas
- waits for enough responses
- responds to client

Cassandra is **leaderless**. There is no special leader replica for a row like Kafka has.

---

## 5.2 Write quorum W

For a write with replication factor RF:

```text
coordinator attempts to write to all RF replicas
client ACK is sent after W replicas ACK
```

Example:

```text
RF = 3
W = 2
```

Coordinator sends write to 3 replicas and can respond to client once 2 replicas acknowledge.

---

## 5.3 Read quorum R

For a read:

```text
coordinator reads from R replicas
```

If `R = 1`, reads are fast and available, but you might read stale data.

If `R > 1`, Cassandra can compare replicas.

To avoid reading full duplicate data from every replica, it can read:
- full data from one replica
- checksum/digest from others

If checksums mismatch, Cassandra knows replicas disagree.

---

## 5.4 Strong consistency rule

The key formula:

```text
R + W > RF
```

If this holds, reads and writes must overlap on at least one replica.

Meaning:

```text
If a write was acknowledged, a later read is guaranteed to contact at least one replica that saw the write.
```

Examples:

```text
RF = 3, W = 2, R = 2
2 + 2 > 3 → strong
```

```text
RF = 10, R = 5
Need W = 6 because 5 + 6 > 10
```

```text
R = 5, W = 2
Need RF < 7 because 5 + 2 > RF
```

This exact formula appears repeatedly in the question bank.

---

## 5.5 Quorum tradeoffs

Assume RF = 3.

| Setting | Result |
|---|---|
| `W=3, R=1` | Reads fast/available, writes fail if any replica unavailable |
| `W=1, R=3` | Writes fast/available, reads fail if any replica unavailable; durability risk |
| `W=2, R=2` | Balanced |
| `W=1, R=1` | Fastest/most available, but not strongly consistent |

General:

```text
Higher W = safer writes, lower write availability
Higher R = safer reads, lower read availability
Lower R/W = faster and more available, but weaker consistency
```

---

## 5.6 Cassandra availability priority

Cassandra prioritizes **availability**.

It is generally considered AP-style in CAP terms.

It keeps accepting operations during failures, with eventual consistency / tunable consistency instead of always requiring strict consistency.

---

# 6. Cassandra conflict resolution

## 6.1 Why conflicts happen

Because Cassandra is leaderless:
- multiple replicas may temporarily disagree
- writes can reach some replicas but not others
- network partitions can allow divergent versions

---

## 6.2 Eventual consistency

Eventual consistency means:

```text
replicas may disagree temporarily,
but if no new writes happen and repair mechanisms run,
they should eventually converge.
```

---

## 6.3 Read repair / checksum idea

On reads:
- Cassandra can compare checksums/digests from replicas.
- If checksums differ, it detects inconsistency.
- It can repair stale replicas.

---

## 6.4 Last-write-wins

Common conflict resolution:

```text
newer timestamp wins
```

Exam-level intuition:

```text
Cassandra may accept conflicting writes for availability, then resolve later.
```

---

# 7. Cassandra exam patterns

## Pattern 1: Primary key uniqueness

If table has:

```text
partition key = X
cluster key = Y
regular column = Z
```

Then uniqueness is based on:

```text
(X, Y)
```

Repeated inserts with same `(X, Y)` overwrite the old row.

Example:

```text
(2,2,4)
(3,2,2)
(3,2,4)
(2,1,5)
(2,1,2)
```

Distinct primary keys:
- `(2,2)`
- `(3,2)`
- `(2,1)`

Final row count = **3**.

---

## Pattern 2: Token ring

Steps:
1. Determine partition key.
2. Hash only the partition key.
3. Sort token map.
4. Walk clockwise.
5. For RF > 1, keep walking to distinct physical nodes.
6. Wrap around if needed.

Important trap:

```text
PRIMARY KEY ((x), y)
```

means:
- partition key = `x`
- cluster key = `y`

Use `hash(x)`, not `hash(x,y)`.

---

## Pattern 3: Quorum math

Always use:

```text
R + W > RF
```

If asked smallest W:

```text
W > RF - R
```

If asked tightest bound on RF:

```text
RF < R + W
```

---

## Pattern 4: CAP

If asked what Cassandra prioritizes:

```text
availability
```

---

## Pattern 5: Gossip

Cassandra uses **gossip** to share cluster membership/state because it has no centralized boss.

---

# 8. Kafka basics

## 8.1 RPC vs streaming

RPC is synchronous:
- both parties interact at the same time
- example: function call over network

Streaming is asynchronous:
- producer sends now
- consumer can receive later
- messages persist in broker

---

## 8.2 Why Kafka exists

Batch ETL problem:
- many OLTP databases
- many downstream systems
- custom ETL scripts between every pair become messy
- data freshness is bad because cron/batch jobs run periodically

Kafka solution:

```text
central unified log
many producers write changes
many consumers read changes
ETL can happen in real time
```

---

# 9. Kafka topics, producers, consumers, brokers

## 9.1 Definitions

| Term | Meaning |
|---|---|
| **Topic** | Named stream/log, like `clicks` or `purchases` |
| **Producer** | Application code that sends messages |
| **Consumer** | Application code that reads messages |
| **Broker** | Kafka server that stores topic partitions |
| **Partition** | Ordered append-only log inside a topic |
| **Offset** | Index of a message within a partition |

You usually write producer and consumer code.

You usually **do not** write broker code; brokers are Kafka servers you configure.

---

## 9.2 Message contents

Kafka message has:
- key: optional bytes
- value: required bytes
- other metadata

Common pattern:
- value = serialized structured data
- key = field used for partitioning

Example:

```python
producer.send("topic", value=b"...")
producer.send("topic", key=b"user123", value=b"...")
```

Value can be JSON bytes or protobuf bytes.

---

# 10. Kafka partitions

## 10.1 Partition basics

A topic can have N partitions.

Each partition is like an array/log:

```text
clicks[0]: offset 0, 1, 2, 3, ...
clicks[1]: offset 0, 1, 2, 3, ...
```

Partitions are assigned to brokers.

They allow Kafka to scale because one topic can be spread over many machines.

---

## 10.2 Append/delete behavior

Kafka:
- appends new messages on the right
- deletes old messages from the left based on retention policy
- deleting old messages does **not** change offsets

So offsets are stable positions in a partition log.

---

## 10.3 Selecting partitions

Case 1: no key

```text
producer rotates between partitions
round-robin policy
```

Case 2: key exists

```text
partition = hash(key) % partition_count
same key goes to same partition
```

This only guarantees same partition if:
- same topic
- same key
- partition count has not changed

---

## 10.4 Ordering guarantees

Kafka ordering is **per partition**.

Guarantees:
- messages in the same partition are consumed in written order
- same topic + same key → same partition → ordered relative to each other
- no ordering guarantee across partitions
- no ordering guarantee across topics

Exam shortcut:

```text
same topic + same key = same partition/order guarantee
different topic or different key = no guarantee
```

Question-bank trap:

```text
topic="purple", key="green"
topic="red", key="green"
```

Same key but different topic → no same-partition guarantee.

---

# 11. Kafka consumers and consumer groups

## 11.1 Consumer offsets

Consumers read from partitions using offsets.

`poll()` returns batches:
- from subscribed topics
- from some partitions
- starting at current offsets

A consumer normally reads forward sequentially, but `seek(partition, offset)` can jump backward or forward.

---

## 11.2 Consumer groups

Consumer groups allow independent applications to consume the same topic.

Rules:
- Each consumer group gets its own offsets.
- Each message is consumed once per consumer group.
- Within a group, one partition is assigned to one consumer at a time.
- More consumers in a group increases parallelism up to the number of partitions.

Exam pattern:

```text
4 consumer groups, 5 consumers each, all subscribed to topic T.
A new message is consumed how many times?
```

Answer:

```text
4
```

Why:
- once per group
- not once per consumer

---

## 11.3 Scaling consumers

If topic has 4 partitions:
- 1 consumer in group → one consumer handles all partitions
- 2 consumers → partitions split across them
- 4 consumers → one partition each
- 5 consumers → one consumer idle

Parallelism within one group is limited by number of partitions.

---

# 12. Kafka replication

## 12.1 Leader/follower model

Each Kafka partition has:
- one leader replica
- RF - 1 follower replicas

Producers send only to the **leader**.

Followers fetch from the leader, just like consumers do.

Huge distinction:

```text
Kafka = leader/follower
Cassandra = leaderless
HDFS = pipelined writes
```

---

## 12.2 Replication write path

For Kafka RF = 3:

```text
producer → partition leader
followers fetch from leader
```

Not:
- producer writes to all replicas
- leader pushes to followers
- pipeline leader → follower → follower

Question-bank answer:

```text
client writes to leader, followers later fetch it
```

---

## 12.3 In-sync vs lagging replicas

Followers that are keeping up are called **in-sync replicas** or ISR.

Important nuance:
- “in-sync” is configurable/fuzzy.
- Better mental model: **eligible to become leader**.
- In-sync followers might still be slightly behind, depending on configuration.

---

## 12.4 `min.insync.replicas`

`min.insync.replicas` is the minimum number of in-sync replicas needed to accept strong writes.

Important:
- includes the leader
- must be ≤ replication factor
- if not enough in-sync replicas exist, Kafka rejects writes with `NotEnoughReplicasException`
- backoff gives lagging followers time to catch up

Tradeoff:

```text
increase min.insync.replicas → better durability, worse write availability
decrease min.insync.replicas → better write availability, worse durability
```

---

## 12.5 Leader failure

If leader fails:
1. Partition is temporarily unavailable.
2. Controller broker elects a new leader.
3. New leader must be an in-sync replica by default.
4. Old leader does not automatically become leader again when it recovers.

If there are no eligible in-sync replicas:
- system may stay unavailable
- or do unclean leader election, which can lose committed data

---

# 13. Kafka committed messages

## 13.1 Definition

Kafka message is **committed** when written to **all in-sync replicas**.

This is the main exam rule.

Example:

```text
Leader: A B C D E
Follower 1 lagging: A B
Follower 2 in-sync: A B C D E
Follower 3 in-sync: A B
```

Is E committed?

Answer:

```text
No
```

Why:

```text
E is not on all in-sync replicas.
Follower 3 is in-sync but only has A B.
```

---

## 13.2 Why committed matters

Kafka wants to avoid anomalies.

Write anomaly:
- producer gets ACK
- leader crashes
- consumers never see message

Avoid with:

```python
KafkaProducer(..., acks="all")
```

`acks` options:
- `acks=0`: do not wait for ACK
- `acks=1`: ACK after leader writes locally
- `acks="all"`: ACK after data is committed

Read anomaly:
- consumer reads uncommitted message
- leader crashes
- new leader does not have that message
- message disappears/changes later

Avoided because Kafka leaders do **not** return uncommitted data to consumers.

---

# 14. Kafka delivery semantics

## 14.1 At-most-once

Producer example:

```python
producer = KafkaProducer(..., acks=1)
producer.send("my-topic", b"some-value")
```

With weak ACKs and no retry:

```text
successful write means message was recorded at most once
could be once, could be zero if leader crashes at bad time
```

---

## 14.2 At-least-once

Producer example:

```python
producer = KafkaProducer(..., acks="all", retries=10)
producer.send("my-topic", b"some-value")
```

With strong ACKs and retry:
- if no ACK, producer retries
- message might already have been written
- retry can produce duplicate

So:

```text
strong ACK + retry = at-least-once
```

---

## 14.3 Idempotence

An operation is idempotent if doing it multiple times has the same effect as doing it once.

Idempotent:

```python
set_x(123)
set_x(123)
set_x(123)
```

Not idempotent:

```python
inc_y(3)
inc_y(3)
inc_y(3)
```

You can make non-idempotent operations safe by using unique operation IDs:

```python
completed_ops = set()

def inc_y(value, operation_id):
    if operation_id not in completed_ops:
        y += value
        completed_ops.add(operation_id)
```

---

## 14.4 Exactly-once producer side

Settings:

```python
acks="all"
retries=N
enable.idempotence=True
```

With idempotence, producer generates unique operation IDs and brokers suppress duplicates.

Java supports this; `kafka-python` may not, so consumers may need to suppress duplicates manually.

---

## 14.5 Consumer-side exactly-once issue

Consumer has two things to do:
1. process messages / write output
2. commit offsets

If it crashes between those, you can get:
- duplicate processing
- skipped messages

Auto-commit is only approximate.

Manual commit is better, but still can fail if crash happens at the wrong time.

Best approach:

```text
atomically save both:
(a) output changes caused by messages
(b) offsets showing those messages were processed
```

Exactly-once needs producer, broker, and consumer all working correctly:
- producer: strong ACKs, retry, idempotence
- broker: replication, failover, ACKs, duplicate suppression, hide uncommitted data
- consumer: careful offset handling / duplicate suppression

---

# 15. Kafka exam patterns

## Pattern 1: Consumer groups

Question:

```text
4 groups, each with 5 consumers. All subscribed to topic T.
How many times does one new message get consumed?
```

Answer:

```text
4
```

Reason:

```text
once per consumer group
not once per consumer
```

---

## Pattern 2: Same partition guarantee

Same partition only if:

```text
same topic + same key
```

No guarantee if:
- same key but different topic
- same topic but different key
- no keys
- partition count changes

---

## Pattern 3: Kafka write path

Question asks how RF=3 data gets written.

Answer:

```text
producer writes to leader; followers fetch from leader
```

---

## Pattern 4: Leader election

Eligible new leader:

```text
in-sync replicas only
```

Lagging replica is not eligible by default, even if it has more data than another in-sync replica.

---

## Pattern 5: Committed message

Rule:

```text
committed = present on all in-sync replicas
```

Ignore lagging replicas for commit check. But every in-sync replica must have it.

---

## Pattern 6: `min.insync.replicas`

Increase it:
- improves durability
- reduces write availability

Decrease it:
- improves write availability
- reduces durability

---

## Pattern 7: Broker code

Question asks which does not need custom code:

```text
brokers
```

You write producers/consumers; brokers are configured servers.

---

# 16. Fast comparison table

| Concept | HBase | Cassandra | Kafka |
|---|---|---|---|
| Main abstraction | Sparse versioned table | Wide partition table | Distributed log |
| Storage base | HDFS | Own distributed storage | Broker log partitions |
| Replication | HDFS replication | Peer-to-peer replicas | Leader/follower replicas |
| Failure handling | Reassign regions | Keep available via quorums | Elect new partition leader |
| Central boss? | HBase has RegionServers + coordination, HDFS NameNode under it | No centralized boss | Controller broker manages leader election |
| Data placement | Row ranges/regions | Consistent hashing token ring | Topic partitions on brokers |
| Writes | Memory then sorted flush | Coordinator writes to replicas | Producer writes to leader |
| Reads | May check multiple HDFS files | Coordinator reads R replicas | Consumers read committed log |
| Consistency model | Single-row transactions | Tunable/eventual | Committed-message log semantics |
| Exam identity | BigTable on HDFS | Dynamo + BigTable | Unified log / streaming |

---

# 17. Must-memorize sheet

```text
HBase = BigTable-style, versioned sparse table
HBase regions = row ranges
One region assigned to one RegionServer at a time
Rows never split across regions
HBase only supports single-row transactions
HBase reliability = HDFS replication + RegionServer failover
HBase writes = buffer in memory, sort, flush to HDFS
HBase deletes = tombstones
Compaction = many small files → fewer big files

Cassandra inspired by BigTable/HBase + Dynamo
Cassandra prioritizes availability
Cassandra = leaderless
Cassandra cluster = ring
Partition key determines machine placement
Cluster key determines sort order within partition
Static column = one value per partition
Primary key = partition key + cluster key
Same primary key insert overwrites old row
Consistent hashing = hash partition key to token, walk ring
Wrapping range = after largest token wraps to smallest token
RF = number of replicas
SimpleStrategy skips same machine but ignores racks/data centers
NetworkTopologyStrategy considers racks/data centers
Coordinator sends reads/writes to replicas
W = write acknowledgments needed
R = read responses needed
Strong read-after-write rule: R + W > RF
Checksum/digest avoids reading full duplicate data
Gossip shares cluster state without central boss

Kafka = distributed log / streaming system
Topic = named stream
Partition = ordered append-only log
Offset = index in partition
Producer writes messages
Consumer reads messages
Broker stores partitions
No key = round robin partitioning
Key = hash(key) % partition_count
Same topic + same key = same partition/order guarantee
Ordering only within a partition
Consumer group = independent application
Each message consumed once per group
More consumers than partitions leaves consumers idle
Kafka replication = leader/follower
Producer writes only to leader
Followers fetch from leader
In-sync replica = leadership eligible
Committed = written to all in-sync replicas
acks=0: no ACK
acks=1: leader local ACK
acks="all": committed ACK
min.insync.replicas includes leader
Increase min.insync.replicas = durability up, write availability down
Leader failure → controller elects in-sync follower
Strong ACK + retry = at-least-once
Idempotent producer + strong ACK + retry = producer-side exactly-once
Consumer exactly-once requires careful/atomic offset handling
```

---

# 18. Final self-test

## HBase

1. What happens if a RegionServer dies?  
**Region data is still in replicated HDFS files; regions are handed to healthy RegionServers.**

2. Why does HBase use tombstones?  
**Old HDFS files are immutable/finalized, so deletion is written as a marker.**

3. What does compaction do?  
**Combines many smaller files into fewer bigger files, improving reads and cleaning old versions/deletes.**

4. Why does HBase support only single-row transactions?  
**Rows are never split across regions; whole row is on one RegionServer.**

## Cassandra

5. What determines Cassandra row placement?  
**Hash of the partition key.**

6. What determines sort order inside a partition?  
**Cluster key.**

7. What is the strong consistency formula?  
**R + W > RF.**

8. If RF=10 and R=5, smallest W for strong reads?  
**6.**

9. If R=5 and W=2, tightest RF bound?  
**RF < 7.**

10. What does Cassandra prioritize?  
**Availability.**

11. What do you skip while walking replicas on a token ring?  
**Duplicate vnodes on the same physical node / same failure domain.**

## Kafka

12. With 4 consumer groups and 5 consumers per group, how many times is a message consumed?  
**4.**

13. What guarantees same partition?  
**Same topic and same key, assuming partition count unchanged.**

14. Kafka RF=3 write path?  
**Producer writes to leader; followers fetch from leader.**

15. Who can become leader after failure?  
**In-sync replica.**

16. When is a Kafka message committed?  
**When it is on all in-sync replicas.**

17. What does `acks="all"` mean?  
**Producer gets ACK only after message is committed.**

18. What happens if `min.insync.replicas` increases?  
**Durability improves, write availability decreases.**

19. What Kafka component do you usually not write custom code for?  
**Brokers.**

20. Strong ACK + retry gives what semantics?  
**At-least-once, unless duplicates are suppressed/idempotent.**
