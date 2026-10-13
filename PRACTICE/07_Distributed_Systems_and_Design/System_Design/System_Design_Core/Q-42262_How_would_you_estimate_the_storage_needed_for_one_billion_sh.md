---
type: "practice"
difficulty: "Deep"
---

# How would you estimate the storage needed for one billion short URLs?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Given a URL-shortener storing 1 billion short URLs, estimate the storage needed. This is a classic back-of-the-envelope (BOTE) sub-problem. The interviewer wants to see a disciplined method: state assumptions, size a single record, multiply by count, add overhead (indexes, replication, metadata), and sanity-check the magnitude — all while doing round-number mental math, not a spreadsheet.

What they're probing (deep-understanding level): Can you decompose a record into fields, pick reasonable byte sizes, remember the hidden multipliers (primary-key/secondary indexes, replication factor, cache, backups, growth), and land on a number you can defend? A weak answer gives a single figure; a strong answer gives a range with the assumptions that drive it, and knows that raw row bytes are only ~a third of real footprint.

Clarifying questions I'd ask:

Is 1 B the total stored or created per year? (Affects growth planning.)
What fields per record — do we store analytics (click counts, per-click logs)? That dwarfs the URL table.
Short-code length / alphabet? (Determines key size and namespace.)
Retention — do links expire, or is it forever?
Replication factor and whether we need backups/cache in the estimate.

Plan: size one row → multiply → apply index + replication + overhead multipliers → sanity-check → discuss what changes the answer (analytics is the big one).

Step 1 — What's in one record?

A minimal short-URL row:

Field	Purpose	Bytes (est.)
short_code	e.g. 7 chars, base62	~7
long_url	original URL	~100 (avg; URLs vary 50–500)
user_id	owner (nullable)	8
created_at	timestamp	8
expires_at	TTL	8
click_count	counter (if denormalized)	8
Field total		~140 bytes

Real databases add per-row overhead: row headers, null bitmaps, padding/alignment, and the primary-key/index structures. A common rule of thumb is to assume ~500 bytes per row all-in once you count the row itself plus its position in the primary B-tree and a secondary index on short_code. I'll carry two figures: a lean ~200 B/row and a realistic ~500 B/row.

Step 2 — Multiply by 1 billion

Use powers of two for mental math: 1 billion ≈ 10⁹, and 1 GB ≈ 10⁹ bytes, 1 TB ≈ 10¹² bytes. So bytes per row = GB total conveniently.

Lean data only (200 B): 200 B × 10⁹ = 2 × 10¹¹ B = 200 GB.
Realistic with row overhead (500 B): 500 B × 10⁹ = 5 × 10¹¹ B = 500 GB ≈ 0.5 TB.
Step 3 — Apply the hidden multipliers

Raw table size is not the storage you provision. Layer on:

Indexes. You need a primary index (the short_code is the lookup key) and often a secondary index (by user_id, or created_at for expiry sweeps). Each B-tree index on the code/id adds roughly another 30–100% of the data size. Call it +50%.
Replication. Production DBs keep 3 copies (1 primary + 2 replicas, or quorum systems like Cassandra with RF=3). ×3.
Backups / snapshots. Point-in-time recovery keeps several days of copies — add ~1× the primary data as a rough allowance.
Cache (Redis). Hot keys (say the top ~20% by traffic) cached in memory. Often a separate budget (RAM), not disk, but worth stating.
Growth headroom. Never fill to 100%; provision to ~50–70%. Add ~1.5× headroom.

Rolling the disk multipliers together on the realistic base (0.5 TB):

Data: 0.5 TB
indexes (×1.5): 0.75 TB
× replication (×3): 2.25 TB
backups (~+0.75 TB): 3.0 TB
× growth headroom (×1.5): ~4.5 TB provisioned

So: raw ~0.5 TB → realistic provisioned ~4–5 TB. State the range and the drivers, not a false-precision single number.

Step 4 — Sanity check the magnitude

1 B rows at ~0.5 KB each landing at hundreds of GB raw and a few TB replicated is exactly the right order of magnitude. This is small by modern standards — it fits on a handful of commodity disks, or a single well-provisioned Postgres/MySQL instance with replicas, or trivially in a sharded KV store (DynamoDB/Cassandra). The mapping table is not the scaling challenge of a URL shortener — read QPS and hot-key caching are. Recognizing that shows maturity.

Step 5 — What changes the answer (the big one: analytics)

If we log every click, the click-event table dwarfs the mapping table. Suppose an average 100:1 read-to-write ratio and, say, 100 B clicks/day → 10¹¹ click rows/day, each maybe 50–100 B → ~5–10 TB/day. That's where a URL shortener actually spends storage, and you'd push it to columnar/cold storage (S3 + Parquet, ClickHouse, BigQuery) with aggregation and TTL, not the OLTP mapping DB. If the interviewer only means the mapping table, the answer is the ~few-TB figure; always separate the two.

Other levers:

Longer URLs (avg 100 → 300 B) roughly triples the long_url contribution.
Compression (many DBs / columnar stores compress URLs 2–4×) shrinks it back.
Short-code length: 7 base62 chars give 62⁷ ≈ 3.5 × 10¹² codes — plenty of namespace for 10⁹ URLs (0.03% used), so no key-space pressure.
Namespace cross-check (bonus)

Does the key length even support 1 B? base62 with n chars = 62ⁿ.

6 chars → 62⁶ ≈ 5.7 × 10¹⁰ (57 B) — enough but tightish.
7 chars → 62⁷ ≈ 3.5 × 10¹² (3.5 T) — comfortable. So a 7-char code is the right choice; it costs 7 bytes/row and never runs out.
Summary / what a strong answer emphasizes
Method over number: size one row (~140 B fields, ~500 B all-in), × 10⁹ → ~0.5 TB raw, then apply indexes (×1.5), replication (×3), backups, growth (×1.5) → ~4–5 TB provisioned.
Use 1 GB ≈ 10⁹ B, so bytes/row ≈ GB total for 1 B rows — clean mental math.
State assumptions and give a range, not false precision.
Note that raw row bytes are only ~⅓ of real footprint — indexes + replicas + backups dominate.
Call out that click analytics, not the mapping table, is the real storage cost, and that this dataset is small enough that read throughput and caching, not storage, are the actual design challenge.

#### Common mistakes
forgetting index/replication/backup multipliers; giving one over-precise number; using 1 GB = 2³⁰ and doing hard binary math under pressure (round to 10⁹ instead); ignoring the analytics table.

#### Follow-up
probes: "What if URLs average 500 B?" "How would you shard this?" "Where do click logs go?" "How big is the in-memory cache for hot links?"

**Comparison:**

```
Step	Component	Multiplier	Running total
1	Raw rows (500 B × 1e9)	—	0.50 TB
2	+ Secondary indexes	×1.5	0.75 TB
3	× Replication (RF=3)	×3	2.25 TB
4	+ Backups / PITR	+0.75 TB	3.00 TB
5	× Growth headroom	×1.5	~4.5 TB
Assumption changed	Effect on raw table
Row 200 B (lean) vs 500 B (realistic)	0.2 TB vs 0.5 TB
Avg URL 100 B → 300 B	url field ~×3 → +~0.2 TB
Compression 2–4× on URLs	shrinks url contribution ~½–¼
Add per-click analytics	+TB per day (separate cold store)
Base62 code length	Namespace (62ⁿ)	Enough for 1 B?
6 chars	~5.7 × 10¹⁰	Yes, but tight
7 chars	~3.5 × 10¹²	Comfortable ✅
8 chars	~2.2 × 10¹⁴	Overkill
```

**Example input / output:**

```text
INPUT:  1,000,000,000 short URLs, ~500 B all-in per row

STEP 1  raw table   = 500 B * 1e9        = 5e11 B      = 0.5 TB
STEP 2  + indexes   = 0.5 TB * 1.5       = 0.75 TB
STEP 3  * replicas  = 0.75 TB * 3        = 2.25 TB
STEP 4  + backups   = 2.25 + 0.75        = 3.0 TB
STEP 5  * headroom  = 3.0 TB * 1.5       = 4.5 TB

OUTPUT: raw ~0.5 TB  ->  provisioned ~4-5 TB  (mapping table only)

Namespace check: 62^7 = 3.5e12 >> 1e9  -> 7-char code is safe (0.03% used)

If click analytics logged (~100 B clicks/day, 50-100 B each):
  ~5-10 TB/DAY -> route to columnar/cold store (S3+Parquet / ClickHouse), TTL'd.
  => analytics, not the mapping table, dominates storage.
```

**Data:**

```yaml
# Assumptions carried through the estimate (state these out loud)
count_urls:            1_000_000_000        # 1 B short URLs
avg_long_url_bytes:    100                  # typical; range 50-500
fields_per_row_bytes:  140                  # code+url+user+timestamps+counter
allin_row_bytes:       500                  # incl. row header + PK/index cell overhead
index_overhead_factor: 1.5                  # secondary indexes ~ +50%
replication_factor:    3                    # 1 primary + 2 replicas
backup_allowance:      1.0                  # ~1x primary data for PITR snapshots
growth_headroom:       1.5                  # never provision to 100%
short_code:            "7 base62 chars"     # 62^7 ~= 3.5e12 namespace
units:                 "1 GB = 1e9 B, 1 TB = 1e12 B"
```

**Flow:**

1. Estimation pipeline (BOTE):
2. [1 row fields ~140B] --add row+index cell overhead--> [~500B all-in / row]
3. |
4. x 1,000,000,000 rows  (1 GB = 1e9 B, so B/row ≈ GB total)
5. v
6. [raw table ~0.5 TB]
7. |
8. x1.5 indexes -> x3 replication -> +backups -> x1.5 headroom
9. v
10. [provisioned ~4-5 TB]   <-- mapping table only
11. (Click analytics -> SEPARATE cold/columnar store, TTL'd, ~TB/day)

