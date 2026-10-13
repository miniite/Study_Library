---
type: "practice"
difficulty: "Deep"
---

# Design a rate-of-change alerting or a leaderboard system. What data structures and classes?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Classic LLD Problems](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Classic_LLD_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what's really being asked

The prompt offers two related design problems — a rate-of-change alerting system or a leaderboard system — and asks specifically about data structures and classes. Both are, at their core, the same LLD skill: pick a data structure whose operations match the query pattern, then wrap it in clean, extensible classes. The interviewer (Dream11/MPL/Zynga gaming tier, plus Amazon/Google/Uber) is probing whether I choose the right structure by reasoning about the required operations and their complexity — not by defaulting to a sorted list. Because gaming companies live and die on leaderboards, I'll make leaderboard the primary design and briefly show how the same reasoning transfers to rate-of-change alerting.

Clarify the query pattern first (this is the whole game): a leaderboard needs, cheaply and concurrently:

addOrUpdateScore(playerId, delta) — scores change constantly.
getRank(playerId) — "you are #4,271."
getTopK(k) — top 10 / 100.
getRange(rank, rank+n) — "around me" (the page of players near you).

The naive HashMap<player,score> gives O(1) update but O(N log N) for rank/top-K. A sorted ArrayList gives O(1) top-K read but O(N) insert. Neither is good. The right structure is one that keeps things ordered and gives rank cheaply.

Plan: state the operations + complexity target → pick the data structure (skip list / balanced BST with subtree counts, i.e. an order-statistics tree; production = Redis Sorted Set) → design the classes → give the alerting variant → discuss scale/failure modes.

Choosing the data structure — reason from the operations

I need ordered by score with two extra tricks: rank by score and k-th element. The data structures that give this:

Balanced BST augmented with subtree size (an order-statistics tree). Each node stores the size of its subtree, so getRank and select(k) are O(log N). Insert/delete/update are O(log N). This is the textbook answer.
Skip list — same O(log N) profile, simpler to implement concurrently, and it's exactly what Redis Sorted Sets (ZSET) use internally. ZADD (update), ZREVRANK (rank), ZREVRANGE (top-K / window) are O(log N) (ZADD/ZRANK) or O(log N + k) (range). This is the production answer.
A Fenwick/BIT (Binary Indexed Tree) over a bounded, discretized score domain gives O(log M) rank where M = number of distinct scores; great when scores are small integers, but awkward for arbitrary/continuous scores.

So: order-statistics tree / skip list conceptually, Redis ZSET in production. I'd say both — the interviewer wants the CS reasoning and the pragmatic build. The complexity table below makes the trade-offs explicit; the headline is that the augmented tree / skip list turns every operation into O(log N) or O(log N + k), which is what makes million-player boards feasible.

The classes (LLD)

I'll design with clean separation and OCP in mind (so a new leaderboard type — daily/weekly/all-time/per-region — is a new strategy, not an edit):

Player — id, name (value data).
Score — comparable value: primary = points, tie-breaker = earlier timestamp (so ties are deterministic, which matters for gaming fairness).
Leaderboard (interface) — submit, getRank, getTopK, getWindow. Program to this interface.
SortedSetLeaderboard (impl) — wraps the order-statistics structure (or a Redis client). Holds a Map<playerId, currentScore> for O(1) "current score" lookups and the ordered structure for ranking; the map is the source of truth for updates (compute delta, remove old, insert new).
TieBreaker (Strategy) — how ties resolve (timestamp, then id). Swappable.
LeaderboardService — façade: validation, concurrency control, choosing which board (daily/global) a submission goes to, exposing pagination.
ScoringPolicy (Strategy) — how a raw event becomes points (a kill = 10, an assist = 3), so new game modes plug in without editing the board.

Key invariant: update = remove-then-reinsert in the ordered structure keyed by (score, tieBreaker), and keep the Map in sync. That keeps getRank/getTopK correct after every change.

The rate-of-change alerting variant (same reasoning)

Rate-of-change alerting = "fire when a metric's slope over a window crosses a threshold" (e.g., error rate jumps 5x in 60s). Data-structure reasoning:

Maintain a sliding window of (timestamp, value) per metric in a deque/ring buffer; evict entries older than now - window in O(1) amortized from the front.
Keep running aggregates (sum, count, and for change: first vs last, or a regression slope) updated incrementally so rateOfChange() is O(1) per tick, not O(window).
AlertRule (Strategy) holds the threshold + comparator; AlertEngine evaluates rules on each update and debounces (fire once, then cool down) to avoid alert storms.
For many series at scale, a min-heap keyed by next-evaluation-time schedules which series to re-check, and a bucketed time series (e.g., 1s buckets) bounds memory.

So both problems reduce to: choose the structure whose cheap operations equal your hot query (ordered+rank for leaderboards; sliding-window+running-aggregate for alerting), then wrap it in strategy-based classes for extensibility.

Scale, concurrency & failure modes (deep-understanding differentiator)
Concurrency: millions of concurrent score updates → don't lock the whole board. Redis ZSET is single-threaded per shard (atomic ops, no app-side locks); in-process, use a lock striped by player or a concurrent skip list. Updates should be idempotent (dedupe by event id) so retries don't double-count.
Sharding / hot boards: a single global board of 100M players is fine in one ZSET for reads, but write-hot events (a viral match) may need to shard by region/segment and merge top-K across shards (a k-way merge of each shard's top-K). Approximate global rank via bucketed histograms when the exact rank of a low-ranked player isn't worth a cross-shard scan.
Time-windowed boards: daily/weekly boards need cheap reset — keep them as separate ZSETs keyed by date with a TTL, so "reset" is just a new key, not a delete loop.
Rank of far-down players is expensive at extreme scale — most systems show exact rank only for the top N and an approximate percentile ("top 5%") below that.
Durability: the ordered structure is a fast index, not the record of truth — persist raw score events (append-only log / DB) so the board can be rebuilt after a crash; Redis is the cache/index.
Ties & fairness: deterministic tie-break (timestamp then id) prevents flicker where equal-score players swap ranks on every read — critical when prizes depend on rank.
Summary — what a strong answer emphasizes
Lead with the operations and their complexity — that's how you justify the structure.
Pick an order-statistics tree / skip list (O(log N) update, rank, select) conceptually; Redis Sorted Set in production (ZADD/ZREVRANK/ZREVRANGE). Pair it with a Map for O(1) current-score.
Design clean classes: Leaderboard interface + SortedSetLeaderboard, with ScoringPolicy and TieBreaker as strategies so new modes/boards don't edit existing code.
Show the alerting variant is the same skill: sliding-window deque + incremental aggregates + AlertRule strategy + heap scheduler.
Close with scale/failure judgment: sharding + k-way top-K merge, per-date TTL boards, approximate rank for the long tail, idempotent updates, raw-event durability, deterministic tie-breaks.

Common follow-ups: "Why not sort on read?" (O(N log N) per query dies at scale). "How do you get rank without scanning?" (subtree-size augmentation / ZREVRANK). "Top-K across shards?" (each shard returns its top-K, merge with a size-K heap). "Concurrent updates?" (atomic ZSET ops or striped locks + idempotency).

**Comparison:**

```
Data structure	update	getRank	getTopK / window	Notes
HashMap<player,score>	O(1)	O(N log N) (sort)	O(N log N)	fine for update only; ranking dies at scale
Sorted ArrayList	O(N) (shift)	O(log N) (bin-search)	O(k)	reads cheap, writes brutal
Order-statistics BST (subtree size)	O(log N)	O(log N)	O(log N + k)	textbook answer, exact rank
Skip list (= Redis ZSET)	O(log N)	O(log N)	O(log N + k)	production answer, easy concurrency
Fenwick/BIT over discretized scores	O(log M)	O(log M)	O(log M · k)	only for bounded integer scores
Concern	Design choice
Millions of concurrent updates	Atomic Redis ZSET ops / per-player striped locks; idempotent by eventId
100M+ players, hot events	Shard by region; k-way merge each shard's top-K with a size-K heap
Daily/weekly reset	Separate ZSET per date + TTL (reset = new key, not delete loop)
Long-tail rank cost	Exact rank for top-N; approximate percentile below
Crash recovery	Rebuild board from append-only score-event log (Redis is index, not truth)
Rank flicker on ties	Deterministic tie-break: earlier timestamp, then playerId
```

**Code:**

```

// ---- Value types ---- record Player(String id, String name) {}

// Score is compared by points DESC, then earlier timestamp (deterministic ties). record Score(long points, long tsMillis) implements Comparable<Score> { public int compareTo(Score o) { if (points != o.points) return Long.compare(o.points, points); // higher first return Long.compare(tsMillis, o.tsMillis); // earlier wins ties } }

// ---- Extension seam: how a raw event becomes points ---- interface ScoringPolicy { long pointsFor(GameEvent e); } // Strategy — new mode = new class

// ---- Leaderboard abstraction (program to this) ---- interface Leaderboard { void submit(String playerId, long deltaPoints, long ts); long getRank(String playerId); // 1-based rank List<Entry> getTopK(int k); List<Entry> getWindow(int fromRank, int count); // "around me" pagination record Entry(String playerId, long points, long rank) {} }

// ---- In-memory impl: order-statistics via a TreeMap keyed by Score ---- // (Production: back this with Redis ZSET; TreeMap here shows the O(log N) shape.) class SortedSetLeaderboard implements Leaderboard { private final Map<String, Score> current = new HashMap<>(); // O(1) current score private final TreeMap<Score, String> ordered = new TreeMap<>(); // ordered index

CODE
Copy
public synchronized void submit(String playerId, long delta, long ts) {
    Score old = current.get(playerId);
    long base = (old == null) ? 0 : old.points();
    if (old != null) ordered.remove(old);                        // remove-then-reinsert
    Score s = new Score(base + delta, ts);
    current.put(playerId, s);
    ordered.put(s, playerId);
}

public synchronized long getRank(String playerId) {
    Score s = current.get(playerId);
    if (s == null) throw new NoSuchElementException(playerId);
    // headMap = everyone ranked strictly higher; +1 for 1-based rank.
    // (TreeMap.size() is O(n); a subtree-size-augmented tree / Redis ZREVRANK makes this O(log N).)
    return ordered.headMap(s).size() + 1L;
}

public synchronized List<Entry> getTopK(int k) {
    List<Entry> out = new ArrayList<>();
    long r = 1;
    for (var e : ordered.entrySet()) {
        if (r > k) break;
        out.add(new Entry(e.getValue(), e.getKey().points(), r++));
    }
    return out;
}

public synchronized List<Entry> getWindow(int fromRank, int count) {
    List<Entry> out = new ArrayList<>();
    long r = 1;
    for (var e : ordered.entrySet()) {
        if (r >= fromRank && out.size() < count)
            out.add(new Entry(e.getValue(), e.getKey().points(), r));
        r++;
        if (out.size() == count) break;
    }
    return out;
}

}

CODE
Copy

```java
// ---- Rate-of-change alerting: same skill, different structure ----
interface AlertRule { boolean triggered(double rateOfChange); } // Strategy: threshold/comparator

class RateWindow {                         // sliding window with incremental aggregate
    private final Deque<long[]> buf = new ArrayDeque<>(); // [ts, value]
    private final long windowMs;
    RateWindow(long windowMs){ this.windowMs = windowMs; }

    void add(long ts, long value){
        buf.addLast(new long[]{ts, value});
        long cutoff = ts - windowMs;
        while(!buf.isEmpty() && buf.peekFirst()[0] < cutoff) buf.pollFirst(); // O(1) amortized
    }
    // change per second across the window (first vs last)
    double rateOfChange(){
        if(buf.size() < 2) return 0;
        long[] a = buf.peekFirst(), b = buf.peekLast();
        double dt = (b[0]-a[0]) / 1000.0;
        return dt == 0 ? 0 : (b[1]-a[1]) / dt;
    }
}
TEXT
Copy
# Production leaderboard = Redis Sorted Set (skip list under the hood)
ZADD  board:daily:2026-07-07  1500  player:42     # add/update, O(log N)
ZINCRBY board:daily:2026-07-07  10   player:42     # atomic delta update
ZREVRANK board:daily:2026-07-07 player:42          # 0-based rank, O(log N)
ZREVRANGE board:daily:2026-07-07 0 9 WITHSCORES     # top-10, O(log N + k)
ZREVRANGE board:daily:2026-07-07 4270 4280 WITHSCORES # "around me" window
EXPIRE board:daily:2026-07-07 172800                # cheap reset via TTL
```

**Example input / output:**

```text
Dry run of `SortedSetLeaderboard` (ties broken by earlier timestamp):

submit(A, 100, t=1) -> ordered: [A:100]
submit(B, 150, t=2) -> ordered: [B:150, A:100]
submit(C, 100, t=3) -> ordered: [B:150, A:100(t1), C:100(t3)]   // A before C: earlier ts wins tie
submit(A, +60, t=4) -> remove A:100, insert A:160
                    -> ordered: [A:160, B:150, C:100]

getTopK(2)     -> [ {A,160,rank1}, {B,150,rank2} ]
getRank(C)     -> headMap(C:100) = {A,B} = 2 higher -> rank 3
getWindow(2,2) -> ranks 2..3 -> [ {B,150,rank2}, {C,100,rank3} ]

Redis equivalent:
ZINCRBY board 60 A ; ZREVRANK board C  -> returns 2 (0-based) => 3rd place. Matches.

Rate-of-change alerting dry run (window=60s, rule: rate > 5 err/s):
add(t=0s, errs=0); add(t=10s, errs=20); add(t=60s, errs=400)
rateOfChange = (400-0)/(60-0) = 6.67 err/s -> 6.67 > 5 -> AlertRule.triggered=true -> fire (then debounce/cooldown).
```

**Data:**

```

// Score submission event (append-only log = source of truth for rebuild) { "eventId": "evt-9f3a-0021", // dedupe key -> idempotent updates "playerId": "player:42", "board": "daily:2026-07-07", "deltaPoints": 10, "reason": "KILL", "tsMillis": 1751846400123 }

CODE
Copy

```json
// getWindow("player:42", count=5) response ("around me" page)
{
  "you": "player:42",
  "window": [
    {"rank": 4270, "playerId": "player:88", "points": 1520},
    {"rank": 4271, "playerId": "player:42", "points": 1500},
    {"rank": 4272, "playerId": "player:17", "points": 1495},
    {"rank": 4273, "playerId": "player:63", "points": 1490},
    {"rank": 4274, "playerId": "player:12", "points": 1488}
  ]
}
```

**Flow:**

1. Scale
2. Game event
3. ScoringPolicyevent -> points
4. LeaderboardServicevalidate, pick board,idempotency
5. Leaderboard iface
6. SortedSetLeaderboard
7. (Map: playerId -> ScoreO(1) current)
8. (Skip list / ZSETordered index O(log N))
9. getRank
10. getTopK / getWindow
11. Shard by region
12. k-way merge top-Ksize-K heap

