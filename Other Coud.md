<!-- cloud-study-guide.md -->

# CS544 Study Guide: Other Cloud

This guide covers the **Other Cloud** topic for the final. It combines the uploaded cloud slides with the question-bank cloud topics.

The final has about **4 Other Cloud questions**, so this is not the biggest section, but it is easy points if the definitions are clear.

Main things to know:

```text
IaaS vs PaaS vs SaaS
VMs
GCS buckets / object storage
BigQuery standard tables, external tables, views
materialized data vs views
Dataform and DAGs
regions, zones, clusters
spot/preemptible vs on-demand
free tier
network ingress vs egress
SSD I/O and garbage collection
GFLOPS calculations
```

---

# 1. Cloud service models: IaaS, PaaS, SaaS

## 1.1 IaaS

IaaS = Infrastructure as a Service.

You rent low-level infrastructure:

```text
virtual machines
disks
networks
firewalls
IP addresses
```

Example:

```text
GCP Compute Engine VM
AWS EC2
Azure Virtual Machines
```

Mental model:

```text
IaaS = cloud provider gives you a machine; you manage most software yourself
```

You usually manage:
- OS packages
- installed programs
- runtime
- your app
- security updates
- scaling logic unless configured separately

Exam answer:

```text
VMs are IaaS.
```

---

## 1.2 PaaS

PaaS = Platform as a Service.

You use a managed platform instead of managing machines directly.

Examples:

```text
BigQuery
Google App Engine
managed database services
serverless platforms
```

Mental model:

```text
PaaS = provider manages infrastructure; you submit code/queries/config
```

You usually manage:
- application/query logic
- data/schema/config
- permissions/cost

Provider manages:
- machines
- scaling infrastructure
- much of the operations layer

Exam shortcut:

```text
BigQuery is closer to PaaS than IaaS.
```

---

## 1.3 SaaS

SaaS = Software as a Service.

You use a complete application.

Examples:

```text
Gmail
Google Docs
Salesforce
Slack
```

Mental model:

```text
SaaS = finished software product delivered through cloud
```

You usually do not manage:
- servers
- runtime
- infrastructure
- application code

---

## 1.4 IaaS vs PaaS vs SaaS exam table

| Model | You get | You manage | Examples |
|---|---|---|---|
| **IaaS** | raw compute/storage/network | OS, packages, app | VM, EC2, Compute Engine |
| **PaaS** | managed platform | app/query/data logic | BigQuery, App Engine |
| **SaaS** | finished app | usage/config | Gmail, Docs, Slack |

Memorize:

```text
IaaS = rented infrastructure
PaaS = managed platform
SaaS = finished software
```

---

# 2. Virtual Machines

## 2.1 What a VM is

A virtual machine is a rented computer in the cloud.

Mental model:

```text
cloud VM = remote Linux/Windows machine with CPU, RAM, disk, network
```

You can SSH into it and run commands.

The cloud slides include a demo for creating a VM on GCP.

---

## 2.2 What VMs are used for

Use a VM when you need:
- a remote machine to run programs
- a client machine inside the cloud
- custom software installation
- manual control over OS/runtime
- experiments similar to using a physical server

Exam shortcut:

```text
VM = compute resource
bucket = storage resource
BigQuery = managed analytics resource
```

---

## 2.3 VM vs BigQuery

| Concept | VM | BigQuery |
|---|---|---|
| Service model | IaaS | PaaS-ish managed service |
| You manage | OS/software/processes | queries/data/config |
| Provider manages | physical host | query execution infrastructure |
| Main use | run arbitrary programs | analytical SQL queries |

---

# 3. Regions, Zones, and Clusters

## 3.1 Region

A region is a geographic area.

Examples:

```text
us-central1
us-east1
europe-west1
```

Mental model:

```text
region = broad geographic location
```

Why it matters:
- latency
- data residency
- price
- disaster isolation
- availability

---

## 3.2 Zone

A zone is an isolated location inside a region.

Example:

```text
us-central1-a
us-central1-b
us-central1-c
```

Mental model:

```text
zone = one failure-isolated area inside a region
```

If one zone fails, another zone in the same region may still work.

---

## 3.3 Multi-zone deployment

If you care about availability, place resources across zones.

Example:

```text
VM 1 in us-central1-a
VM 2 in us-central1-b
VM 3 in us-central1-c
```

Why:

```text
a single zone failure should not take down everything
```

---

## 3.4 Cluster

A cluster is a group of machines working together.

Examples from the course:
- HDFS cluster
- Spark cluster
- Cassandra cluster
- Kafka broker cluster
- Kubernetes cluster

Mental model:

```text
cluster = many machines coordinated for one system
```

---

## 3.5 Region/zone exam shortcuts

```text
region = geographic area
zone = isolated location within a region
cluster = group of machines
multi-zone = better availability
same-zone = lower latency/possibly cheaper internal communication
cross-region = better disaster tolerance but more latency/cost
```

---

# 4. On-demand vs Spot / Preemptible VMs

## 4.1 On-demand VM

On-demand VM means you request normal cloud capacity.

Properties:
- more reliable availability
- usually more expensive
- provider generally does not randomly take it away just because someone else wants capacity

Mental model:

```text
on-demand VM = normal VM rental
```

---

## 4.2 Spot / Preemptible VM

Spot/preemptible means you use spare cloud capacity at a discount.

Properties:
- cheaper
- can be interrupted/preempted
- good for fault-tolerant batch jobs
- bad for critical long-running services unless designed for interruption

Mental model:

```text
spot/preemptible = cheap but can be taken away
```

---

## 4.3 What workloads fit spot/preemptible?

Good fit:
- batch processing
- stateless workers
- recomputable jobs
- fault-tolerant distributed jobs
- experiments where interruption is acceptable

Bad fit:
- primary database
- critical service needing stable uptime
- anything where losing the VM causes major failure
- single long-running job with no checkpointing

---

## 4.4 Important BigQuery terminology trap

For normal VMs:

```text
on-demand = stable normal capacity
spot/preemptible = spare interruptible capacity
```

For BigQuery billing:

```text
capacity = reserved slots
on-demand = uses excess/spare capacity and charges by I/O
```

So:

```text
VM on-demand and BigQuery on-demand do not mean the same thing.
```

Use this carefully if the question compares VM billing to BigQuery billing.

---

# 5. Free Tier Mechanics

## 5.1 What free tier means

Free tier means the cloud provider gives some limited usage for free.

Examples:
- limited VM hours
- limited storage
- limited network
- limited BigQuery query bytes
- credits for new accounts

Mental model:

```text
free tier = free only up to limits
```

---

## 5.2 Free tier exam traps

Free tier does **not** mean:
- all cloud usage is free
- usage beyond the limit is free
- every resource is included
- egress is always free
- expensive machine types are free

Exam shortcut:

```text
Free tier is quota-limited.
After the free quota, normal billing applies.
```

---

## 5.3 Common surprise costs

Watch for:
- leaving VMs running
- large disks attached to stopped VMs
- public IP addresses
- network egress
- large BigQuery scans
- repeated query/materialized rebuilds
- logs/monitoring storage

---

# 6. Network I/O: Ingress vs Egress

This is one of the specific cloud topics from the question bank.

## 6.1 Ingress

Ingress means data entering the cloud/provider/resource.

Examples:

```text
your laptop → GCS bucket
your laptop → VM
external API → cloud service
```

Exam shortcut:

```text
ingress = data coming in
```

Cloud providers often make ingress free because they want you to bring data into their platform.

Memorize:

```text
ingress is usually free
```

---

## 6.2 Egress

Egress means data leaving the cloud/provider/resource.

Examples:

```text
GCS bucket → your laptop
VM in cloud → internet user
cloud region → another cloud provider
cloud region → your local machine
```

Exam shortcut:

```text
egress = data going out
```

Egress often costs money.

Memorize:

```text
egress usually costs
```

---

## 6.3 Ingress/egress examples

### Example 1

You upload 10 GB from your laptop to GCS.

```text
direction = laptop → cloud
type = ingress
cost pattern = usually free
```

### Example 2

You download 10 GB from GCS to your laptop.

```text
direction = cloud → laptop
type = egress
cost pattern = usually charged
```

### Example 3

A VM sends 100 GB to users over the public internet.

```text
direction = cloud → internet
type = egress
cost pattern = charged
```

### Example 4

An external client sends requests/data to your cloud service.

```text
direction = outside → cloud
type = ingress
cost pattern = usually free
```

---

## 6.4 Cross-zone and cross-region traffic

Traffic between resources may cost more when it crosses boundaries.

General pattern:

```text
same machine/internal local traffic = cheapest
same zone = often cheapest cloud network path
cross-zone = may cost more
cross-region = usually more expensive
internet egress = often charged
```

Exact pricing varies by provider, but exam-level rule:

```text
the farther data moves out of the local cloud boundary, the more likely it costs money
```

---

## 6.5 Ingress/egress memory trick

```text
INgress = INto cloud
Egress = Exit cloud
```

---

# 7. Cloud Storage / GCS Buckets

## 7.1 What a bucket is

A GCS bucket is object storage.

It stores objects/files like:

```text
CSV
Parquet
images
logs
model outputs
raw data
```

Mental model:

```text
bucket ≈ cloud folder for objects/files
```

But technically, object storage is not the same as a normal local filesystem.

---

## 7.2 Object storage vs local filesystem

| Concept | Object storage / GCS | Local filesystem |
|---|---|---|
| Unit | object/blob | file/block |
| Access | API calls | POSIX file operations |
| Good for | durable large objects | local reads/writes |
| Common use | data lake / raw files | OS/application storage |
| Random overwrite | not the main pattern | normal filesystem can modify blocks |

Exam shortcut:

```text
GCS bucket stores files externally from BigQuery.
```

---

## 7.3 GCS and BigQuery

The cloud slides show GCS storing files like:

```text
sightings.parquet
beaches.parquet
```

BigQuery can:
1. copy/load those files into standard BigQuery tables
2. reference those files through external tables

---

# 8. BigQuery table types in the cloud slides

The cloud slides heavily cover these because they connect GCS storage and BigQuery.

The three main concepts:

```text
standard table
external table
view
```

Related concept:

```text
materialized data / materialized table
```

---

# 9. Standard Tables

## 9.1 What a standard table is

A standard BigQuery table stores data inside BigQuery.

In the slides:

```text
GCS files: CSV, Parquet, etc.
BigQuery standard table: Capacitor files in Colossus
```

To create a standard table from GCS data, BigQuery performs a load/copy:

```text
GCS Parquet/CSV files → BigQuery Capacitor storage
```

The slides call this a:

```text
capacitor load
```

Meaning:

```text
copy data over to BigQuery
```

---

## 9.2 Why standard tables are fast

Standard tables use BigQuery’s optimized internal format:

```text
Capacitor
```

Exam shortcut:

```text
standard table = data copied into BigQuery internal storage
standard table usually gives better query performance and lower repeated query cost
```

---

## 9.3 Downside of standard tables

The key downside:

```text
upstream updates are not immediately reflected
```

If the original GCS files change, the loaded BigQuery table does not automatically update unless you reload/rebuild it.

Exam wording:

```text
standard table has better performance, but stale data risk if upstream files changed
```

---

# 10. External Tables

## 10.1 What an external table is

An external table is a BigQuery table object that references data outside BigQuery storage.

In the slides:

```text
external table = reference to GCS files
```

The data stays in GCS as files like:

```text
CSV
Parquet
other external formats
```

BigQuery does not copy it into Capacitor when you create the external table.

---

## 10.2 Why use external tables?

Use external tables when:
- you want to query files in GCS directly
- you do not want to copy/load the data
- you want upstream file changes to be visible more directly
- data is shared by systems outside BigQuery

---

## 10.3 Downside of external tables

External tables are usually slower or more expensive for repeated queries than standard tables because the data is not stored in BigQuery’s optimized Capacitor format.

Exam shortcut:

```text
external table = BigQuery metadata pointing at external GCS files
standard table = data copied into BigQuery Capacitor files
```

---

# 11. Views

## 11.1 What a view is

A view is a saved query with a name.

Example:

```sql
CREATE VIEW beach_animals
AS
SELECT s.*
FROM sightings s
JOIN beaches b ON s.beach_id = b.beach_id
WHERE b.name = "Bernies Beach";
```

Conceptually:

```text
view = named SQL query
```

A view does not store the full query result as a new physical table.

When you query the view, BigQuery runs the underlying query.

---

## 11.2 View advantages

Views are useful because:
- fast to create
- no duplicated stored data
- upstream updates are reflected when the view is queried
- good for logical abstraction

Example:

```text
If sightings or beaches changes, querying the view sees the updated underlying data.
```

---

## 11.3 View disadvantages

Views can be slower or cost more for repeated queries because BigQuery may need to recompute the query every time.

Exam shortcut:

```text
view = fresh with upstream data, but may repeatedly pay recomputation cost
```

---

# 12. Standard Table vs External Table vs View

| Feature | Standard Table | External Table | View |
|---|---|---|---|
| Data stored where? | BigQuery/Colossus as Capacitor | GCS/external files | not stored as result |
| Is data copied into BigQuery? | yes | no | no |
| Query performance | usually best | often worse than standard | depends on underlying query |
| Repeated query cost | usually lower | can be higher | can be higher |
| Upstream updates visible immediately? | no | generally yes | yes |
| Best use | optimized repeated analytics | query external files directly | reusable logical query |

Memorize:

```text
standard table = fastest/repeated-use, but can become stale
external table = references external files
view = saved query, fresh but recomputed
```

---

# 13. Materialized Data

## 13.1 Normal view version

A normal view:

```sql
CREATE VIEW beach_animals
AS
SELECT s.*
FROM sightings s
JOIN beaches b ON s.beach_id = b.beach_id
WHERE b.name = "Bernies Beach";
```

This stores the query definition, not the result.

---

## 13.2 Materialized version using a table

The slides show using a standard table like a materialized view:

```sql
CREATE TABLE beach_animals
AS
SELECT s.*
FROM sightings s
JOIN beaches b ON s.beach_id = b.beach_id
WHERE b.name = "Bernies Beach";
```

Mental model:

```text
normal view = recipe
materialized table = cooked result
```

---

## 13.3 Why materialize?

Materializing improves:
- performance
- repeated query cost

If `beach_animals` is queried often, storing the result as a table avoids rerunning the join/filter every time.

Exam answer:

```text
A standard/materialized table usually gives better performance and lower cost when queried repeatedly.
```

---

## 13.4 Downside of materializing

Materialized data can become stale.

If upstream data changes:

```text
sightings changes
beaches changes
```

then the already-created `beach_animals` table does not automatically reflect those changes unless refreshed/rebuilt.

Exam shortcut:

```text
materialized result = faster but stale unless refreshed
view = fresh but recomputed
```

---

# 14. Update Visibility

## 14.1 Standard/materialized table

If upstream files/tables change:

```text
standard/materialized table does not automatically update
```

So old query results may remain visible.

---

## 14.2 External table

External tables reference files directly.

So if external files are updated, the external table can reflect those updates because it reads from the external source.

---

## 14.3 View

Views run the underlying query when queried.

So views reflect upstream table changes.

Exam comparison:

```text
Tables usually give best performance and lowest cost,
but upstream updates are not immediately reflected.
External tables and views reflect upstream changes more directly.
```

---

# 15. Refreshing Materialized Data

## 15.1 Can we just rerun CREATE TABLE?

The slide asks:

```text
Can we just re-run this query?
```

and answers:

```text
NO
```

Why?

If a table already exists, simply running:

```sql
CREATE TABLE beach_animals AS ...
```

again may fail because the table already exists.

You usually need:
- replace the table
- delete and recreate
- run a managed pipeline
- use dependency tracking
- schedule refreshes

Exam concept:

```text
Refreshing materialized data requires managing dependencies and rerunning operations in the right order.
```

Do not assume materialized data magically stays updated.

---

# 16. Dependency DAGs

## 16.1 What a DAG is

DAG = Directed Acyclic Graph.

```text
Directed = arrows show dependency direction
Acyclic = no cycles/loops
Graph = nodes connected by edges
```

In data pipelines:
- nodes = data resources or transformation steps
- edges = dependencies

Example:

```text
sightings table ─┐
                 ├─> beach_animals table
beaches table ───┘
```

This means:

```text
beach_animals depends on sightings and beaches
```

---

## 16.2 Why DAGs matter

If upstream data changes, we need to know:

```text
what downstream data should be refreshed?
in what order?
```

For large pipelines:

```text
100s of nodes
many dependencies
```

manual tracking becomes hard.

Exam shortcut:

```text
DAG = dependency graph for data pipeline refresh order
```

If asked why DAGs matter:

```text
They tell the system what depends on what, so updates can be recomputed in the correct order.
```

---

# 17. Dataform

## 17.1 What Dataform is

Dataform is a tool for managing SQL pipelines/DAGs for BigQuery.

Main role:

```text
Dataform helps define and run SQL-based data pipelines that populate BigQuery datasets.
```

---

## 17.2 Key Dataform features

The slide lists these key features:

```text
version control on pipeline/DAG code, integrates with Git
adds syntax to SQL for specifying references to data objects
infers DAG structure
enables runs with dependencies
scheduler integration
```

Question-bank detail:

```text
SQLX is provided by Dataform.
```

---

## 17.3 Cost

Dataform itself is free in the sense that:

```text
you pay for BigQuery when Dataform runs BigQuery work
```

Exam shortcut:

```text
Dataform cost = Dataform free, but BigQuery queries/jobs still cost money
```

---

# 18. SSD I/O and Garbage Collection

This appears in the question-bank “Other Cloud” review list.

## 18.1 HDD vs SSD basic difference

HDD:
- spinning disk
- mechanical movement
- sequential access much faster than random access

SSD:
- no spinning disk
- much faster random reads than HDD
- still has important write behavior constraints

---

## 18.2 SSD erase-block issue

SSDs write and erase differently.

Important idea:

```text
SSDs can write pages, but erase larger blocks.
```

This means overwriting small random pieces can be expensive.

Why?

```text
To overwrite data, SSD may need to copy still-valid pages elsewhere,
erase a larger block,
then write new data.
```

This background process is related to garbage collection.

---

## 18.3 SSD garbage collection

SSD garbage collection cleans up invalid/deleted pages so blocks can be reused.

Problem:

```text
random writes create scattered invalid pages
```

Then the SSD has to do more internal work:
- move valid data
- erase blocks
- rewrite pages

This causes write amplification.

Exam shortcut:

```text
random writes are worst for SSDs because of erase-block dynamics and garbage collection
```

---

## 18.4 Sequential vs random I/O

General rule:

```text
sequential I/O is usually better than random I/O
```

For disks:
- HDDs hate random I/O because seeking is slow.
- SSDs handle random reads better, but random writes can still be bad because of garbage collection and erase blocks.

Exam answer:

```text
SSD random writes are especially problematic compared with sequential writes.
```

---

# 19. GFLOPS Calculations

GFLOPS = billion floating-point operations per second.

```text
1 GFLOP/s = 10^9 floating-point operations per second
```

## 19.1 Basic formula

```text
GFLOPS = number of floating-point operations / runtime seconds / 10^9
```

Example:

```text
Program does 50 billion floating-point operations in 10 seconds.

GFLOPS = 50 billion / 10 / 1 billion
       = 5 GFLOPS
```

---

## 19.2 CPU peak GFLOPS style formula

If a CPU does:

```text
cores × clock rate × operations per cycle
```

then:

```text
peak FLOPS = cores × cycles/sec × FLOPs/cycle
```

Convert to GFLOPS:

```text
GFLOPS = peak FLOPS / 10^9
```

Example:

```text
4 cores
2.5 GHz
8 FLOPs/cycle

2.5 GHz = 2.5 × 10^9 cycles/sec

FLOPS = 4 × 2.5 × 10^9 × 8
      = 80 × 10^9

GFLOPS = 80
```

---

## 19.3 Common exam traps

### Trap 1: GHz already means billions per second

```text
2 GHz = 2 × 10^9 cycles/sec
```

### Trap 2: Multiply by cores

If each core can do work independently:

```text
total = per-core × number of cores
```

### Trap 3: FLOPs/cycle matters

If given vectorization/SIMD operations per cycle, include it.

### Trap 4: Runtime calculation

If asked runtime:

```text
seconds = total FLOPs / FLOPs per second
```

Example:

```text
100 billion operations
machine = 20 GFLOPS

20 GFLOPS = 20 billion ops/sec

seconds = 100 / 20 = 5 seconds
```

---

# 20. Cloud Cost and Performance Patterns

## 20.1 Things that often cost money

```text
running VMs
persistent disks
public IPs
object storage
database/query usage
egress network traffic
large analytical scans
logs/monitoring
```

---

## 20.2 Things often free or cheaper

```text
ingress into cloud
free-tier usage under quota
same-zone internal traffic
using managed services efficiently
stopping/deleting unused resources
```

Exact pricing varies, but the exam-level rule is:

```text
bringing data in is usually cheaper/free;
sending data out usually costs.
```

---

## 20.3 Managed service tradeoff

Managed services often cost more per visible unit than raw VMs, but save operational work.

Example:

```text
BigQuery may cost more than raw VMs for compute,
but you do not manage the distributed query system yourself.
```

Exam framing:

```text
IaaS gives control.
PaaS/SaaS reduces management burden.
```

---

# 21. Cloud Exam Patterns

## Pattern 1: IaaS/PaaS/SaaS

Question:

```text
Which service model gives you a VM?
```

Answer:

```text
IaaS
```

Question:

```text
Which service model is BigQuery closest to?
```

Answer:

```text
PaaS
```

Question:

```text
Which service model is Gmail?
```

Answer:

```text
SaaS
```

---

## Pattern 2: Regions and zones

Question:

```text
What is a region?
```

Answer:

```text
a geographic cloud location
```

Question:

```text
What is a zone?
```

Answer:

```text
a failure-isolated location inside a region
```

Question:

```text
Why use multiple zones?
```

Answer:

```text
higher availability if one zone fails
```

---

## Pattern 3: Spot/preemptible

Question:

```text
Which VM type is cheaper but can be interrupted?
```

Answer:

```text
spot/preemptible
```

Question:

```text
What workload is best for spot?
```

Answer:

```text
fault-tolerant batch/recomputable work
```

---

## Pattern 4: Ingress/egress

Question:

```text
Uploading data into cloud is what?
```

Answer:

```text
ingress
```

Question:

```text
Downloading data out of cloud is what?
```

Answer:

```text
egress
```

Question:

```text
Which usually costs?
```

Answer:

```text
egress
```

Question:

```text
Which is usually free?
```

Answer:

```text
ingress
```

---

## Pattern 5: GCS bucket

Question:

```text
What stores files like CSV/Parquet in GCP?
```

Answer:

```text
GCS bucket
```

---

## Pattern 6: Standard vs external table

Question:

```text
Which BigQuery table type copies data into Capacitor?
```

Answer:

```text
standard table
```

Question:

```text
Which BigQuery table type references GCS files?
```

Answer:

```text
external table
```

---

## Pattern 7: View vs materialized table

Question:

```text
Which is fresher after upstream updates?
```

Answer:

```text
view
```

Question:

```text
Which is usually faster/lower-cost for repeated queries?
```

Answer:

```text
materialized table / standard table
```

---

## Pattern 8: Dependency DAG

Question:

```text
Why track a dependency DAG?
```

Answer:

```text
to know what data resources depend on what, so refreshes happen in the correct order
```

---

## Pattern 9: Dataform

Question:

```text
What tool manages SQL pipeline DAGs for BigQuery and integrates with Git?
```

Answer:

```text
Dataform
```

Question:

```text
What extension comes from Dataform?
```

Answer:

```text
SQLX
```

---

## Pattern 10: SSD random writes

Question:

```text
Which SSD I/O pattern is worst?
```

Answer:

```text
random writes
```

Why:

```text
erase-block dynamics and garbage collection cause extra internal work
```

---

## Pattern 11: GFLOPS

Question:

```text
A machine does 40 billion floating-point operations in 5 seconds. GFLOPS?
```

Work:

```text
40 billion / 5 = 8 billion ops/sec = 8 GFLOPS
```

Answer:

```text
8 GFLOPS
```

---

# 22. Must-Memorize Sheet

```text
Cloud service models:
IaaS = rented infrastructure; VM is IaaS.
PaaS = managed platform; BigQuery is PaaS-ish.
SaaS = finished software; Gmail/Docs are SaaS.

Regions/zones:
Region = geographic cloud area.
Zone = isolated location inside a region.
Multi-zone improves availability.
Cluster = group of machines working together.

VM pricing:
On-demand VM = normal, more stable, more expensive.
Spot/preemptible VM = cheaper, can be interrupted.
Spot is good for batch/recomputable/fault-tolerant work.

Network:
Ingress = data entering cloud, usually free.
Egress = data leaving cloud, usually costs.
Cross-region/internet traffic is more likely to cost than same-zone traffic.

Storage:
GCS bucket = object/file storage.
Standard BigQuery table = data copied into Capacitor/Colossus.
External BigQuery table = references GCS files.
View = saved query, not stored result.
Materialized table = stored query result.

Freshness/performance:
Standard/materialized tables usually have better performance and lower repeated-query cost.
Views/external tables reflect upstream updates more directly.
Materialized data can become stale.
Refreshing materialized data requires dependency management.

DAG/Dataform:
DAG = Directed Acyclic Graph.
DAG tracks dependencies and refresh order.
Dataform manages SQL pipeline DAGs for BigQuery.
Dataform integrates with Git.
Dataform infers dependencies from references.
Dataform supports scheduling.
Dataform itself is free, but BigQuery jobs still cost.
SQLX comes from Dataform.

SSD:
Sequential I/O generally better than random.
SSD random writes are bad because of erase-block behavior and garbage collection.
Garbage collection moves valid pages and erases blocks.
Random writes cause write amplification.

GFLOPS:
GFLOPS = FLOPs / seconds / 10^9.
Peak FLOPS = cores × cycles/sec × FLOPs/cycle.
GHz = 10^9 cycles/sec.
```

---

# 23. Final Self-Test

## Service models

1. VM is which service model?  
**IaaS.**

2. BigQuery is closest to which service model?  
**PaaS.**

3. Gmail is which service model?  
**SaaS.**

---

## Regions/zones

4. What is a region?  
**Geographic cloud location.**

5. What is a zone?  
**Failure-isolated location inside a region.**

6. Why deploy across zones?  
**To survive a zone failure / improve availability.**

---

## VM pricing

7. Which VM type can be interrupted?  
**Spot/preemptible.**

8. Which VM type is better for critical stable services?  
**On-demand.**

9. Which workload fits spot/preemptible best?  
**Fault-tolerant batch/recomputable work.**

---

## Network

10. Uploading data into cloud is called what?  
**Ingress.**

11. Downloading data out of cloud is called what?  
**Egress.**

12. Which usually costs money?  
**Egress.**

13. Which is usually free?  
**Ingress.**

---

## Storage / BigQuery cloud slides

14. What does a GCS bucket store?  
**Objects/files like CSV and Parquet.**

15. Which table type copies data into BigQuery Capacitor storage?  
**Standard table.**

16. Which table type references files in GCS?  
**External table.**

17. What is a view?  
**A saved SQL query.**

18. Which is usually faster for repeated queries: view or materialized table?  
**Materialized/standard table.**

19. Which reflects upstream updates more directly: view or materialized table?  
**View.**

20. Why can materialized data be stale?  
**It stores old query results and does not automatically update when upstream data changes.**

---

## DAG/Dataform

21. What does DAG stand for?  
**Directed Acyclic Graph.**

22. Why track a dependency DAG?  
**To know what depends on what and refresh in the correct order.**

23. What does Dataform manage?  
**SQL pipeline DAGs for BigQuery.**

24. What does Dataform integrate with for version control?  
**Git.**

25. What SQL extension is associated with Dataform?  
**SQLX.**

26. What do you pay for when using Dataform?  
**BigQuery usage/jobs.**

---

## SSD / GFLOPS

27. Which SSD pattern is worst: sequential writes or random writes?  
**Random writes.**

28. Why are SSD random writes bad?  
**They cause extra erase-block garbage-collection work / write amplification.**

29. Formula for GFLOPS?  
**FLOPs / seconds / 10^9.**

30. 80 billion FLOPs in 20 seconds = ?  
**4 GFLOPS.**

31. 4 cores, 2 GHz, 4 FLOPs/cycle = ?  
**4 × 2 × 4 = 32 GFLOPS.**
