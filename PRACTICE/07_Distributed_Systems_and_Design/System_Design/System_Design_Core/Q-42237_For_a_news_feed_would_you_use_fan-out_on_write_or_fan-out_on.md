---
type: "practice"
difficulty: "Deep"
---

# For a news feed would you use fan-out on write or fan-out on read?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: When a user posts, how do we get that post into their followers' feeds — do we push it into every follower's feed at write time (fan-out on write), or do we assemble the feed by pulling from everyone the user follows at read time (fan-out on read)? The interviewer wants to hear that there is no single right answer — the correct senior answer is both, chosen per-user by a threshold, i.e. a hybrid. They're probing whether you understand the read/write trade-off, the "celebrity problem," fan-out amplification, and how you'd operate this at scale.

Clarifying questions I'd ask:

Read-heavy or write-heavy? Social feeds are overwhelmingly read-heavy (a post is written once, read by thousands; users refresh feeds constantly). Rule of thumb: reads outnumber writes ~100:1.
What's the follower distribution? Is there a long tail plus a few accounts with tens of millions of followers (celebrities)? Yes, almost always — this is the deciding factor.
Latency SLA for feed load? Typically feed must render in <200 ms; posting can tolerate seconds of async propagation.
Do we need real-time ("post appears instantly") or is eventual (a few seconds) fine? Eventual is fine for feeds.
Ranked feed (ML-scored) or reverse-chronological? Affects whether the materialized feed stores post IDs or scored entries.

Plan: define both strategies from first principles, work the math on why pure fan-out-on-write breaks for celebrities and why pure fan-out-on-read breaks for normal reads, then land on the hybrid that every large social feed (Twitter/X, Instagram, LinkedIn) actually uses.

Fan-out on write (push model)

When user A posts, we immediately push the post ID into a precomputed feed list for each of A's followers, usually stored in a fast store (Redis sorted set per user, keyed by follower ID, scored by timestamp/rank). Reading a feed is then trivial: ZREVRANGE feed:{userId} 0 50 — one O(log N) lookup, no joins.

Read cost: tiny and constant. This is why it dominates: feeds are read constantly.
Write cost: one post triggers F writes, where F = follower count. If Ariana Grande (≈380M followers) posts, that's hundreds of millions of writes for a single tweet — a "fan-out storm" that saturates the write path and delays propagation for minutes.
Storage cost: every post is duplicated into every follower's list. High but cheap (IDs, not content).
Fan-out on read (pull model)

We store each user's posts once in an author-partitioned "posts" table. When user B loads their feed, we query the recent posts of everyone B follows, then merge-sort and rank them at request time.

Write cost: trivial — one insert into the author's timeline.
Read cost: expensive and grows with following count. If B follows 2,000 accounts, that's a scatter-gather across 2,000 timelines, merged and ranked on every refresh — hard to keep under 200 ms, and it hammers the DB because reads are frequent.
The core trade-off

You're trading write amplification (push) against read amplification (pull). Because feeds are read ~100× more than written, the default instinct is push (precompute so reads are cheap). The exception is the celebrity / high-fan-out account, where push cost explodes.

The answer: hybrid fan-out

Do fan-out on write for "normal" authors (below a follower threshold, e.g. <10k–100k followers) and fan-out on read for celebrities (above the threshold).

At feed-read time for user B:

Read B's precomputed feed (contains posts fan-out-on-write from all the normal accounts B follows) — one cheap lookup.
Separately, pull recent posts from the small set of celebrities B follows (fan-out on read — cheap because B follows only a handful of them).
Merge and rank the two sources, return top N.

This bounds both costs: no single write triggers hundreds of millions of pushes (celebrities are pulled), and no read has to scatter-gather across thousands of timelines (only across the few celebrities). It's exactly the "Twitter used a hybrid" answer that senior interviewers look for.

Operational details a strong answer adds
Async fan-out via a queue. On post, enqueue a fan-out job (Kafka/SQS); workers write to follower feeds in batches. Posting returns immediately; propagation is eventual (seconds). This decouples the spike.
Threshold, not a hard flag. Reclassify accounts as their follower count crosses the threshold. Some designs also skip fan-out to inactive followers (no login in 30 days) and backfill their feed on next login (fan-out-on-read for the dormant tail) — a huge write saving.
Feed store: Redis sorted sets (post_id → score) capped to ~800 entries per user; source of truth stays in the DB so a feed can be rebuilt/backfilled.
Ranking: the materialized list can hold model scores, or store IDs and re-rank at read time with a lightweight ranker; celebrity posts get ranked in during the merge.

#### Cons
istency: eventual is acceptable; a post appearing a few seconds later in followers' feeds is fine. Deletes/edits are handled by tombstones or by filtering at read/hydration time.
Hydration: feeds store IDs only; actual post content, author, counts are hydrated from caches at read time so edits/deletes/count-updates aren't stale.
Summary — what a strong answer emphasizes

Lead with: "Neither alone — a hybrid, because feeds are read-heavy so you want push, but pure push has the celebrity fan-out problem, so you pull for high-fan-out accounts and merge." Show the math (100M-follower post = 100M writes) to justify the threshold. Mention async queue-based fan-out, capping feed size, skipping inactive users, and merging at read time. Common mistakes: committing to one model absolutely; ignoring the celebrity problem; forgetting feeds are read-heavy (which is what makes push the default); doing fan-out synchronously on the post request; and storing full content instead of IDs in the feed.

#### Follow-up
probes to be ready for: How do you pick the threshold? (measure write cost vs read latency; tune empirically). How do you handle a normal account that suddenly goes viral? (dynamic reclassification). How do deletes work? (tombstone + hydration-time filter). How do you rebuild a corrupted feed? (replay from author timelines in the DB).

**Comparison:**

```
Dimension	Fan-out on WRITE (push)	Fan-out on READ (pull)	Hybrid (chosen)
Feed read cost	O(1), precomputed — cheap	O(following) scatter-gather — expensive	Cheap: 1 feed read + few celeb pulls
Write/post cost	O(followers) — explodes for celebrities	O(1) — trivial	Bounded: push for normals, skip celebs
Storage	High (post duplicated per follower)	Low (single copy)	Moderate
Feed latency	Lowest, most consistent	High, variable	Low
Weak spot	Celebrity fan-out storm	Users who follow thousands	None dominant
Freshness	Eventual (async push)	Real-time on read	Eventual + real-time for celebs
Best when	Most accounts (feeds are read-heavy)	Very high-fan-out authors	Real systems (Twitter/IG/LinkedIn)
```

**Code:**

```python
# Fan-out on WRITE: worker consumes post events, pushes into follower feeds.
FANOUT_THRESHOLD = 100_000  # accounts above this are pulled, not pushed

def on_new_post(post_id, author_id, ts, follower_count):
    if follower_count >= FANOUT_THRESHOLD:
        return  # celebrity: skip push; readers pull these at read time
    # push in batches to avoid a single huge transaction
    for batch in iter_follower_batches(author_id, size=1000):
        pipe = redis.pipeline()
        for follower_id in batch:
            if is_active(follower_id):            # skip dormant followers
                pipe.zadd(f"feed:{follower_id}", {post_id: ts})
                pipe.zremrangebyrank(f"feed:{follower_id}", 0, -801)  # cap ~800
        pipe.execute()

# READ path: precomputed feed  +  pulled celebrity posts  ->  merge/rank
def get_feed(user_id, limit=50):
    pushed = redis.zrevrange(f"feed:{user_id}", 0, 400, withscores=True)
    celeb_ids = get_followed_celebrities(user_id)          # small set
    pulled = []
    for cid in celeb_ids:
        pulled += db.recent_posts(cid, since="24h")        # fan-out on read
    merged = merge_and_rank(pushed + [(p.id, p.score) for p in pulled])
    ids = [pid for pid, _ in merged[:limit]]
    return hydrate(ids)   # fetch content/author/counts from cache
```

**Example input / output:**

```text
Case A — normal user (5,300 followers) posts:
  write path : 1 event -> ~5,300 ZADDs (batched, async)  ~= 5.3k writes
  follower read: ZREVRANGE feed:{id} 0 50  -> O(log N), ~1 ms

Case B — celebrity (100M followers) posts:
  write path : SKIPPED (above threshold) -> 1 insert into author timeline
  follower read: 1 pull of celeb's recent posts, merged into precomputed feed

Feed load for user B who follows 1,998 normal + 2 celebrities:
  1) read precomputed feed (has all 1,998 normals' posts)   1 lookup
  2) pull recent posts of the 2 celebrities                 2 lookups
  3) merge + rank + hydrate top 50                           -> <200 ms
```

**Data:**

```text
# Redis materialized feed (fan-out-on-write target) — one sorted set per user
KEY   feed:{userId}          # ZSET, capped to ~800 entries
MEMBER post_id
SCORE  timestamp_or_rank

# Author timeline (source of truth, fan-out-on-read source for celebrities)
posts(post_id PK, author_id, created_at, body_ref)   # partitioned by author_id
INDEX (author_id, created_at DESC)

# Post-created event on the fan-out queue (Kafka/SQS)
{ "post_id": 991, "author_id": 42, "ts": 1720300000, "follower_count": 5300 }
```

**Flow:**

1. No (normal)
2. Yes (celebrity)
3. User posts
4. follower_count >= threshold?
5. Fan-out on WRITE:async queue -> push post_idinto each active follower's feedZSET
6. Skip push:store only in author timeline
7. Follower opens feed
8. Read precomputed feed ZSET(all normal authors)
9. Pull recent posts offollowed celebrities (read)
10. Merge + rank
11. Hydratecontent/author/counts
12. Return top N

