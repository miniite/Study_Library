---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/04_Distributed_Failures/Network_Faults_and_Timeouts"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Network Faults and Timeouts

**L4 Advanced** · [↑ Faults and Failure in Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Packet-switched networks give no upper bound on delay or guarantee of delivery, so a timeout is the only way to detect a fault — and there is no universally correct timeout value.

### 1. The big picture
Distributed systems here are *shared-nothing*: machines communicate only by sending messages over a network. The network is the one thing you can't make fully reliable, so every remote call has four unhappy outcomes that look identical to the caller: the request was lost, it is queued somewhere, the remote node has failed or paused, or the response was lost or delayed. All you learn is "no answer yet".

This lesson explains why that ambiguity is fundamental (queueing, packet switching, shared resources), why timeouts are therefore a judgment call, and how the synchronous/asynchronous/partially-synchronous models describe it. It feeds Failure Detection, Fencing and System Models and leader election.

### 2. Core concepts and vocabulary
- **Shared-nothing** — nodes share no memory or disk; the network is the only link.
- **Asynchronous network** — delivers packets as fast as it can but with no maximum delay.
- **Synchronous network** — guarantees a bounded delay (e.g. a telephone circuit).
- **Bounded vs unbounded delay** — a known upper limit on latency vs none.
- **Packet switching** — packets share links dynamically (Ethernet, IP); causes queueing.
- **Circuit switching** — a fixed slice of bandwidth is reserved end-to-end for a call.
- **Network congestion** — many senders target one link/port; switch queues grow, then drop packets.
- **Flow control / backpressure** — TCP limiting its own sending rate to avoid overloading a link or receiver.
- **Retransmission** — TCP resends unacknowledged packets after a timeout; the app sees only added delay.
- **Network partition (netsplit)** — part of the network is cut off from the rest.
- **Round-trip time (RTT)** — time from request sent to response received.
- **QoS / admission control** — prioritizing packets / rate-limiting senders to emulate bounded delay.
- **Partially synchronous** — usually well-behaved, occasionally exceeds any bound.

### 3. How it works, step by step
**Why delay is variable (the sources of queueing)**
1. Several nodes send to the same destination at once; the switch queues packets and feeds them to the link one at a time. If the queue overflows, packets are dropped and must be resent.
2. The packet arrives, but all CPU cores are busy, so the OS queues the request until the application is ready.
3. In a virtualized host, the VM may be paused for tens of milliseconds while another VM runs; inbound data is buffered by the hypervisor.
4. TCP's flow control adds queueing at the *sender* before data even enters the network.
5. A lost packet is retransmitted after TCP's timeout (derived from observed RTT); the application sees no loss, just a latency spike.

![Several machines sending to one destination fill its switch queue](<../../../../_assets/Designing_Data_Intensive_Applications_The_Big_Ideas_Behind_R/ddia_0802.png>)

**Choosing a timeout**
1. In a hypothetical network with max packet delay *d* and max processing time *r*, a response must arrive within `2d + r`; silence beyond that proves a fault.
2. Real asynchronous networks have neither *d* nor *r*, so no such guarantee exists.
3. Therefore pick the timeout experimentally from measured RTT distributions (e.g. a high percentile plus margin) and revisit as conditions change.
4. A long timeout = users wait / errors are slow; a short timeout = premature "dead" declarations.

**Why not just make the network reliable?**
- Telephone circuits reserve bandwidth, so no queueing and fixed latency — but that wastes capacity for bursty traffic (web pages, files) and requires guessing the rate in advance.
- Packet switching is optimized for *bursty traffic*: TCP takes whatever bandwidth is available. The price is queueing and unbounded delay.
- Variable delay is a **cost/benefit trade-off, not a law of nature**: statically partitioning resources gives latency guarantees at the cost of utilization; multi-tenancy gives better utilization and cheaper hardware but variable delays. Public clouds and the internet don't offer bounded-latency guarantees.

### 4. Worked example
Suppose p50 RTT is 1 ms but occasionally 800 ms under load. Measure and set a timeout:

```bash
# sample RTT distribution to the peer
ping -c 200 10.0.0.12 | tail -n 3
# application-level: histogram of request latency, look at p99 / p99.9
```

```python
# client with a bounded wait and capped, jittered retry
import random, time, requests
def call(url, timeout=0.5, retries=3):
    for i in range(retries):
        try:
            return requests.post(url, json=payload, timeout=timeout)
        except requests.Timeout:
            time.sleep(min(2**i * 0.1, 2) * random.uniform(0.5, 1.5))
    raise
```

What the caller can conclude after a `Timeout`: nothing about whether the server processed the request. If it was a payment or an email, a blind retry may do it twice — so the operation needs an **idempotency key** or the receiver must deduplicate. This is why timeouts and idempotence always travel together.

### 5. Under the hood
- **Measured reality.** Studies of datacenters report on the order of a dozen network faults a month in a medium-sized facility, about half isolating a single machine and half an entire rack; redundant gear does not remove them because misconfiguration (human error) is a leading cause. A switch software upgrade has delayed packets for over a minute; some NICs drop all inbound packets while sending outbound fine, so a link can work in one direction only.
- **Adaptive timeouts.** Instead of a constant, measure jitter and RTT over time and adjust automatically (the idea behind phi-accrual in Failure Detection).
- **TCP vs UDP.** UDP skips flow control and retransmission, trading reliability for lower delay variance — useful for VoIP/video where late data is worthless; still subject to switch queues and scheduling delay.
- **Synchrony models.** Asynchronous: no timing assumptions, consensus impossible in general (FLP). Synchronous: bounded delays and clock drift make timeouts reliable, but real failures violate the bounds and algorithms built on it (leader election, failure detection) can then misfire — e.g., two nodes both believing they lead, or a live node declared dead. Partially synchronous: bounds hold "most of the time" — the practical sweet spot.
- **InfiniBand / ATM** are hybrids with link-level flow control that reduce queueing but don't eliminate congestion delays.

### 6. Performance and design implications
- Tail latency is dominated by queueing, not by distance; measure p99 and p99.9, not the mean.
- Premature timeouts amplify load: retries plus failover move work onto already stressed nodes, possibly cascading to a full outage. Use exponential backoff with jitter, retry budgets, and circuit breakers.
- Keep payloads and fan-out small; every extra synchronous hop adds another independent chance to hit a delay spike.
- Always carry deadlines through the call chain so downstream work is abandoned when the caller has given up.
- Test with fault injection (latency, loss, partitions) — happy-path testing hides these issues.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ping`, `mtr`, `traceroute` | Measure RTT, loss and per-hop delay | First look at suspected network trouble |
| `ss -ti`, `netstat -s` | TCP retransmits, RTT, congestion window | Diagnosing retransmission-driven latency |
| `tc qdisc add dev eth0 root netem delay 200ms loss 1%` | Inject latency/loss on Linux | Chaos tests of timeouts and retries |
| Connect/read/write timeouts in clients | Bound each phase of a call | Every RPC/HTTP/JDBC client |
| Retry policy with backoff + jitter | Spread retries over time | Preventing retry storms |
| Circuit breaker / bulkhead | Stop calling a failing dependency | Avoiding cascading failure |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Short timeouts — latency-sensitive, many healthy replicas to fail over to | Service already near saturation (premature failover increases load) |
| Long timeouts — expensive or non-idempotent work, heavy tail latency | Users or upstream callers have tight deadlines |
| Adaptive timeouts | You need deterministic, auditable behavior |
| UDP-style best-effort | Data must arrive complete and in order |
| Statically reserved bandwidth / QoS | Cost matters more than latency guarantees |

### 9. Common mistakes
- No timeout at all → threads hang forever → resource exhaustion → always set connect and read timeouts.
- Retrying non-idempotent operations → duplicate side effects → idempotency keys or dedupe.
- Assuming a timed-out request failed → it may have executed → reconcile by querying state.
- One fixed timeout everywhere → false positives across regions → per-dependency, measured values.
- Synchronized retries without jitter → thundering herd → backoff + jitter.
- Assuming the network is symmetric → one-way failures confuse detectors → probe both directions.

### 10. 🎯 Interview kit
**30-second answer:** In an asynchronous packet-switched network there's no bound on delay, so when a response doesn't arrive you can't tell whether the request was lost, the node died, or the reply is late. A timeout is the only detector, and picking it is a trade-off between slow failure detection and false positives that cause duplicate work and cascading load.
**Structure a longer answer:** the four ambiguous outcomes → causes of variable delay (queueing, VM pauses, TCP) → why we don't use circuits (bursty traffic, utilization) → timeout selection and adaptivity → retries/idempotence/backoff → partial synchrony model.
**Likely follow-ups:**
- *What's the difference between synchronous and asynchronous networks?* → bounded vs unbounded delay; circuit vs packet switching.
- *How do you pick a timeout?* → measure RTT percentiles, add margin, adapt; no analytic answer.
- *Why can retries be dangerous?* → duplicates and retry storms.
- *Is UDP faster?* → less delay variance, no reliability.
**Signals interviewers look for:** you tie timeouts to idempotency and cascading failures, and you understand the cost/utilization reason behind variable delay.

### 11. Check yourself
- [ ] List four indistinguishable reasons for "no response" and what each implies.
- [ ] Explain why `2d + r` isn't usable as a timeout on the internet.
- [ ] Why does packet switching win over circuits for most workloads?
- [ ] Describe how a premature timeout can cascade into an outage.
- [ ] Design a client retry policy that is safe for a payment API.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks › Timeouts and Unbounded Delays
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks › Can we not simply make network delays predictable?
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks › Network congestion and queueing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Glossary
- *Database Internals* — Alex Petrov — Introduction and Overview › System Synchrony
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks › Network Faults in Practice

