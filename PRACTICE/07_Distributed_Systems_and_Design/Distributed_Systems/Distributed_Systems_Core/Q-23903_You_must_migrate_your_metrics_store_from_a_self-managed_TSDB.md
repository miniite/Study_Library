---
type: "practice"
difficulty: "Deep"
---

# You must migrate your metrics store from a self-managed TSDB to a managed one with zero detection downtime on a live pipeline. Walk me through dual-write, shadow-read verification of baselines, backfill, cutover, and where you keep rollback armed.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Verifying Live Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate your metrics store from a self-managed TSDB to a managed one with zero detection downtime on a live pipeline, we need to employ a dual-write strategy, perform shadow reads to verify baselines, handle backfilling data, execute the cutover process, and ensure rollback capabilities are armed. This approach ensures that our system remains stable during the transition.

**Comparison:**

| Metric     | Current Value (Self-Managed) | Managed TSDB Value | Discrepancy |
|------------|------------------------------|--------------------|-------------|
| cpu_usage  | 85.4                         | 86.2               |             |
| memory_free| 1976 MB                      | 1968 MB            | 8 MB        |

**Code:**

```python
# Example of setting up dual writes in Python
def start_dual_write(self):
    self.managed_tsdb.start_write()
    self.self_managed_tsdb.start_write()

def stop_dual_write(self):
    self.managed_tsdb.stop_write()
    self.self_managed_tsdb.stop_write()
```

**Example input / output:**

```text
# Example input and output for dual write

Input:
metrics_data = {
  "timestamp": "2023-10-01T00:00:00Z",
  "metric_name": "cpu_usage",
  "value": 85.4,
  "tags": {"instance_id": "i-0123456789abcdef"}
}

Output:
Managed TSDB write result: SUCCESS
Self-managed TSDB write result: SUCCESS

# Shadow read verification example
shadow_read_result = verify_baselines(metrics_data)
print("Verification Result:", shadow_read_result)  # Output: Verification Result: PASSED
```

**Data:**

```json
{
  "metrics_data": {
    "timestamp": "2023-10-01T00:00:00Z",
    "metric_name": "cpu_usage",
    "value": 85.4,
    "tags": {"instance_id": "i-0123456789abcdef"}
  }
}
```

**Flow:**

1. Start Dual Write
2. Migrate Data to ManagedTSDB
3. Verify Baselines using ShadowReads
4. Backfill Missing Data fromSelf-Managed
5. Cutover to Managed TSDB
6. Audit Rollback Capabilities
7. End Migration Process

