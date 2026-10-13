---
type: "practice"
difficulty: "Deep"
---

# How would you count unique visitors to a website approximately?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Count the number of distinct visitors to a website — but approximately, trading a little accuracy for a huge saving in memory. The word "approximately" is the whole point: the interviewer wants me to recognize this as a cardinality-estimation problem and reach for a probabilistic data structure — HyperLogLog (HLL) — rather than a SET or COUNT(DISTINCT). At "deep understanding" level they want: why exact counting doesn't scale, the intuition for why HLL works, its memory/accuracy characteristics, its mergeability (unions across servers/time windows), and the operational trade-offs versus alternatives (sampling, bitmap/Bloom, linear counting).

Clarifying questions:

What defines a "visitor" — a user ID, or a cookie/device fingerprint/IP? (The identifier is what we hash.)
Over what window — per hour, per day, all-time? And do we need to combine windows (e.g., "unique this week" = union of 7 daily counts)?
Cardinality scale — thousands, or hundreds of millions to billions? This decides whether exact counting is even feasible.
Acceptable error — is ±1–2% fine? For analytics dashboards it almost always is.
Do we ever need exact counts (billing, fraud)? Then approximation is out for those.

Assume: identifier = user/cookie ID, we want daily + rolling-window unique counts, cardinality up to hundreds of millions, ~1–2% error acceptable (typical analytics). Let me build up to HLL.

Why not exact?

Naive SET: put every visitor ID into a hash set, answer = SET.size. Correct, but memory grows linearly with the number of distinct visitors. Storing 100M unique 16-byte IDs is ~1.6 GB per counter — and we usually want many counters (per page, per day, per campaign). That's the killer.

COUNT(DISTINCT id) in a DB: either materializes all distinct values (same memory problem) or does an expensive sort/hash aggregation over the full event log on every query — slow and heavy at billions of rows.

The insight: we don't need the identities, only the count of distinct identities. That's exactly what probabilistic cardinality estimators exploit — they use constant, tiny memory regardless of cardinality.

HyperLogLog — the intuition

HLL is built on a beautifully simple observation about hashing and leading zeros:

Hash each visitor ID to a uniformly random bit string. Two visits by the same user hash to the same value → duplicates are naturally ignored (idempotent). This is why HLL de-dupes without storing IDs.
In a stream of random hashes, the maximum number of leading zeros you've seen is a proxy for how many distinct items there are. If you've seen a hash starting with k zeros, you've probably seen about 2^k distinct values — because a run of k zeros happens with probability 1/2^k, so seeing one implies you sampled ~2^k items.

A single "longest run of zeros" is a wildly noisy estimator, though. HLL fixes the variance with stochastic averaging:

Use the first p bits of the hash to pick one of m = 2^p registers (buckets). In each register, store the max number of leading zeros seen among hashes routed to that bucket. Now we have m independent estimators.
Combine them with a bias-corrected harmonic mean across all registers to produce the cardinality estimate. Averaging m buckets crushes the variance.

Accuracy/memory: the standard error is ≈ 1.04 / √m. With m = 16,384 registers (p = 14), error ≈ 0.81%, and each register needs only ~6 bits, so the whole structure is about 12 KB — and that's fixed whether you counted 1,000 or 1,000,000,000 visitors. That 12 KB → ~0.8% error, constant-memory result is the headline the interviewer wants. (Redis's HLL uses exactly p=14, ~12 KB, ~0.81% error, and switches from a sparse to dense encoding as cardinality grows to save memory for small counts.)

Why HLL is the right choice for websites: mergeability

The property that makes HLL perfect here is that HLL sketches are unionable. To get "unique visitors this week," I don't re-scan events — I take the 7 daily HLLs and merge them (register-wise max), then estimate. Merging is lossless and commutative, which means:

Distributed counting: each web server keeps its own HLL, ship the small sketches to a central place, merge → global uniques. No shuffling raw IDs across the network.
Roll-ups: hourly → daily → weekly → monthly, all by merging pre-aggregated sketches. Arbitrary date-range "uniques" become cheap.
Set unions: uniques(pageA ∪ pageB) = merge(HLL_A, HLL_B). (Note: HLL cannot do intersection directly — that needs inclusion-exclusion, which amplifies error, or a different structure.)
Operational design
CODE
Visit event → hash(visitor_id) → PFADD hll:day:2026-07-07:pageX
Query "uniques today"      → PFCOUNT hll:day:...:pageX
Query "uniques this week"  → PFMERGE tmp <7 daily keys> ; PFCOUNT tmp  (or PFCOUNT of many keys)
Ingestion: on each request, PFADD the visitor into the relevant day/segment HLL. PFADD is O(1); adding an existing member is a no-op (register max unchanged).
Storage: 12 KB per (day × segment). A year of daily per-page counts for 1,000 pages = 1000 × 365 × 12 KB ≈ 4.4 GB total — versus terabytes for exact sets.
Sharding/HA: HLLs live in Redis; replicate for HA. For very high write rates, keep per-shard HLLs and PFMERGE at read time.
Alternatives and when to prefer them
Random sampling (count 1/100 of traffic, multiply by 100): trivial and cheap, but variance is high for rare segments and it can't be merged cleanly across arbitrary windows without double-counting. Fine for coarse traffic estimates, poor for "distinct."
Bitmap / SETBIT (one bit per user ID): exact uniques, O(1) merge via bitwise OR, but only works when IDs are small dense integers (e.g., logged-in user IDs 1..N). Memory = N/8 bytes; great for "distinct logged-in users up to 100M" (~12 MB) but useless for high-cardinality cookie strings.
Bloom filter: tells membership (seen this exact user before?), can be adapted to count, but is bigger than HLL for pure cardinality and answers a different question.
Linear counting / KMV: other estimators; HLL wins on the memory-vs-accuracy frontier at high cardinality, which is why it's the industry default (Redis, Presto/Trino approx_distinct, BigQuery APPROX_COUNT_DISTINCT, Druid).
Correctness & pitfalls
Error is relative and probabilistic, not a hard bound — expect ±~0.8% typically, occasionally more. Never use HLL where an exact number is legally/financially required (billing, "you have 4 unique invited users").
Small cardinalities: raw HLL is biased for small n; production HLLs (Redis) apply bias correction / linear-counting for low counts, so small numbers stay accurate.
Intersections aren't native — computing "users who saw both A and B" via HLL inclusion-exclusion compounds error; reach for a different tool if that's the real need.
Hash quality matters: a good 64-bit hash (avoids collisions up to huge cardinalities) is assumed.
Summary — what a strong answer emphasizes
Recognize it as cardinality estimation and that "approximately" invites a probabilistic structure.
HyperLogLog: hash IDs → route by leading bits into m registers → track max leading-zero run per register → harmonic-mean estimate. ~12 KB, ~0.8% error, constant memory for any cardinality.
The decisive property is mergeability (register-wise max): distributed per-server counting and arbitrary window/segment roll-ups without moving raw IDs.
Contrast with exact SET/COUNT(DISTINCT) (linear memory), SETBIT bitmaps (exact but only for dense small IDs), and sampling (cheap but noisy, not mergeable).
State the trade-off honestly: not for exact/billing use, no native intersection.

#### Common mistakes
proposing a SET and not addressing memory; not knowing the error/memory numbers; forgetting mergeability (the reason HLL fits distributed web analytics); using it where an exact count is required. Follow-up probes: how the harmonic-mean/bias correction reduces variance, intersections via inclusion-exclusion, and choosing p to trade memory for accuracy.

**Comparison:**

```
Method	Memory for 100M uniques	Accuracy	Mergeable?	Best for
Exact SET / COUNT(DISTINCT)	~GBs (linear in n)	exact	via set union (costly)	small cardinality, exact needs
HyperLogLog	~12 KB (constant)	~0.8% error	yes (register max)	high-cardinality web analytics
Bitmap SETBIT	N/8 bytes (dense IDs)	exact	yes (bitwise OR)	dense small integer user IDs
Random sampling	tiny	high variance	no (double-count risk)	coarse traffic estimates
Bloom filter	larger than HLL	membership-based	yes	"have we seen this exact user?"
HLL parameter	Value	Consequence
p (index bits)	14	m = 16384 registers
register size	~6 bits	~12 KB total sketch
standard error	1.04/√m	~0.81%
add / count	O(1) / O(m)	write cheap, count trivial
```

**Code:**

```
Redis HyperLogLog — count unique visitors approximately
PFADD: add a visitor (O(1), duplicates are no-ops)

PFADD hll:uv:2026-07-07 user:42 cookie:abc user:42 # user:42 twice -> counted once PFCOUNT hll:uv:2026-07-07 # -> ~2 (approx distinct)

Weekly uniques = MERGE 7 daily sketches (register-wise max), then count

PFMERGE hll:uv:week:2026-W27
hll:uv:2026-07-06 hll:uv:2026-07-07 hll:uv:2026-07-08
hll:uv:2026-07-09 hll:uv:2026-07-10 hll:uv:2026-07-11 hll:uv:2026-07-12 PFCOUNT hll:uv:week:2026-W27

Or count the union on the fly without a persisted merge key:

PFCOUNT hll:uv:2026-07-06 hll:uv:2026-07-07 hll:uv:2026-07-08

CODE
Copy

```python
# Application: record a visit, query uniques (redis-py)
import redis
r = redis.Redis()

def record_visit(visitor_id: str, page: str, day: str):
    r.pfadd(f"hll:uv:{day}:{page}", visitor_id)   # O(1), auto-dedupes

def uniques_today(page: str, day: str) -> int:
    return r.pfcount(f"hll:uv:{day}:{page}")       # ~0.8% error

def uniques_range(page: str, days: list[str]) -> int:
    keys = [f"hll:uv:{d}:{page}" for d in days]
    return r.pfcount(*keys)                         # union across days, no re-scan
PYTHON
Copy
# Minimal HLL from scratch — shows the mechanism (p=14 -> m=16384, ~0.8% error)
import math, mmh3   # 64-bit MurmurHash

class HLL:
    def __init__(self, p=14):
        self.p, self.m = p, 1 << p
        self.reg = [0] * self.m
        self.alpha = 0.7213 / (1 + 1.079 / self.m)   # bias constant for large m

    def add(self, item: str):
        h = mmh3.hash64(item, signed=False)[0] & ((1 << 64) - 1)
        idx = h >> (64 - self.p)                      # first p bits -> register
        w = (h << self.p) & ((1 << 64) - 1)           # remaining bits
        rank = (64 - self.p) - w.bit_length() + 1 if w else (64 - self.p) + 1
        self.reg[idx] = max(self.reg[idx], rank)      # track max leading-zero run

    def count(self) -> float:
        Z = sum(2.0 ** -x for x in self.reg)          # harmonic-mean core
        return self.alpha * self.m * self.m / Z       # (Redis adds small-range correction)

    def merge(self, other: "HLL"):                    # unionable: register-wise max
        self.reg = [max(a, b) for a, b in zip(self.reg, other.reg)]
```

**Example input / output:**

```text
Feed 3 distinct users (each visiting many times):
  PFADD hll u1 u1 u1        -> stored once
  PFADD hll u2 u2
  PFADD hll u3
  PFCOUNT hll               -> 3        (exact at tiny n via correction)

Scale test (conceptual):
  add 1,000,000 distinct ids -> PFCOUNT -> ~1,001,900   (error ~0.19%, well under 1%)
  memory used                -> ~12 KB  (unchanged from the 3-user case)

Merge across days (union, no double count of returning users):
  day1 users {A,B,C}; day2 users {B,C,D}
  PFCOUNT day1 day2  -> ~4   (A,B,C,D)   not 6  -> returning users deduped by max-register
```

**Data:**

```

HLL register layout (p = 14): 64-bit hash: [ 14 bits: register index ][ 50 bits: pattern for leading-zero rank ] m = 2^14 = 16384 registers, ~6 bits each -> ~12 KB total, FIXED for any cardinality standard error ~= 1.04 / sqrt(16384) ~= 0.81%

CODE
Copy

```sql
-- Warehouse equivalents (same idea, different engines)
SELECT APPROX_COUNT_DISTINCT(visitor_id) FROM visits WHERE day = '2026-07-07'; -- BigQuery
SELECT approx_distinct(visitor_id)       FROM visits WHERE day = '2026-07-07'; -- Trino/Presto
```

**Flow:**

1. register max
2. Merge / roll-up
3. (HLL day1)
4. register-wise max
5. (HLL day2)
6. (HLL week)
7. PFCOUNT
8. Visit event: visitor_id
9. 64-bit hash
10. first p bits -> register index
11. remaining bits -> leading-zerorank
12. (m = 2^14 registers)
13. harmonic mean + bias corr ->estimate

