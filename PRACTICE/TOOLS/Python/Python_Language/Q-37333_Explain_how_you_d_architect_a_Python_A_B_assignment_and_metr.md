---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd architect a Python A/B assignment and metric-logging layer for competing models so results are statistically comparable across teams, and what you'd log to avoid biased attribution in a shared platform.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a robust Python-based A/B testing and metric-logging system for multiple machine learning (ML) models, we need to ensure that experiments are statistically valid, results can be compared fairly across teams, and logging is comprehensive yet unbiased. The core components include a structured experiment framework, detailed logging mechanisms, and statistical analysis tools. Let’s break this down from first principles.

Firstly, the system should support clear separation of concerns: defining experiments, running them, collecting metrics, and analyzing results. Each component will be designed to work independently but communicate effectively through well-defined interfaces.

#### Triage

The question is essentially about designing a scalable and robust A/B testing framework for ML models in Python. The interviewer might probe into:

How you design the experiment setup (e.g., randomization, control groups).
What metrics are important for logging to ensure fairness.
How statistical significance is maintained across different experiments.

#### Assumptions
include:

Models from multiple teams will be tested concurrently.
Each team has unique but compatible models and datasets.
We aim to provide fair attribution of results without introducing bias.

#### Solution
1. Experiment Setup

To start, we define an experiment class that encapsulates the core logic for A/B testing. This class should handle randomization, assign instances to control or treatment groups, and manage metadata such as experiment name, start/end times, and team identifiers.

PYTHON
class ABExperiment:
    def __init__(self, experiment_name, start_time, end_time, num_control, num_treatment):
        self.experiment_name = experiment_name
        self.start_time = start_time
        self.end_time = end_time
        self.num_control = num_control
        self.num_treatment = num_treatment

    def assign_instance(self, instance_id):
        if random.random() < 0.5:  # Simple randomization for illustration
            return "control"
        else:
            return "treatment"

2. Model Logging Layer

The logging layer should capture metrics like model performance, execution time, and any other relevant data points. We need to ensure that logs are comprehensive but also avoid introducing bias.

PYTHON
class MetricLogger:
    def __init__(self):
        self.metrics = {}

    def log_metric(self, experiment_name, metric_name, value, instance_id, timestamp):
        if experiment_name not in self.metrics:
            self.metrics[experiment_name] = {}
        if metric_name not in self.metrics[experiment_name]:
            self.metrics[experiment_name][metric_name] = []
        self.metrics[experiment_name][metric_name].append((value, instance_id, timestamp))

    def get_metrics(self):
        return self.metrics
3. Statistical Analysis Tools

For statistical significance and fair comparison across teams, we can use tools like hypothesis testing (e.g., t-tests for means). A library like scipy in Python provides these functionalities.

PYTHON
from scipy.stats import ttest_ind

def compare_groups(control_data, treatment_data):
    control_values = [item[0] for item in control_data]
    treatment_values = [item[0] for item in treatment_data]

    t_stat, p_value = ttest_ind(control_values, treatment_values)
    return t_stat, p_value
4. Data Integrity and Bias Prevention

To avoid biased attribution, ensure that:

Randomization is truly random.
Metrics are logged consistently across all models.
Control groups and treatment groups have similar initial conditions.

#### Closing Summary

In summary, the architecture involves defining clear experiment boundaries, logging detailed metrics to prevent bias, and using statistical tools for fair comparisons. By rigorously designing these components, we ensure that results from different teams can be compared fairly.

#### Common Mistakes/Follow-Ups
Mistake: Failing to randomize properly or introducing systemic biases.

#### Follow-Up
Discussing how to handle edge cases such as imbalanced groups and ensuring consistent measurement practices across models.

**Code:**

```python
import random

class ABExperiment:
    def __init__(self, experiment_name, start_time, end_time, num_control, num_treatment):
        self.experiment_name = experiment_name
        self.start_time = start_time
        self.end_time = end_time
        self.num_control = num_control
        self.num_treatment = num_treatment

    def assign_instance(self, instance_id):
        if random.random() < 0.5:
            return "control"
        else:
            return "treatment"

class MetricLogger:
    def __init__(self):
        self.metrics = {}

    def log_metric(self, experiment_name, metric_name, value, instance_id, timestamp):
        if experiment_name not in self.metrics:
            self.metrics[experiment_name] = {}
        if metric_name not in self.metrics[experiment_name]:
            self.metrics[experiment_name][metric_name] = []
        self.metrics[experiment_name][metric_name].append((value, instance_id, timestamp))

    def get_metrics(self):
        return self.metrics

def compare_groups(control_data, treatment_data):
    control_values = [item[0] for item in control_data]
    treatment_values = [item[0] for item in treatment_data]

    t_stat, p_value = ttest_ind(control_values, treatment_values)
    return t_stat, p_value
```

**Example input / output:**

```text
# Example usage
exp1 = ABExperiment("ModelA_Benchmark", "2023-09-01T00:00:00Z", "2023-09-30T23:59:59Z", 100, 100)
control_logs = []
treatment_logs = []

# Simulating some log entries
for i in range(100):
    if exp1.assign_instance(i) == "control":
        control_logs.append((random.random(), i))
    else:
        treatment_logs.append((random.random(), i))

logger = MetricLogger()
logger.log_metric("ModelA_Benchmark", "performance", 0.9, 1234567890, "2023-09-15T12:00:00Z")
print(logger.get_metrics())

t_stat, p_value = compare_groups(control_logs, treatment_logs)
print(f"T-statistic: {t_stat}, P-value: {p_value}")
```

**Flow:**

1. Control
2. Treatment
3. Yes
4. No
5. Experiment Setup
6. Randomize Instances
7. (Control Data)
8. (Treatment Data)
9. Metric Logging
10. Metric Logging
11. (Control Logs)
12. (Treatment Logs)
13. F,G
14. Compare Groups
15. Statistical Significance
16. Analyze Results
17. Adjust Experiment Parameters

