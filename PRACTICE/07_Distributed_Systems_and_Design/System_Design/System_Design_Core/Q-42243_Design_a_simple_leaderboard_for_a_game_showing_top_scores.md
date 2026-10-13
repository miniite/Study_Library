---
type: "practice"
difficulty: "Intermediate"
---

# Design a simple leaderboard for a game showing top scores.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Design a leaderboard service that shows the top-N scores for a game (e.g., top 100 players by points). At the "intermediate" level the interviewer wants to see that I understand why a naive SELECT ... ORDER BY score DESC LIMIT 100 gets expensive at scale, and that I can reach for a sorted-set data structure (Redis ZSET) as the core primitive. They're probing: data-model choice, read/write patterns, how to also return a specific player's rank (not just the top list), and basic operational concerns (persistence, consistency, ties).

Clarifying questions I'd ask up front:

Is it a global all-time leaderboard, or time-windowed (daily/weekly/seasonal)? This drives keying and expiry.
How many players? (Thousands vs. hundreds of millions changes sharding needs.)
Read/write ratio and update rate? (A casual game vs. a live esports match.)
Do we need only the top-N, or also "my rank" and "players around me"? The last one is the interesting requirement.
Is score monotonically increasing (high score) or the latest score? Affects update semantics.

I'll assume: a single global (or per-window) leaderboard, up to ~10M players, we need top-N and a given user's rank/neighbors, score = best score achieved. Now let me reason to the design.

From first principles

A leaderboard is fundamentally an ordered set keyed by score. The three operations we must make fast are:

updateScore(user, score) — write.
getTopN(N) — read a prefix of the ordering.
getRank(user) and getAround(user, k) — a random-access read into the ordering.

Why not just a SQL table? With a B-tree index on score, ORDER BY score DESC LIMIT 100 is cheap (index scan of 100 rows). The pain is getRank(user): to know a user is rank 3,472,118 you must count how many rows have a higher score — SELECT COUNT(*) WHERE score > ?. That's an O(N) scan on every profile view, and it's the query that kills a SQL-only design under load. This is the crux the interviewer wants me to surface.

The right primitive: a sorted set. A Redis Sorted Set (ZSET) stores member -> score and keeps members ordered by score using a skip list + hash map. This gives:

ZADD key score member — insert/update in O(log N).
ZREVRANGE key 0 N-1 WITHSCORES — top-N in O(log N + N).
ZREVRANK key member — a member's rank in O(log N) (the skip list maintains span counts, so rank is not an O(N) count).
ZSCORE, and ZREVRANGE start stop for neighbors around a user.

So the operation SQL can't do cheaply — rank — is exactly what a ZSET makes O(log N). That's the whole reason this pattern exists.

Architecture
CODE
Game clients → API/Score service → Redis ZSET (leaderboard)  ← reads (top-N, rank, neighbors)
                     │
                     └──→ Durable store (Postgres/DynamoDB): system of record for scores
Score service validates a submitted score (anti-cheat, auth), writes the authoritative score to the durable DB, and updates Redis.
Redis is the serving layer for all leaderboard reads. It's a cache/index, not the source of truth — if it's lost, we can rebuild it from the DB.
Durable store keeps (user_id, score, updated_at) so we can recover, audit, and support "score = best": we only update if the new score beats the stored one.

Update flow (best-score semantics): Use ZADD key GT score member — the GT flag updates only if the new score is greater than the current, which matches "keep the player's highest score" and is atomic, avoiding a read-modify-write race. Write to the DB in the same transaction/handler.

Top-N read: ZREVRANGE lb:global 0 99 WITHSCORES returns the top 100 with scores. Hydrate display names from a user cache. This is the hot path — cache the serialized top-N for a few seconds if traffic is huge, since the top rarely changes.

"My rank" and neighbors: ZREVRANK lb:global {user} for rank; ZREVRANGE lb:global {rank-5} {rank+5} WITHSCORES for the players around them. Both O(log N + k) — instant.

Handling ties

Two players with equal score: ZSET breaks ties lexicographically by member, which is arbitrary. If the product wants "earlier achiever ranks higher," encode a composite score: score * 10^13 + (maxTimestamp - firstAchievedAt) packed into the float, so the ordering is score-first, time-second. (Redis floats are 64-bit doubles — mind precision; if scores are large, keep the tiebreaker small or use a separate secondary ZSET.)

Time-windowed leaderboards

Very common in games. Key by window: lb:daily:2026-07-07, lb:weekly:2026-W27, lb:season:5. Set a TTL just past the window (EXPIRE) so old boards self-clean. A user's score often goes to multiple keys on each update (all-time + current daily + weekly). This is a handful of O(log N) writes — cheap.

Persistence, availability, consistency
Persistence: Enable Redis AOF (append-only file) so a restart doesn't lose recent updates; still keep the DB as the real source of truth and have a rebuild job (ZADD all users) as the recovery path.
Availability: Redis primary + replica; reads can go to replicas. Use Redis Sentinel or a managed offering (ElastiCache/MemoryStore) for failover.

#### Cons
istency: The leaderboard is eventually consistent and that's acceptable — a player briefly seeing rank 51 vs 50 is fine. The DB write is what must be durable; Redis can lag by milliseconds.
Scaling notes (enough for "intermediate")

A single Redis ZSET comfortably holds tens of millions of members. If a single global board grows past one node's memory/throughput, options are: shard by region/game-mode (natural product boundaries), or use bucketed ZSETs (many small boards merged for the global view) — but I'd only introduce sharding when a real limit is hit, since a global top-N across shards requires scatter-gather and re-merge. For the stated scale, one ZSET (with replicas) is the right, simple answer.

Summary — what a strong answer emphasizes
Recognize the leaderboard as an ordered-set problem, and that the hard operation is rank, not top-N.
Use a Redis Sorted Set: ZADD (O(log N)) to write, ZREVRANGE for top-N, ZREVRANK for rank, all O(log N).
Keep a durable DB as source of truth; Redis is a rebuildable serving index.
Handle ties with a composite score, windows with keyed+TTL boards, best-score with ZADD GT.
Discuss persistence/failover and note that eventual consistency is acceptable here.

#### Common mistakes
proposing only SQL and missing that getRank is O(N); forgetting the "player's own rank/neighbors" requirement; treating Redis as the source of truth with no recovery path; ignoring ties and time windows. Follow-up probes usually go to: sharding a global board, exact vs approximate rank at billion scale, and anti-cheat on score submission.

**Comparison:**

```
Operation	SQL (scores table)	Redis ZSET	Winner
Update score	O(log N) index write	ZADD O(log N)	tie
Top-N	ORDER BY score DESC LIMIT N — O(log N + N)	ZREVRANGE O(log N + N)	tie
Get a user's rank	COUNT(*) WHERE score > ? — O(N)	ZREVRANK O(log N)	ZSET
Players around a user	needs rank first → O(N)	ZREVRANGE O(log N + k)	ZSET
Durability / audit	strong (source of truth)	needs AOF; rebuildable	SQL
Concern	Choice	Why
Serving store	Redis ZSET (+replica)	O(log N) rank + top-N in memory
Source of truth	Postgres/DynamoDB	durability, audit, rebuild Redis
Best-score update	ZADD ... GT	atomic, no read-modify-write race
Time windows	keyed boards + EXPIRE	self-cleaning daily/weekly/season
Consistency	eventual (Redis) / durable (DB)	rank lag is fine; write must persist
```

**Code:**

```
Redis commands — core leaderboard operations
Update a player's best score (GT = only if greater), atomic

ZADD lb:global GT 15230 user:42

Top 10 with scores, highest first

ZREVRANGE lb:global 0 9 WITHSCORES

A player's rank (0-based; add 1 for display)

ZREVRANK lb:global user:42 # -> 3471 (so rank #3472)

Player's own score

ZSCORE lb:global user:42

5 players above and below a given rank (neighbors)
if ZREVRANK returned r, fetch [r-5, r+5]

ZREVRANGE lb:global 3466 3476 WITHSCORES

Total players

ZCARD lb:global

Time-windowed board with auto-expiry (daily)

ZADD lb:daily:2026-07-07 GT 15230 user:42 EXPIRE lb:daily:2026-07-07 172800 # keep ~2 days

CODE
Copy

```python
# Score service handler (Python, redis-py) — write path
import redis, time
r = redis.Redis()

def submit_score(user_id: int, score: int, db):
    member = f"user:{user_id}"
    # 1) durable source of truth (best score)
    db.execute(
        "INSERT INTO scores(user_id, score, updated_at) VALUES(%s,%s,now()) "
        "ON CONFLICT(user_id) DO UPDATE SET score=GREATEST(scores.score, EXCLUDED.score), "
        "updated_at=now()",
        (user_id, score),
    )
    # 2) serving index: update only if greater (best-score semantics)
    pipe = r.pipeline()
    pipe.zadd("lb:global", {member: score}, gt=True)
    today = time.strftime("lb:daily:%Y-%m-%d")
    pipe.zadd(today, {member: score}, gt=True)
    pipe.expire(today, 172800)
    pipe.execute()

def get_top(n=100):
    rows = r.zrevrange("lb:global", 0, n - 1, withscores=True)
    return [{"user": m.decode(), "score": int(s)} for m, s in rows]

def get_rank_and_neighbors(user_id: int, window=5):
    m = f"user:{user_id}"
    rank = r.zrevrank("lb:global", m)      # None if not present
    if rank is None:
        return {"rank": None, "neighbors": []}
    lo, hi = max(0, rank - window), rank + window
    rows = r.zrevrange("lb:global", lo, hi, withscores=True)
    neighbors = [{"user": mm.decode(), "score": int(s)} for mm, s in rows]
    return {"rank": rank + 1, "neighbors": neighbors}
```

**Example input / output:**

```text
State (lb:global) after these updates:
  ZADD lb:global GT 100 A
  ZADD lb:global GT 300 B
  ZADD lb:global GT 200 C
  ZADD lb:global GT 250 A      # A improves 100 -> 250 (GT applies)
  ZADD lb:global GT 240 A      # 240 < 250, GT ignores it (best score kept)

Ordered (high -> low): B=300, A=250, C=200

ZREVRANGE lb:global 0 1 WITHSCORES   -> [B, 300, A, 250]   # top 2
ZREVRANK  lb:global A                 -> 1                   # rank #2 (0-based)
ZSCORE    lb:global A                 -> 250
ZREVRANGE lb:global 0 -1 WITHSCORES  -> [B,300, A,250, C,200]
```

**Data:**

```

-- Durable source of truth (Postgres): rebuild Redis from this on cold start CREATE TABLE scores ( user_id BIGINT PRIMARY KEY, score BIGINT NOT NULL, updated_at TIMESTAMPTZ NOT NULL DEFAULT now() ); CREATE INDEX idx_scores_score ON scores (score DESC); -- for top-N fallback / rebuild

CODE
Copy

```json
// GET /leaderboard/top?n=3  -> response
{
  "window": "global",
  "top": [
    { "rank": 1, "user": "user:9001", "name": "Aria",  "score": 98230 },
    { "rank": 2, "user": "user:42",   "name": "Kabir", "score": 91500 },
    { "rank": 3, "user": "user:777",  "name": "Meera", "score": 88010 }
  ]
}
```

**Flow:**

1. Reads
2. submit score
3. validate + auth
4. 1. best score, source oftruth
5. 2. ZADD GT
6. rebuild on cold start
7. Game client
8. Score service
9. (Postgres: scores)
10. (Redis ZSET: lb:global +lb:daily)
11. getTopN
12. getRank / neighbors

