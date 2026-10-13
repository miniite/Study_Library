---
type: "practice"
difficulty: "Deep"
---

# Your service mesh sidecars all failed to start after a control-plane cert rotation, bricking the data plane. Explain how you'd design cert rotation and last-known-good config so a control-plane hiccup degrades gracefully.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When a service mesh's control plane undergoes a critical event such as a certificate (cert) rotation, it is crucial to design a system that can degrade gracefully without affecting the data plane's functionality. This involves ensuring that sidecars retain their last-known-good configurations and continue serving traffic until they can re-establish connections with an updated control plane.

#### Triage

The question requires understanding how to handle unexpected failures in the control plane of a service mesh, specifically focusing on certificate rotations. The goal is to ensure that data plane services (sidecars) do not become completely non-functional during these events and can continue serving traffic until they synchronize with the new state. This involves concepts such as last-known-good configurations, circuit breakers, and fallback mechanisms.

#### First-Principles Solution

Certificate Rotation Strategy:

Implement a smooth certificate rotation strategy where certificates are updated in stages (e.g., during scheduled maintenance windows).
Ensure that the control plane can handle multiple versions of certificates simultaneously to support both old and new sidecars.

Last-Known-Good Configurations:

Store last-known-good configurations in persistent storage, such as a distributed file system or a database.
Sidecars should periodically check this storage for updates but continue using the cached configuration until they can sync with the control plane.

Circuit Breaker Mechanism:

Implement circuit breaker patterns to isolate failures and prevent cascading issues within the data plane.
Gradually transition from old configurations to new ones based on health checks, ensuring that only healthy instances move to the new state.

Graceful Degradation Strategies:

Design fallback mechanisms where sidecars can serve traffic using a default or previously known configuration when unable to connect to the control plane.
Use health probes and metrics to monitor the status of connections between sidecars and the control plane, allowing for dynamic reconfiguration.

Fallback Mechanisms:

Implement retry policies with exponential backoff to handle temporary network issues.
Define timeout values to ensure that sidecars do not hang indefinitely when connectivity is lost.

Monitoring and Alerting:

Set up comprehensive monitoring and alerting for certificate expiration, control plane availability, and sidecar health.
Ensure that alerts are triggered early enough to prevent catastrophic failures in the data plane.

#### Closing Summary

By implementing a robust certificate rotation strategy with last-known-good configurations, circuit breaker mechanisms, graceful degradation strategies, fallback mechanisms, and comprehensive monitoring, you can ensure that service mesh sidecars degrade gracefully during control plane hiccups. This approach minimizes downtime and ensures continuous service availability even when critical infrastructure changes are being made.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring the sidecar's health and failing to implement fallback mechanisms.

#### Follow-Up
Designing robust retry policies and circuit breaker patterns can mitigate this risk.
Mistake 2: Failing to monitor critical metrics and not setting up alerting for timely intervention.

#### Follow-Up
Implementing a strong monitoring and alerting strategy ensures proactive management of issues.

**Comparison:**

```
Sidecar State	Action
Unhealthy	Retry
Unresponsive	Fallback to last known good config
Healthy	Transition to new config
```

**Code:**

```python
# Example code snippet for fallback mechanism in sidecar configuration
class SidecarConfig:
    def __init__(self, control_plane_address):
        self.control_plane_address = control_plane_address
        self.last_known_good_config = None

    def update_from_control_plane(self):
        try:
            # Attempt to fetch new config from the control plane
            self.last_known_good_config = get_new_config_from_control_plane()
        except Exception as e:
            print(f"Failed to fetch new configuration: {e}")
            if self.last_known_good_config is not None:
                print("Using last known good configuration.")

    def serve_traffic(self):
        # Serve traffic using the current config, falling back to last known good if necessary
        try:
            serve_with_new_config(self.control_plane_address)
        except Exception as e:
            if self.last_known_good_config is not None:
                serve_with_last_known_good(self.last_known_good_config)
            else:
                raise e
```

**Example input / output:**

```text
Input:
- A control plane cert rotation starts at 10 AM.

Output:
- At 10 AM, the control plane begins to rotate certificates.
- Sidecars continue serving traffic with their last-known-good configuration until they can sync with the new state.
- After a few health checks, sidecars begin to transition from old configurations to new ones based on connection status.
- By noon, all sidecars are using the updated configuration, and no degradation in service has occurred.
```

**Data:**

```json
{
  "sidecar_config": {
    "control_plane_address": "192.0.2.1:443",
    "last_known_good_config": {
      "cert": "-----BEGIN CERTIFICATE-----\nMIIDXTCC... \n-----END CERTIFICATE-----"
    },
    "health_check_interval": 5,
    "max_retries": 3
  }
}
```

**Flow:**

1. Yes
2. No
3. Sidecar starts
4. Checks health
5. Is healthy?
6. Use new config
7. Fallback to last known good
8. Health check timeout, retry

