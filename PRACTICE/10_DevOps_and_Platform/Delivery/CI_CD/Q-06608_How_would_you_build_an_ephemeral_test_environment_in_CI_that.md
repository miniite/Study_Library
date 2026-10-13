---
type: "practice"
difficulty: "Deep"
---

# How would you build an ephemeral test environment in CI that replays a captured slice of real production telemetry to validate a detector or correlation change end-to-end?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an ephemeral test environment in Continuous Integration (CI) that replays a captured slice of real production telemetry, you need to ensure that your changes are validated against realistic data without impacting live systems or causing delays. This process involves several key steps:

Capture Real Production Telemetry: Collect and store the relevant production telemetry data.
Create an Ephemeral Environment: Set up a disposable environment for testing.
Replay the Data: Simulate the captured telemetry in your test setup.
Validate Changes: Use the replayed data to validate changes end-to-end.

To ensure a deep understanding, let's break down each step and consider various factors.

**Comparison:**

```
Step	Description
1	Capture real production telemetry using appropriate monitoring tools.
2	Create an ephemeral environment in CI that can run independently of live systems.
3	Replay the captured telemetry data within this environment to simulate real conditions.
4	Validate changes (e.g., detector or correlation logic) against the replayed data.
```

**Code:**

```python
# Example Python function to capture telemetry
def capture_telemetry():
    # Code to log or store telemetry data
    pass

# Example setup for replaying telemetry in CI
def setup_replay_environment():
    # Steps to create ephemeral environment
    pass

# Example replay logic
def replay_telemetry(telemetry_data):
    # Code to simulate and validate the telemetry data
    pass
```

**Example input / output:**

```text
Input:
- Production telemetry data captured during a specific time window

Output:
- Ephemeral test environment setup and validation results for the detector or correlation change
```

**Data:**

```json
{
  "telemetry": {
    "timestamp": "2023-10-01T00:00:00Z",
    "events": [
      {"type": "alert", "details": {...}},
      {"type": "log", "details": {...}}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Capture Telemetry
4. Store Data?
5. Setup Ephemeral Environment
6. Refine Capture
7. Replay Telemetry in CI
8. Test Changes End-to-End
9. Validate Success/Failure

