---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you'd unify metric naming and label conventions across teams before their telemetry lands in a shared lake.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To unify metric naming and label conventions across multiple AIOps teams, we need to establish consistent standards that ensure interoperability and ease of analysis when telemetry data is ingested into a shared Observability Data Lake. The goal is not only to standardize the metrics but also to define clear labeling practices that facilitate querying and aggregation.

#### Triaging the Question

The core issue here is ensuring uniformity in metric naming and label conventions across various teams contributing to an AIOps platform. This unification will help in maintaining a cohesive data model within the Observability Data Lake, making it easier for analysts and engineers to query, visualize, and correlate metrics from different sources.

#### Assumptions
Diverse Teams: Multiple development or operational teams contribute telemetry data.
Shared Lake: A centralized data lake is used to store this telemetry data.
Metric and Label Standardization: Consistent naming conventions for metrics and labels across all contributing teams.
Tooling Support: There are tools available to enforce these standards during the ingestion process.

#### Solution
from First Principles

Define Naming Conventions:

Metrics: Establish a standardized prefix or suffix system (e.g., app_name_metric for application metrics, system_service for infrastructure).
Labels: Define common label keys and their meanings across all teams (e.g., environment, team, component).

Develop Guidelines Document:

Create a comprehensive document that outlines the metric naming conventions and label definitions.
Include examples of valid metric names and labels to illustrate the correct format.

Automate Validation:

Use tools to automatically validate incoming telemetry data against the defined standards before ingestion into the Data Lake.
Tools like Fluent Bit, Telegraf, or custom scripts can be employed for this purpose.

Training and Communication:

Conduct training sessions with teams to ensure they understand the new conventions and practices.
Establish clear communication channels (e.g., Slack, internal wikis) where teams can refer to the guidelines and ask questions.

Feedback Loop:

Implement a feedback mechanism where any deviations or issues are reported back to the standardization team for review and resolution.
Periodically audit data quality to ensure adherence to the established standards.

#### Closing Summary

To unify metric naming and label conventions across teams, we need to define clear guidelines, automate validation processes, provide thorough training, and maintain open communication channels. By doing so, we can ensure that telemetry data is consistent and easily analyzable within a shared Observability Data Lake.

#### Common Mistakes/Follow-Ups
Ignoring Feedback: Failing to implement feedback mechanisms can lead to incomplete adherence to standards.
Overly Complex Standards: Overly detailed or overly broad standards might create confusion rather than clarity.
Lack of Tooling: Relying solely on manual validation can be error-prone and time-consuming.

**Comparison:**

```
Metric Prefixes	Examples
app_	app1_request_count, app2_error_rate
system_	system_service_uptime
Label Keys	Description
environment	Development, Production
team	dev_team_a, dev_team_b
component	frontend, backend, database
```

**Code:**

```python
# Example Python script for basic metric naming validation
import re

def validate_metric_name(metric_name):
    # Simple regex pattern to match a valid metric name
    pattern = r'^[a-z_]+_[a-zA-Z0-9_.]+$'
    return bool(re.match(pattern, metric_name))

# Test cases
valid_metric = "app1_request_count"
invalid_metric = "3rdPartyErrorCount"

print(f"Valid: {validate_metric_name(valid_metric)}")  # True
print(f"Invalid: {validate_metric_name(invalid_metric)}")  # False
```

**Example input / output:**

```text
Input:
- Metric Name: `app1_request_count`
- Label Keys: `environment`, `team`, `component`

Output:
- Validated as per defined naming conventions.
```

**Data:**

```json
{
  "metric_definitions": {
    "prefixes": ["app_", "system_"],
    "label_keys": ["environment", "team", "component"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Metric Name Valid?
5. Log and Ingest Data
6. Notify Dev Team
7. Audit and Validate Periodically
8. Fix and Retest

