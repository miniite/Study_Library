---
type: "practice"
difficulty: "Deep"
---

# Design the org's standard for how streaming pipelines fail: fail-open (keep flowing, drop correctness) versus fail-closed (stop, protect correctness). How do you make the choice per-pipeline explicit and enforced, and what's the safe default for AIOps?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Streaming for AIOps](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Streaming_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing streaming pipelines in an AIOps environment, choosing between a fail-open or fail-closed strategy is crucial. Fail-open ensures continuous operation but risks losing data integrity, while fail-closed guarantees data correctness at the cost of potentially interrupting service. The decision should be explicit and enforced per-pipeline, with a safe default that balances these trade-offs.

To achieve this, we'll leverage a metadata-driven approach where each pipeline specifies its failure mode explicitly. We’ll use a combination of configuration management tools and robust governance policies to enforce these decisions. Let's break down the solution step-by-step:

Define the Failure Modes: Fail-open and fail-closed are the primary options.
Metadata Configuration: Use metadata or annotations in pipeline configurations to specify failure modes.
Enforcement Mechanism: Implement a governance layer that ensures pipeline configurations adhere to specified failure modes.
Default Settings: Set a default value, such as fail-closed, and ensure this is configurable on a per-pipeline basis.

#### Triage
Understanding the Question

The question probes understanding of real-time streaming architecture principles and decision-making processes in distributed systems. It seeks an explicit mechanism for handling failures in pipelines, ensuring compliance with organizational policies while providing flexibility where necessary.

Step-by-Step Solution From First Principles
1. Define Failure Modes

Fail-open:

Allows data to continue flowing even when there are issues.
Use cases: scenarios where data loss is acceptable but service continuity is critical (e.g., financial trading systems).

Fail-closed:

Halts the pipeline upon detecting an issue, ensuring no incorrect data flows downstream.
Use cases: scenarios requiring high data integrity and correctness (e.g., medical records processing).
2. Metadata Configuration

Each streaming pipeline should have a metadata field indicating its failure mode. This can be stored in a configuration file or managed through a centralized service.

YAML
# Example YAML configuration for a pipeline
pipeline:
  name: financial-trading-system
  description: Real-time trading system
  failure_mode: fail-open  # Options: "fail-closed" or "fail-open"
3. Enforcement Mechanism

Implement a governance layer that validates the failure mode settings against predefined policies.

PYTHON
def validate_pipeline_config(config):
    if config["failure_mode"] not in ["fail-closed", "fail-open"]:
        raise ValueError("Invalid failure_mode specified.")
    # Additional validation logic can be added here
4. Default Settings

Set a default value for the failure_mode that aligns with best practices and organizational needs. For AIOps, a common safe default might be fail-closed.

YAML
# Example of setting a default in the configuration file or via code
default_failure_mode: "fail-closed"

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
Explicitly defining failure modes through metadata ensures that each pipeline's behavior is clear and predictable. The governance layer enforces these definitions, ensuring compliance with organizational policies. A safe default of fail-closed provides a robust baseline while allowing for flexible configuration.

#### Common Mistakes/Follow-Ups

Lack of Explicitness: Not specifying the failure mode in each pipeline can lead to inconsistent behavior.
Overly Permissive Defaults: Setting too permissive defaults (e.g., fail-open) without proper validation can introduce data integrity issues.
Manual Override: Allowing manual overrides without governance checks can bypass critical policies.
Edge Cases: Handling edge cases, such as partial failures or transient errors, requires careful consideration to ensure system resilience.

For follow-ups:

How would you handle partial failures within a pipeline?
What are the implications of choosing fail-open for pipelines with high throughput but low latency requirements?

**Code:**

```python
def validate_pipeline_config(config):
    if config["failure_mode"] not in ["fail-closed", "fail-open"]:
        raise ValueError("Invalid failure_mode specified.")
```

