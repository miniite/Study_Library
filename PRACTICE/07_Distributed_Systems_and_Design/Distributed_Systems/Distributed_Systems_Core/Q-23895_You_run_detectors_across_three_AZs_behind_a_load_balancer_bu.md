---
type: "practice"
difficulty: "Deep"
---

# You run detectors across three AZs behind a load balancer, but a zonal outage still caused a full detection gap. Walk me through how cross-zone balancing, unhealthy-host ejection, and zonal shift interact and where the design failed to isolate the AZ.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, especially in AIOps environments, ensuring high availability and fault tolerance is crucial. The challenge here involves understanding how cross-zone balancing, unhealthy-host ejection, and zonal shift interact to maintain detection capabilities across three Availability Zones (AZs). Let’s break down the scenario and identify where the design might have failed.

The system is structured with detectors distributed across three AZs behind a load balancer. During a zonal outage, there was a full detection gap despite these mechanisms being in place. This suggests that the interactions between cross-zone balancing, unhealthy-host ejection, and zonal shift are not as effective as intended.

#### Triage
Cross-Zone Balancing: Ensures that requests are evenly distributed across multiple AZs to mitigate the impact of a single zone failure.
Unhealthy-Host Ejection: Automatically removes hosts from the load balancer if they become unhealthy, preventing them from further contributing to the load.
Zonal Shift: A mechanism to automatically move workloads between zones to ensure high availability.
Key Concepts and Assumptions
Cross-Zone Balancing: Configured correctly but might not have redundant paths or failover strategies.
Unhealthy-Host Ejection: Should detect failures quickly, but the detection logic might be flawed.
Zonal Shift: Designed to handle zone-level issues, but might lack robustness in critical scenarios.
Step-by-Step Solution

Understand the Outage Scenario:

A zonal outage occurred.
All detectors in that AZ were unavailable for detecting anomalies.
Cross-zone balancing did not fully mitigate this issue.
Unhealthy-host ejection and zonal shift mechanisms should have kicked in but did not.

Analyze Cross-Zone Balancing:

Cross-zone balancing might be configured to send requests only if the primary zone is healthy, which means all detectors in a failing AZ would not receive any traffic.
Alternatively, cross-zone balancing could be set up with a timeout mechanism that allows fallback paths, but these might need configuration or adjustment.

Examine Unhealthy-Host Ejection:

Unhealthy-host ejection should ideally detect failures and eject unhealthy hosts from the load balancer.
If the detection logic is slow or incomplete, it could miss the failure in time to prevent a full detection gap.
There might be issues with how health checks are implemented. For example, if pings or heartbeats are used, they could fail to detect deeper-level failures.

Evaluate Zonal Shift:

Zonal shift should automatically move workloads from an unhealthy zone to healthy ones.
However, the shift might not be rapid enough to prevent a detection gap in critical scenarios.
There could also be issues with how resources are managed during shifts, such as data loss or temporary unavailability.
Detailed Analysis and Potential Improvements

Cross-Zone Balancing:

Ensure that cross-zone balancing is configured to handle partial outages by setting up fallback paths.
Implement a retry mechanism for failed requests within the same AZ before falling back to other zones.

Unhealthy-Host Ejection:

Enhance health check mechanisms to detect deeper-level failures more accurately and quickly.
Introduce multiple layers of health checks, such as monitoring CPU usage, memory usage, and network connectivity in addition to basic pings or heartbeats.

Zonal Shift:

Optimize the zonal shift mechanism to ensure rapid failover without significant data loss or unavailability.
Implement a hybrid approach that combines proactive zonal shifts with reactive ejection of unhealthy hosts.

#### Closing Summary

The design failed primarily in ensuring continuous detection despite a zonal outage. Cross-zone balancing, unhealthy-host ejection, and zonal shift are crucial mechanisms but need to be robustly configured and monitored. Improvements could involve enhanced health check mechanisms, fallback paths for cross-zone balancing, and optimized zonal shift strategies.

#### Common Mistakes/Follow-Ups
Common Mistake: Insufficient health check granularity leading to late detection of failures.

#### Follow-Up
How would you implement a monitoring system that can detect these issues in real-time?
Further Discussion: What other resilience mechanisms would you consider adding, and how do they interact with the existing ones?

**Comparison:**

```
AZ	Status	Detectors Healthy	Last Heartbeat
Zone1	Down	0	N/A
Zone2	Up	1	14:05
Zone3	Up	2	14:15
```

**Code:**

```python
# Example health check implementation for enhanced detection
def check_health(ip):
    response = ping(ip)
    if not response:
        return False
    # More detailed checks like network connectivity, CPU usage, etc.
    return True
```

**Example input / output:**

```text
Input:
- Zone1 is down due to a zonal outage.
- Cross-zone balancing is configured but does not provide fallback paths.

Output:
- Detectors in Zone1 are unreachable, leading to detection gaps.
- Unhealthy-host ejection fails to eject unhealthy hosts quickly.
- Zonal shift is too slow or incomplete, causing further delays in recovery.
```

**Data:**

```json
{
  "zone1": {
    "status": "down",
    "detectors": [
      {"id": 101, "status": "unhealthy", "last_heartbeat": "2023-10-05T14:00Z"}
    ]
  },
  "zone2": {
    "status": "up",
    "detectors": [
      {"id": 201, "status": "healthy", "last_heartbeat": "2023-10-05T14:05Z"},
      {"id": 202, "status": "unhealthy", "last_heartbeat": "2023-10-05T14:10Z"}
    ]
  },
  "zone3": {
    "status": "up",
    "detectors": [
      {"id": 301, "status": "healthy", "last_heartbeat": "2023-10-05T14:15Z"},
      {"id": 302, "status": "healthy", "last_heartbeat": "2023-10-05T14:20Z"}
    ]
  }
}
```

**Flow:**

1. Zone1 Down
2. Health Check
3. Unhealthy Host Ejection
4. Zonal Shift
5. Cross-Zone Balancing Fallback

