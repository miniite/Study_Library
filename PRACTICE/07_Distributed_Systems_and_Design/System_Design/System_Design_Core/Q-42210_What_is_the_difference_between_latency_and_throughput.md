---
type: "practice"
difficulty: "Basic"
---

# What is the difference between latency and throughput?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Queue vs Synchronous Calls](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Queue_vs_Synchronous_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define latency and throughput, explain how they differ, and show you understand that they are two distinct dimensions of performance that are related but not the same thing.

What the interviewer is probing for: Whether you can reason about performance precisely rather than lumping everything into "the system is fast/slow." A strong candidate names the correct units, gives a concrete example, and — this is the differentiator — explains how the two interact (they are not simply inverses of each other) and when to optimize for one over the other.

#### Assumptions
/ clarifying questions: None strictly needed for a definition, but a good candidate signals awareness: "Are we talking about a single request path, a batch pipeline, or an end-to-end user journey? Latency numbers only make sense with a percentile attached (p50 vs p99)."

Plan: Define each term with units → give a real-world analogy → give a concrete software example → explain the relationship and why they trade off → summarize when to optimize which.

The definitions

Latency is the time it takes for a single unit of work to complete — how long one request waits from start to finish. It is measured in units of time: milliseconds, microseconds, sometimes seconds. Example: "The API responds in 40 ms." Latency is about delay.

Throughput is the amount of work completed per unit of time — how many units of work the system finishes in a window. It is measured in units of work-per-time: requests per second (RPS/QPS), transactions per second (TPS), megabytes per second (MB/s), messages per second. Example: "The service handles 50,000 requests per second." Throughput is about rate / volume.

An analogy that makes it click

Think of a highway:

Latency = how long it takes one car to drive from city A to city B (say, 60 minutes).
Throughput = how many cars pass a point on the highway per hour (say, 2,000 cars/hour).

You can add lanes (more parallelism) so 4,000 cars/hour pass — that doubles throughput — without any single car arriving faster; each car still takes 60 minutes. That is the crux: you can raise throughput without lowering latency, and vice versa. They measure different things.

A concrete software example

Imagine a web service where each request takes 100 ms of processing.

If the server can only handle one request at a time (single-threaded, no concurrency), throughput is at most 10 requests/second (1000 ms / 100 ms), and latency is 100 ms.
Now run 10 worker threads in parallel. Latency for each individual request is still ~100 ms — the work per request didn't change — but throughput jumps to ~100 requests/second because ten are in flight at once.

This is why the two are independent knobs. Adding parallelism (more threads, more servers, more lanes) buys throughput. Making the actual work faster (a better algorithm, an index, caching) buys latency.

The relationship — and where they trade off

They are related through Little's Law: Concurrency = Throughput × Latency. For a fixed level of concurrency, if latency rises, throughput falls, and vice versa. This is why they feel like inverses, but the relationship only holds when concurrency is fixed.

Crucially, they often fight each other under load:

Batching raises throughput but increases latency: waiting to collect 100 items before writing them means item #1 sits in the buffer. Great for a data pipeline, bad for an interactive request.
Queuing under saturation kills latency: once arrival rate approaches capacity, requests pile up in the queue and tail latency (p99) explodes even though average throughput looks fine. This is the single most common production trap — the system's throughput is "OK" but latency has fallen off a cliff.

So a healthy answer stresses: measure latency at percentiles (p50, p95, p99), not as an average, because averages hide the tail that users actually feel.

When to optimize which
Latency-sensitive: interactive/user-facing paths — a search box, a payment confirmation, a page load. Users perceive delay directly. Here you optimize the single-request path (caching, indexes, fewer network hops, no synchronous batching).
Throughput-sensitive: background/bulk work — ETL jobs, log ingestion, video encoding, analytics. Nobody is watching a single item; you care about total volume per hour and cost efficiency. Here batching, parallelism, and buffering are wins.

Many real systems need both, with a stated SLO like: "p99 latency ≤ 200 ms at a sustained throughput of 20,000 RPS." That single line captures the whole trade-off: hold the tail latency bound while sustaining the rate.

Summary — what a strong answer emphasizes
Latency = time per one request (ms); throughput = requests completed per unit time (RPS). Different units, different meanings.
They are not simple inverses: you can raise throughput with parallelism without changing per-request latency, and improve latency without raising throughput.
Little's Law (Concurrency = Throughput × Latency) ties them together at fixed concurrency.
Report latency as percentiles, and remember queuing under saturation makes latency spike even when throughput looks acceptable.
Optimize latency for interactive paths, throughput for bulk/background work; real SLOs bound both.

#### Common mistakes
saying "throughput is just 1/latency" (false — that ignores parallelism); quoting average latency instead of p99; confusing a throughput problem (need more capacity) with a latency problem (need faster work).

#### Follow-up
probes an interviewer may ask: "If p99 latency is high but CPU is idle, what's happening?" (queuing/lock contention/slow downstream) and "How would you raise throughput without hurting latency?" (horizontal scaling, connection pooling, async I/O — add lanes rather than making cars slower).

**Comparison:**

```
Aspect	Latency	Throughput
Measures	Time for ONE unit of work	Units of work per unit time
Units	ms, µs, s	RPS/QPS, TPS, MB/s, msgs/s
Question it answers	"How long does one request take?"	"How much can the system do per second?"
Improved by	Faster work: caching, indexes, fewer hops, better algorithm	More parallelism: threads, servers, batching
Reported as	Percentiles (p50/p95/p99)	Sustained rate (avg over a window)
Optimize for	Interactive / user-facing paths	Bulk / background / pipeline work
Batching effect	Hurts it (adds wait)	Helps it (amortizes overhead)
```

**Example input / output:**

```text
Worked numeric example (single-server, 100 ms of work per request):

Scenario A — 1 worker (no concurrency)
  Input:  request processing time = 100 ms, concurrency = 1
  Output: latency ≈ 100 ms, throughput ≈ 10 req/s   (1000 ms / 100 ms)

Scenario B — 10 workers (parallel)
  Input:  request processing time = 100 ms, concurrency = 10
  Output: latency ≈ 100 ms (unchanged), throughput ≈ 100 req/s
  Takeaway: throughput 10x-ed, per-request latency unchanged.

Little's Law check:  Concurrency = Throughput × Latency
  Scenario B: 100 req/s × 0.100 s = 10 in-flight requests  ✓ (matches 10 workers)
```

**Flow:**

1. Throughput = requests / second
2. e.g. 50,000 RPS
3. Req
4. Server
5. Req
6. Req
7. Completed work per second
8. Latency = time for 1 request
9. 40 ms
10. Request in
11. Response out

