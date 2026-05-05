<!-- bigquery-study-guide.md -->

# BigQuery Final Exam Study Guide

Use this as your BigQuery guide. This is built around the question bank patterns and the three BigQuery slide decks.

---

# 1. BigQuery Architecture

## Core idea

BigQuery is a distributed analytical query system.

Memorize:

```text
BigQuery query engine = Dremel
BigQuery storage = Capacitor files in Colossus
```

Dremel provides the distributed query execution, while Capacitor files live in the Colossus file system.

## Exam answers

| Question style | Answer |
|---|---|
| BigQuery internal query engine? | **Dremel** |
| Storage system BigQuery uses? | **Colossus** |
| BigQuery internal storage format? | **Capacitor** |
| Similar system to BigQuery? | **Spark** |
| Similar system to HDFS? | **Colossus** |

---

# 2. BigQuery Billing

BigQuery has two main billing models.

## Capacity billing

You pay for **reserved slots**.

A slot is roughly:

```text
1/2 core + 1 GB RAM
```

Capacity billing is compute-based. You pay whether or not you use the slot.

### Good for

```text
predictable cost
predictable performance
not affected by other customers
```

### Bad because

```text
you pay even when idle
you cannot suddenly use tons of extra resources for a short burst
```

## On-demand billing

You pay for **Colossus I/O**, meaning bytes scanned/read.

Compute slots are effectively “free” to you, but BigQuery uses leftover capacity from the capacity system.

### Good for

```text
pay as you go
use nothing, pay nothing
can use huge compute if spare slots exist
```

### Bad because

```text
costs can surprise you
performance can depend on available leftover resources
```

## Very important billing trap

In normal cloud VM language:

```text
on-demand = non-preemptible
spot/preemptible = cheap leftover capacity
```

But in BigQuery:

```text
capacity = reserved
on-demand = uses leftover capacity
```

BigQuery “on-demand” has the opposite meaning from VM on-demand.

## Cost-control methods

To control on-demand cost:

```python
job_config = bigquery.QueryJobConfig(dry_run=True)
```

Use this to estimate cost before running.

```python
bigquery.QueryJobConfig(maximum_bytes_billed=200*1024**2)
```

Use this to cap max bytes billed.

Also know:

```text
daily quota
INFORMATION_SCHEMA.JOBS_BY_PROJECT
```

for checking expensive queries.

---

# 3. Partitioning vs Clustering

This is very likely exam material.

## Partitioning

Partitioning splits a table into mini-tables based on a partition column.

Example:

```text
date = 5/1/23 → partition 1
date = 5/2/23 → partition 2
```

A `WHERE` filter on the partition column lets BigQuery skip entire partitions, saving I/O cost.

### Exam phrasing

```text
Each unique value in the partition column corresponds to a partition.
```

### Good when

```text
queries often filter on the partition column
each partition has substantial data
```

### Bad when

```text
too many tiny partitions
filter is not on the partition column
```

## Clustering

Clustering stores data semi-sorted by cluster key.

The important idea:

```text
BigQuery can skip files whose cluster-key range cannot contain the answer.
```

Clustering is more flexible than partitioning because it supports more types and combinations of columns.

### “Semi-sorted” means

```text
files do not overlap much on the cluster key
but rows inside a file are not necessarily fully sorted
```

### Why unclustered rows can exist

New rows do not always trigger a full reorganization. BigQuery may keep some fresh data unclustered instead of constantly rewriting everything.

## Partitioning vs clustering table

| Concept | Partitioning | Clustering |
|---|---|---|
| Mental model | mini-tables | semi-sorted files |
| Saves cost by | skipping partitions | skipping files |
| Filter that helps | partition column | cluster key |
| Type flexibility | limited | broader |
| Common gotcha | tiny partitions bad | new rows may be unclustered |

---

# 4. Arrays, Structs, and Nested Data

BigQuery supports nested/repeated data.

## ARRAY

An array is a repeated field.

Access element by offset:

```sql
myarray[OFFSET(5)]
```

## STRUCT

A struct is like a record/object.

Access field:

```sql
mystruct.some_attribute
```

BigQuery tables can contain arrays of structs, such as books with repeated author records.

---

# 5. CROSS JOIN

## Normal cross join

A cross join pairs every row from table 1 with every row from table 2.

```sql
SELECT *
FROM tbl1
CROSS JOIN tbl2;
```

Comma syntax means the same thing:

```sql
SELECT *
FROM tbl1, tbl2;
```

## Row count rule

```text
rows output = rows in table 1 × rows in table 2
```

Example:

```text
tbl1 has 2 rows
tbl2 has 3 rows
output = 6 rows
```

## Filtering cross joins

Cross joins do not use `ON`.

They can use `WHERE`:

```sql
SELECT *
FROM tbl1
CROSS JOIN tbl2
WHERE tbl1.x = tbl2.x;
```

Query engines can sometimes optimize certain `WHERE` filters with cross joins.

---

# 6. UNNEST and Correlated Cross Join

This is probably the highest-yield BigQuery concept.

## UNNEST

`UNNEST` turns an array into rows.

Example:

```sql
SELECT x, y, z
FROM tbl
CROSS JOIN UNNEST(tbl.coord);
```

`UNNEST(tbl.coord)` creates a different logical table for each original row.

## Correlated cross join meaning

“Correlated” means:

```text
each row only cross joins with arrays from that same row
```

It does **not** mix unnested values across different original rows.

No output row contains values from unrelated original rows.

## Exam row-count formula

For each original row:

```text
output rows = length(array y) × length(array z)
```

Then add across all original rows.

## Example 1

```text
x, y, z
1, [2, 3], [4]
5, [6, 7], [8, 9, 10]
```

Row 1:

```text
2 × 1 = 2
```

Row 2:

```text
2 × 3 = 6
```

Total:

```text
2 + 6 = 8
```

## Example 2

```text
x, y, z
1, [2, 3], [4, 5]
6, [7], [8, 9, 10]
```

Row 1:

```text
2 × 2 = 4
```

Row 2:

```text
1 × 3 = 3
```

Total:

```text
7
```

## Shortcut

When you see:

```text
correlated cross join after unnesting y and z
```

Think:

```text
DO NOT multiply all arrays globally.
Multiply within each row, then sum.
```

---

# 7. BigQuery Geography

BigQuery supports geographic operations.

## Key facts

```text
BigQuery geography uses latitude/longitude by default.
No altitude by default.
```

Shape constructors:

```sql
ST_GEOGPOINT
ST_MAKELINE
ST_MAKEPOLYGON
```

BigQuery supports common geo operations like geographic joins.

## Scalar vs aggregate geo functions

Question bank pattern:

```sql
SELECT FUNC(geom)
FROM geotable;
```

Which creates more output rows?

```text
ST_CENTROID creates one output per input row.
ST_CENTROID_AGG aggregates rows, so fewer rows.
```

So:

```text
ST_CENTROID generally produces more rows than ST_CENTROID_AGG.
```

---

# 8. BigQuery ML

BigQuery ML lets you train models using SQL.

BigQuery ML topics include:
- `AI.CLASSIFY`
- `CREATE MODEL`
- `TRANSFORM`
- `ML.*` functions for prediction/evaluation/inspection

## Training syntax

Basic model training:

```sql
CREATE OR REPLACE MODEL myproj.mydataset.mymodel
OPTIONS(
  MODEL_TYPE='LINEAR_REG',
  INPUT_LABEL_COLS=['temp']
)
AS
SELECT yesterday_temp, humidity, temp
FROM weather;
```

Meaning:

```text
MODEL_TYPE = what kind of model
INPUT_LABEL_COLS = column to predict
all other selected columns = features
```

## BigQuery ML hierarchy

```text
project
  dataset
    tables
    models
```

So a model name looks like:

```text
myproj.mydataset.mymodel
```

## Model functions

| Function | Use |
|---|---|
| `ML.WEIGHTS` | inspect coefficients/weights |
| `ML.PREDICT` | make predictions |
| `ML.EVALUATE` | evaluate model quality |

## Exam trap

Question:

```text
Which function gives coefficients used to multiply features?
```

Answer:

```text
ML.WEIGHTS
```

Not `ML.PREDICT`, not `ML.EVALUATE`.

---

# 9. TRANSFORM Clause

This is high-yield because it appears directly in the question bank.

Question:

```text
Which clause related to machine learning does BigQuery add to SQL?
```

Answer:

```text
TRANSFORM
```

BigQuery ML uses `TRANSFORM` for feature engineering before training/prediction.

## Why it matters

Feature transformation lets you convert raw features into better features.

Examples:

```text
x → x^2
category → one-hot encoding
```

Linear models may need transformed features to capture nonlinear patterns or categorical variables.

---

# 10. Train/Test Split

BigQuery has unusual default splitting.

Default behavior:

```text
<500 rows: 100% training
<50K rows: 80% training
larger: 10K rows test, rest training
```

Recommended approach:

```sql
RAND() < ratio
```

and disable automatic splitting:

```sql
DATA_SPLIT_METHOD='NO_SPLIT'
```

## Exam point

If you see:

```sql
RAND() < 0.8
```

That gives a random approximate 80/20 split.

It will not be perfectly exact because `RAND()` is random.

---

# 11. AI.CLASSIFY

BigQuery can call LLM-style classification from SQL.

Basic form:

```sql
SELECT value, AI.CLASSIFY(value, ["A", "B", "C"])
FROM table;
```

With endpoint:

```sql
SELECT value, AI.CLASSIFY(value, ["A", "B", "C"], endpoint => 'gemini-2.5-flash')
FROM table;
```

With label descriptions:

```sql
SELECT value, AI.CLASSIFY(value, [("A", "description"), ("B", "description")])
FROM table;
```

---

# 12. File Formats

## Column-oriented formats

Column-oriented:

```text
Capacitor
ColumnIO
Parquet
```

Not column-oriented:

```text
CSV
```

CSV is not column-oriented because it stores rows as full lines.

## Parquet inspiration

Parquet was inspired by:

```text
ColumnIO
```

ColumnIO is the column format from the Dremel paper.

## Capacitor access

If you want BigQuery data stored in Capacitor format:

```text
use a load job
```

Why:

```text
Capacitor is BigQuery’s internal storage format.
External tables keep data in external formats like CSV/Parquet.
```

---

# 13. High-Yield BigQuery Question Patterns

## Pattern 1: Correlated cross join row counting

Question gives:

```text
x, y, z
1, [..], [..]
2, [..], [..]
```

Answer method:

```text
multiply array lengths per row
sum totals
```

## Pattern 2: Billing leftovers

If question says:

```text
uses leftover CPU/memory
```

Answer may be:

```text
spare capacity / on-demand-style leftover slots
```

Exam strategy:

```text
If “billing model” choices are capacity vs on-demand → choose on-demand.
If “spare” is an option and wording says leftover resources → likely spare.
```

## Pattern 3: Capacity billing free item

Question:

```text
What does capacity billing give for free?
```

Answer:

```text
Colossus I/O
```

Capacity billing charges for slots/compute, not per-byte I/O.

## Pattern 4: BigQuery engine

Question:

```text
BigQuery query engine is based on what?
```

Answer:

```text
Dremel
```

## Pattern 5: ML clause

Question:

```text
What ML clause does BigQuery add?
```

Answer:

```text
TRANSFORM
```

## Pattern 6: Coefficients

Question:

```text
How do you inspect coefficients?
```

Answer:

```text
ML.WEIGHTS
```

---

# 14. Last-Minute Memorization Sheet

Memorize these directly:

```text
BigQuery engine = Dremel
BigQuery storage = Capacitor files in Colossus
Slot ≈ 1/2 core + 1 GB RAM
Capacity billing = pay for reserved slots
On-demand billing = pay for Colossus I/O / bytes scanned
On-demand uses leftover capacity
Capacity billing gives Colossus I/O for free
Partitioning = mini tables
Clustering = semi-sorted files
ARRAY access = arr[OFFSET(i)]
STRUCT access = struct.field
CROSS JOIN = every pair of rows
UNNEST = array to rows
Correlated UNNEST = only within same original row
Correlated row count = multiply array lengths per row, then sum
BigQuery geo = lat/lon by default, no altitude
Geo constructors = ST_GEOGPOINT, ST_MAKELINE, ST_MAKEPOLYGON
CREATE MODEL = train model
INPUT_LABEL_COLS = label column
ML.WEIGHTS = coefficients
ML.PREDICT = predictions
ML.EVALUATE = metrics
TRANSFORM = feature engineering
AI.CLASSIFY = LLM classification in SQL
CSV = not column-oriented
Parquet inspired by ColumnIO
Capacitor data requires load job
```

---

# 15. 20-Minute Drill

Do these without notes.

## Q1

```text
x, y, z
1, [2,3], [4,5]
6, [7], [8,9,10]
```

Correlated cross join after unnesting `y` and `z`: how many rows?

Answer:

```text
Row 1: 2×2 = 4
Row 2: 1×3 = 3
Total = 7
```

## Q2

BigQuery on-demand billing charges based on what?

Answer:

```text
Colossus I/O / bytes scanned
```

## Q3

Capacity billing pays for what?

Answer:

```text
reserved slots
```

## Q4

What does partitioning skip?

Answer:

```text
whole partitions / mini tables
```

## Q5

What does clustering skip?

Answer:

```text
files/subfiles based on cluster-key ranges
```

## Q6

Which function gives model coefficients?

Answer:

```text
ML.WEIGHTS
```

## Q7

Which function makes predictions?

Answer:

```text
ML.PREDICT
```

## Q8

Which BigQuery ML clause handles feature engineering?

Answer:

```text
TRANSFORM
```

## Q9

BigQuery query engine?

Answer:

```text
Dremel
```

## Q10

BigQuery internal storage format?

Answer:

```text
Capacitor
```

---

# 16. What to actually study first

Priority order:

```text
1. Correlated cross join / UNNEST row counting
2. Capacity vs on-demand billing
3. Partitioning vs clustering
4. BigQuery ML: CREATE MODEL, TRANSFORM, ML.WEIGHTS/PREDICT/EVALUATE
5. Dremel / Colossus / Capacitor facts
6. Geo constructors and lat/lon default
7. Column formats: CSV vs Parquet/ColumnIO/Capacitor
```

For the exam, the most question-bank likely thing is:

```text
UNNEST correlated cross join row counting
```

That should be automatic.
