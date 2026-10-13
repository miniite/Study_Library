---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/01_Data_System_Fundamentals/Load_and_Performance_Metrics"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Describing Load and Performance

**L4 Advanced** · [↑ Reliability Scalability Maintainability](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Describe performance as a distribution of response times (median plus high percentiles), not as an average, and measure it from the client side under realistic, independent load.

### 1. The big picture
After you have described the load on a system, you ask what happens to performance when load grows. Performance has to be put in numbers first. Batch systems (Hadoop, Spark jobs) care about **throughput**: records per second, or total job time for a dataset of a given size. Online systems care about **response time**: the time between a client sending a request and receiving the answer. This lesson explains why response time must be treated as a distribution, why tail percentiles matter, and how to compute and test them correctly.

### 2. Core concepts and vocabulary
- **Throughput** — units of work completed per unit time.
- **Service time** — the time actually spent processing the request.
- **Response time** — what the client sees: service time + network delay + queueing delay.
- **Latency** — strictly the time a request waits to be handled (it is "latent"); often used loosely as a synonym for response time.
- **Percentile (pXX)** — the value below which XX% of requests fall. **Median / p50** is half-way; **p95, p99, p999** describe the slow tail.
- **Tail latency** — the high-percentile response times that the slowest users experience.
- **SLO / SLA** — service level objective / agreement: targets such as "median < 200 ms, p99 < 1 s, 99.9% uptime".
- **Head-of-line blocking** — a few slow requests occupy the limited parallel slots, holding up fast requests queued behind them.
- **Tail latency amplification** — when one user request needs many backend calls, the slowest call dominates.

### 3. How it works, step by step
1. **Collect response times** for every request for a window (e.g., 10 minutes), measured on the **client side** so queueing is included.
2. **Sort or bucket** them; read off p50, p95, p99, p999.
3. **Compare against targets** (SLOs) and look at how the tail moves as load changes.
4. **Load test correctly**: the generator must send requests at a fixed independent rate and *not* wait for the previous response, otherwise queues are artificially short and numbers look better than reality.
5. **Aggregate correctly** across machines or time: merge histograms, never average percentiles.

Why the mean misleads: it does not say how many users saw a given delay. A service with a 100 ms median and a 3 s p99 has a perfectly respectable mean that hides the 1-in-100 awful experience.

### 4. Worked example
```python
# naive percentile calc over a window of latencies in ms
import numpy as np
lat = np.array([...])           # all request times in the last 10 minutes
print("mean", lat.mean())
print("p50 ", np.percentile(lat, 50))
print("p99 ", np.percentile(lat, 99))
print("p999", np.percentile(lat, 99.9))
```
Suppose a page needs 10 parallel backend calls and each backend has p99 = 1 s. The probability that *all ten* are faster than p99 is 0.99^10 ≈ 90%, so about **10% of page loads** will see at least one slow call. A backend p99 becomes the page's p90. That is tail latency amplification, and it is why backend teams are held to p99.9 targets.

### 5. Under the hood
- **Sources of variance** even for identical requests: context switches, TCP retransmits after packet loss, garbage-collection pauses, page faults forcing a disk read, noisy neighbours, rack vibrations.
- **Queueing** explains most of the high-percentile behaviour: as utilization approaches 100%, queue length (and therefore waiting time) grows super-linearly. A server with N cores serves only N things in parallel, so a few slow requests block many fast ones.
- **Efficient percentile computation** for dashboards: keeping all values and sorting is O(n log n) per window. Streaming approximations (forward decay, t-digest, HdrHistogram) give accurate percentiles with fixed memory. These structures are *mergeable*, which is how multi-machine percentiles are computed correctly; averaging per-machine p99 values is mathematically meaningless.
- **Diminishing returns:** pushing the p99.99 down is expensive because random events outside your control dominate; Amazon reportedly targets p99.9 for internal services (its slowest customers are often its most valuable) and judged p99.99 not worth the cost.

### 6. Performance and design implications
- Track p50 (typical), p99/p999 (worst typical), and throughput separately; plot them against load to find the knee of the curve.
- Business impact is measurable: a 100 ms slowdown was reported to cost ~1% of sales; a 1 s slowdown a 16% drop in a satisfaction metric.
- Mitigations for tails: reduce fan-out per request, hedged or backup requests, timeouts with retries, load shedding, bulkheads, isolating slow tenants, avoiding GC pauses.
- Benchmarks must compare like with like. Vendor "benchmark wars" often compare systems designed for different workloads or use unrealistic test scenarios; test with your own data shape and concurrency.

### 7. Commands and settings you should know
| Tool / concept | What it does | When to use |
|---|---|---|
| Histograms (HdrHistogram, Prometheus buckets) | Store latency distributions compactly and mergeable | Service metrics across many instances |
| t-digest | Approximate quantiles with small memory | Streaming percentile estimation |
| Open-loop load generators (wrk2, k6 arrival-rate, Gatling) | Send requests at fixed rate regardless of responses | Honest capacity testing |
| Distributed tracing | Per-request breakdown across services | Finding which backend creates the tail |
| SLO dashboards / error budgets | Track the target and the remaining slack | Governing release pace |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Percentiles (p50/p99) for user-facing latency | Using the mean as the headline number |
| Exact quantiles on small windows | High-volume streams (use sketches) |
| Client-side measurement | Server-side only; it hides queueing and network |
| Closed-loop testing for simple regression checks | Capacity planning (use open-loop) |

### 9. Common mistakes
- Reporting averages → tail users invisible → report percentiles.
- Averaging percentiles across hosts → wrong number → merge histograms.
- Closed-loop load test that waits for each response → coordinated omission, flattering results → open-loop generator.
- Measuring only at the server → misses queueing/network → measure at the client or edge.
- Optimizing for p99.99 first → huge cost, small gain → fix p50/p99 and fan-out first.

### 10. 🎯 Interview kit
**30-second answer:** Response time is a distribution. Use percentiles: median for the typical case, p99 and p999 for the tail, because slow requests hit your most engaged users and are amplified when one request fans out into many backend calls. Measure on the client and aggregate with histograms.

**Structure a longer answer:** define throughput vs response time → why mean fails → percentile selection and SLOs → tail amplification and head-of-line blocking → correct measurement/load testing → mitigation options.

**Likely follow-ups:**
- *Latency vs response time?* → latency is waiting time; response time adds service time and network.
- *How do you combine p99 from 20 servers?* → you cannot average; merge the underlying histograms.
- *Why does the tail grow near saturation?* → queueing delay rises non-linearly with utilization.

**Signals interviewers look for:** percentile fluency, awareness of coordinated omission and amplification, and a measurement-first mindset.

### 11. Check yourself
- [ ] Explain why p50 of 200 ms with p99 of 3 s is not "fine".
- [ ] Compute the chance that a page with 20 backend calls sees a call slower than p99.
- [ ] Explain head-of-line blocking and why client-side measurement matters.
- [ ] Describe why you cannot average percentiles.
- [ ] Design an honest load test for an API.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Latency and response time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Describing Performance
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Optimization, Performance, and the Benchmark Wars

