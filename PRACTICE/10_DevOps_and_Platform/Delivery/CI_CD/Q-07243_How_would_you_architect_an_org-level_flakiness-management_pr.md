---
type: "practice"
difficulty: "Expert"
---

# How would you architect an org-level flakiness-management program (detection, quarantine, ownership SLAs, budget) and defend it as a platform investment rather than each team's private problem?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an org-wide flakiness management program, we need to establish a robust system that detects, quarantines, and manages intermittent test failures across all teams in the organization. This involves setting up clear ownership SLAs and budget allocation, ensuring that flaky tests do not impede development velocity or project timelines. The key is to treat this as a platform investment rather than leaving it to each team to handle independently.

The fundamental challenge lies in aligning stakeholders from various teams on why a centralized approach is necessary, especially when some might initially resist due to concerns about additional overhead and perceived lack of immediate benefit. By framing the solution within the context of improving overall system reliability and developer productivity, we can make a compelling case for this investment.

Here’s how we would architect such a program step-by-step:

Define Flakiness: Establish criteria for identifying flaky tests. This includes defining what constitutes a flaky test (e.g., failures that occur sporadically) and setting up mechanisms to detect these anomalies in real-time.
Detection Mechanisms: Implement CI/CD pipelines with enhanced monitoring tools that can automatically flag flaky tests based on predefined thresholds or patterns of failure.
Quarantine Processes: Once a test is identified as flaky, it should be quarantined from the main testing processes to prevent it from causing false negatives in the overall build status. This requires a controlled environment where these tests can run without impacting regular workflows.
Ownership SLAs and Roles: Assign clear ownership and responsibilities for managing flaky tests. Each team should have defined SLAs (Service Level Agreements) outlining how quickly they must address identified issues, with penalties or incentives tied to performance.
Budget Allocation: Allocate dedicated budget towards this program to support necessary tools, personnel, and infrastructure improvements. This ensures that the initiative has sufficient resources to be effective without burdening individual teams financially.

In defending this as a platform investment rather than a private problem for each team, we need to highlight several strategic benefits:

Enhanced Reliability: Centralized management can lead to better identification and resolution of intermittent issues, improving overall system reliability.
Cost Efficiency: By centralizing resources and expertise, we avoid redundant investments in tooling and personnel across different teams.
Developer Productivity: Reducing flaky tests improves developer satisfaction by minimizing frustration from false negatives and broken builds. This boosts productivity as developers can focus on more valuable work.

**Comparison:**

```
Role	Responsibilities	SLA (Days)
DevOps	Monitor CI/CD pipelines, trigger quarantine	24
Test Engineer	Investigate and resolve flaky tests	7
Developer	Collaborate with teams to fix issues	As needed
```

**Code:**

```python
# Example of a simple CI/CD configuration flagging flakiness
def detect_flakiness(test_results):
    # Criteria for detecting flaky tests (e.g., failures over 3 consecutive runs)
    if len([result for result in test_results[-5:] if result == "failure"]) > 2:
        return True
    return False

# Example of handling a detected flaky test
def handle_flakiness(flaky_test, team):
    print(f"Flaky test {flaky_test} detected in Team {team}.")
    # Automated quarantine process (simplified for illustration)
    quarantine_test(flaky_test)

def quarantine_test(test_name):
    print(f"Quarantining test {test_name}...")
```

**Example input / output:**

```text
Input: Test results for a particular build.
Output:
- Detection of a flaky test based on predefined criteria.
- Quarantine process initiated for the identified flaky test.
```

**Data:**

```yaml
# Example configuration file for flakiness detection and handling
flakiness_config:
  criteria: "at least 2 out of last 5 runs failed"
  actions:
    - name: "Detect Flaky Test"
      description: "Identify tests that fail intermittently"
    - name: "Quarantine Flaky Test"
      description: "Isolate test from regular pipeline to prevent false negatives"
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a detected flaky test?
5. Log and quarantine the flakytest
6. Continue monitoring builds fornew flakiness issues
7. Notify relevant stakeholders(DevOps, Test Engineer)
8. Maintain logs and metrics onresolution time

