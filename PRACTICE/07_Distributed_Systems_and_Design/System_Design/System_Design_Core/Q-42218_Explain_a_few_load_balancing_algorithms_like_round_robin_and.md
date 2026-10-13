---
type: "practice"
difficulty: "Intermediate"
---

# Explain a few load balancing algorithms like round robin and least connections.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Load Balancers](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Load_Balancers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

This is intermediate, so the interviewer isn't satisfied by "round robin cycles through servers." They want to see that you understand how each algorithm actually decides, what workload each is good/bad for, and the trade-offs — especially the failure mode where a simple algorithm sends traffic to an overloaded or heterogeneous server. The strongest answer walks through the major algorithms, explains the mechanics and the scenario each one fixes, and closes with "how I'd actually choose." Let me build up from the simplest to the smartest.

The job every algorithm is doing

A load balancer has a pool of healthy backends and, for each incoming request/connection, must pick one. The algorithm is the selection policy. Two broad families:

Static algorithms decide without looking at live server state (round robin, weighted round robin, IP hash). Cheap, predictable.
Dynamic algorithms use live signals — current connections, response times — to adapt (least connections, least response time, least load). Smarter, slightly costlier.
1. Round Robin

Cycle through servers in order: A, B, C, A, B, C… Each server gets an equal share of requests over time.

#### Pros
dead simple, no state, perfectly even count distribution.

#### Cons
/ failure mode: it assumes all requests cost the same and all servers are equal. If one request is a cheap health-check and the next is a heavy report generation, round robin can pile heavy requests on one server while another idles. It also ignores that server C might be a smaller/older box. Great for uniform, stateless, short requests (e.g., a fleet of identical stateless API pods).
2. Weighted Round Robin

Same idea, but each server has a weight proportional to its capacity. A server with weight 3 receives 3 requests for every 1 that a weight-1 server gets.

When: heterogeneous fleets — mixed instance sizes, or gradually shifting traffic to a new server pool. Weights let you send more work to the beefier machines.

#### Cons
weights are static/manual; they don't react to live load, only to configured capacity.
3. Least Connections

Send the next request to whichever server currently has the fewest active connections. This is dynamic — it reacts to real-time load.

The problem it fixes: the round-robin failure mode. If some requests are long-lived (a slow query, a big download, a websocket), those connections pile up on their servers; least-connections naturally steers new work to the servers that have drained. It self-balances when request costs vary.
When: requests have variable or long durations — DB-heavy endpoints, long-polling, streaming. This is a very common production default.

#### Cons
"fewest connections" is a proxy for load, not load itself — a server with 2 expensive connections may be busier than one with 5 cheap ones. And a freshly-added (empty) server can get flooded because it momentarily has the fewest connections. Weighted least connections (connections ÷ weight) mixes in capacity to handle heterogeneous fleets.
4. Least Response Time

Pick the server with the lowest average response time (often combined with fewest connections). This directly optimizes for what users feel — latency — and catches a server that's technically up but slow (GC pause, hot neighbor, degraded disk). Costs more monitoring but is the most user-centric dynamic policy.

5. IP Hash / Consistent Hashing

Hash a key from the request (client IP, or a session/user ID) to deterministically map it to a server. The same client always lands on the same server.

When: you need session affinity / stickiness (server-side session in memory) or cache locality (route a user/tenant to the node that has their data cached, boosting hit rate).
Plain modulo hashing (hash % N) has a fatal flaw: when N changes (add/remove a server), almost every key remaps — cache-wipe / session-loss storm. Consistent hashing solves this: adding/removing a node remaps only ~1/N of keys. This is the algorithm behind sharded caches (Memcached, Redis Cluster) and CDNs.
6. Random / Power-of-Two-Choices

Pick a server at random. Naive random is surprisingly decent at scale but can create imbalance. Power of two choices — pick two servers at random, send to the less loaded of the two — is a famous result: nearly as good as full least-connections but with far less coordination/state, so it scales beautifully across many load balancers. Worth naming to show depth.

How I actually choose (the senior signal)
Identical servers, short uniform requests → Round Robin (simple, predictable).
Different-sized servers → Weighted (RR or least-conn).
Variable / long request durations → Least Connections (or weighted least-conn) — my most common default.
Latency-sensitive, want to dodge slow-but-alive nodes → Least Response Time.
Need stickiness or cache locality → IP/consistent hash.
Many independent LB nodes, huge fleet → Power of two choices.

And two cross-cutting points that separate an intermediate answer: (1) every algorithm operates only over servers that pass health checks — a dead server is removed regardless of policy; and (2) if you use stickiness, plan for what happens when that server dies (session must be recoverable — externalize to Redis — or the user is logged out).

Summary — what a strong answer emphasizes
Round robin = equal turns, simple, assumes uniform requests/servers (its weakness).
Weighted = round robin adjusted for unequal server capacity.
Least connections = dynamic, routes to the least-busy server; fixes round robin's variable-cost weakness; watch the "new empty server flood" and use weighted variant for heterogeneous fleets.
Round out with least-response-time (latency), IP/consistent hashing (stickiness + cache locality, and why consistent hashing beats modulo), and power-of-two-choices.
Close by mapping algorithm → workload, and note health checks + session handling apply on top of all of them.

#### Common mistakes
/ follow-ups: describing least connections as "least load" (it's connection count, a proxy); forgetting the cold-start flooding problem; using modulo hashing and not knowing why it's bad; not tying the choice to request-duration variance. Likely probe: "A new server was added and immediately got slammed — why?" → least-connections saw it with zero connections; fix with slow-start / warm-up ramping.

**Comparison:**

```
Algorithm	How it picks	Type	Best for	Main weakness
Round Robin	next in cycle	static	identical servers, uniform short requests	ignores request cost & server size
Weighted RR	cycle by weight	static	heterogeneous server sizes	weights are manual, not live
Least Connections	fewest active conns	dynamic	variable/long request durations	conn count ≠ true load; new-server flood
Weighted Least Conn	conns ÷ weight	dynamic	variable load + mixed sizes	needs tuned weights
Least Response Time	lowest latency	dynamic	latency-sensitive, dodge slow nodes	more monitoring overhead
IP / Consistent Hash	hash(key)→server	static	session stickiness, cache locality	rebalancing on node change (use consistent hashing)
Power of Two Choices	2 random, pick lighter	dynamic	huge fleets, many LBs	slightly worse than full least-conn
```

**Code:**

```
Round robin — cycle through backends

class RoundRobin: def init(self, servers): self.servers = servers self.i = -1 def pick(self): self.i = (self.i + 1) % len(self.servers) return self.servers[self.i]

Weighted round robin — expand by weight (server, weight)

class WeightedRR: def init(self, weighted): # [("A",3),("B",1)] self.pool = [s for s, w in weighted for _ in range(w)] self.i = -1 def pick(self): self.i = (self.i + 1) % len(self.pool) return self.pool[self.i]

Least connections — route to the server with fewest active conns

class LeastConnections: def init(self, servers): self.active = {s: 0 for s in servers} # live connection counts def acquire(self): s = min(self.active, key=self.active.get) self.active[s] += 1 return s def release(self, s): self.active[s] -= 1

CODE
Copy

```nginx
# Nginx: least-connections with weights (heterogeneous fleet)
upstream backend {
    least_conn;
    server 10.0.0.11 weight=3;   # bigger box gets ~3x
    server 10.0.0.12 weight=1;
    server 10.0.0.13 slow_start=30s;   # ramp new server to avoid flooding
}
```

**Example input / output:**

```text
ROUND ROBIN (A,B,C):
  req1->A  req2->B  req3->C  req4->A  req5->B ...   (equal turns, ignores cost)

WEIGHTED RR  A:3  B:1  ->  pool = [A,A,A,B]:
  req1->A  req2->A  req3->A  req4->B  req5->A ...

LEAST CONNECTIONS — active counts start {A:0,B:0,C:0}:
  req1 -> A (all tie, pick A)          active {A:1,B:0,C:0}
  req2 -> B (fewest)                   active {A:1,B:1,C:0}
  req3 -> C                            active {A:1,B:1,C:1}
  A's request is SLOW and still open; B and C finish -> {A:1,B:0,C:0}
  req4 -> B (A is busy, correctly skipped)   {A:1,B:1,C:0}
  ^ round robin would have sent req4 to A and overloaded it
```

**Flow:**

1. uniform reqs, equal servers
2. unequal server sizes
3. variable/long durations
4. latency critical
5. need stickiness / cachelocality
6. Incoming request
7. Server healthy?filter by health check
8. Choose algorithm
9. Round Robin
10. Weighted RR / Weighted LC
11. Least Connections
12. Least Response Time
13. IP / Consistent Hash
14. Forward to chosen server

