---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd achieve zero-downtime cutover of a public DNS name from an old load balancer to a new one, accounting for DNS TTL and in-flight connections.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Limits and Throttling](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Serverless_Limits_and_Throttling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To achieve a zero-downtime cutover of a public DNS name from an old load balancer (OLB) to a new one (NLB), we need to carefully manage the DNS TTL, handle in-flight connections, and employ a phased approach. This process involves strategic planning and coordination across both DNS and network layers.

#### Triage

The core challenge is ensuring that no user requests are lost or experience downtime during the transition period when users may still be hitting the old load balancer even as the new one becomes active. We need to ensure that any in-flight connections are handled correctly, and the DNS TTL is managed so that it does not create unnecessary delays.

#### Assumptions
Both OLB and NLB are operational.
The DNS system supports zone transfers or zone apex updates.
There are no critical dependencies on the old load balancer during the transition.
The application layer can handle a brief period of redundancy or potential traffic spikes.
Step-by-Step Solution

DNS Configuration Update

Set up a new DNS record pointing to the NLB with a very short TTL (e.g., 5 seconds). This ensures that any DNS caches will start updating quickly.
Monitor the DNS propagation and ensure it is within acceptable limits.

Phased Rollout of Traffic

Initially, route a small percentage (1-5%) of traffic to the NLB via a network-level load balancer or by using an A/B testing approach.
Monitor the new setup for issues such as response times, errors, and resource utilization on both OLB and NLB.

Health Check

Implement health checks on the NLB to ensure that it is functioning correctly under traffic.
Set up a similar mechanism on the OLB to allow seamless failover if needed during the transition phase.

Gradual Traffic Shift

Gradually increase the percentage of traffic directed to the NLB over time, typically in increments of 10% every hour or so.
This allows for sufficient time to detect and address any issues that may arise.

DNS TTL Management

As more traffic is shifted to the NLB, incrementally extend the DNS TTL on the old load balancer's records.
For example, after 10% of traffic is switched over, increase the TTL from 5 seconds to 300 seconds (5 minutes). Continue this process until the new DNS record has a full TTL that matches the original.

Complete Transition

Once all traffic is routed through the NLB and there are no issues detected in health checks, set the DNS TTL of the old load balancer's records to 1 second or less to force immediate DNS cache invalidation.
Finally, remove the old load balancer from the network configuration.

Post-Transition Monitoring

Perform a final round of monitoring and logging on both OLB and NLB to ensure that no issues are present.
Document any lessons learned for future transition efforts.

#### Closing Summary

By carefully managing DNS TTLs, phased traffic rollout, and health checks, we can achieve a zero-downtime cutover. This approach ensures minimal disruption and allows time to detect and mitigate potential issues before the final DNS record update.

#### Common Mistakes/Follow-Ups
Ignoring In-Flight Connections: Failing to account for connections that are in progress during DNS TTL changes.
Insufficient Monitoring: Not setting up adequate monitoring and logging can lead to undetected issues post-transition.
Overly Aggressive TTOF (Time To Overlap Failover): Too quickly removing the old load balancer, which could cause downtime if critical services are not ready.

**Comparison:**

| Time | Percentage Shifted | TTL (Seconds) |
|------|--------------------|---------------|
| 0    | 1%                 | 5             |
| 2h   | 11%                | 30            |
| 4h   | 21%                | 60            |
| 6h   | 31%                | 90            |
| 8h   | 41%                | 120           |
| 10h  | 51%                | 150           |
| 12h  | 61%                | 180           |
| 14h  | 71%                | 300           |
| 16h  | 81%                | 600           |
| 18h  | 91%                | 900           |
| 20h  | 100%               | 3600          |

**Code:**

```python
# Example Python code for health check and traffic redirection

def setup_health_check(nlb_ip):
    # Set up a health check mechanism on NLB
    pass

def shift_traffic(percentage, nlb_dns_record):
    # Route a percentage of traffic to the new load balancer
    pass

def monitor_and_log():
    # Monitor the health and logs of both OLB and NLB
    pass

# Example usage
setup_health_check("10.0.0.2")
shift_traffic(5, "newlb.example.com")
monitor_and_log()
```

**Example input / output:**

```text
Input:
- DNS TTLs set to initial short value
- Traffic routed initially to old load balancer

Output:
- Gradual traffic shift from old to new load balancer
- DNS TTL increased over time until full transition
- Removal of old load balancer after final TTL update
```

**Data:**

```yaml
load_balancer_config:
  old_lb: "olb.example.com"
  new_lb: "nlb.example.com"

dns_records:
  - name: www.example.com
    ttl: 300
    records:
      - A: 192.0.2.1 (old_lb)
      - A: 192.0.2.2 (new_lb)
```

**Flow:**

1. No
2. Yes
3. Yes
4. Start Transition
5. Is Traffic Completely Shifted?
6. Increase TTL and Shift MoreTraffic
7. Check Health of NLB
8. Is NLB Healthy?
9. Reroute 100% to NLB
10. Remove Old Load Balancer
11. Monitor and Log

