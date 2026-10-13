---
type: "practice"
difficulty: "Deep"
---

# A Prometheus scrape config with relabel_configs stopped matching after a YAML edit. How do the ordered YAML list of relabel rules and their source_labels sequence combine, and how do you debug ordering bugs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring Helm Values](<../../../../TOOLS/YAML/YAML_Core/Refactoring_Helm_Values.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Prometheus's relabel_config in YAML, understanding the combined effect of multiple relabel rules is crucial to ensure they work as intended. The key point here is that the order and sequence of source_labels within each rule matter significantly, especially when these rules are applied in a list.

#### Triage

The question is essentially asking how the ordered list of relabel rules and their corresponding source_labels interact in Prometheus. Specifically, it aims to understand how changes in YAML structure can affect scraping behavior, and what steps can be taken to debug issues related to rule ordering.

Step-by-Step Solution
Understanding Relabel Rules: A relabel_config consists of multiple rules applied sequentially. Each rule contains a source_labels, a target_label, and optional conditions like regex_replace.
Ordering of Rules: The order in which these rules are listed matters because they can modify the labels at each step before moving to the next rule.
Impact of source_labels Sequence:
Each relabel_config has a list of source_labels. These labels must exist and be correct for the rule to apply. If any source_label is missing, the entire rule sequence may fail as expected.
Debugging Steps:
Check Rule Order: Verify that the order of rules aligns with your intent. Sometimes, moving a rule up or down can fix issues.
Review Source Labels: Ensure all required source_labels are present and correctly specified in each rule.
Use Debug Logging: Enable Prometheus logging to trace label modifications at each step.
Detailed Example

#### Cons
ider the following simplified YAML configuration:

YAML
relabel_configs:
  - source_labels: ["__name__"]
    target_label: "job"
    replacement: "prometheus"

  - source_labels: ["__metrics_path__"]
    target_label: "path"
    replacement: "/metrics"

  - source_labels: ["__address__"]
    target_label: "instance"

Here, the rules are applied in order:

The first rule sets job to "prometheus" based on __name__.
The second rule sets path to /metrics based on __metrics_path__.
The third rule sets instance based on __address__.

#### Common Mistakes
and Follow-Ups
Incorrect Order: Failing to understand the dependency between rules can lead to issues if a required label is not present when needed.
Missing Labels: If any source_labels are missing, Prometheus will skip that rule sequence, which might cause unexpected behavior or downtime.
Debugging Tips:
Use Prometheus's built-in logging features to trace the label modifications at each step.
Ensure you have a clear understanding of how each rule affects the scraping process.

#### Closing Summary

To effectively manage and debug relabel_config issues, it’s essential to carefully consider the order and sequence of rules. Each source_label must be correctly specified and present for the relabeling process to succeed. Debugging techniques like enabling Prometheus logging can provide valuable insights into label modifications at each step.

#### Common Mistakes
Incorrect Order: Misordering rules can lead to missing or incorrect label values.
Missing Labels: Rules may fail if required source_labels are not present, causing unintended behavior.

**Code:**

```yaml
relabel_configs:
  - source_labels: ["__name__"]
    target_label: "job"
    replacement: "prometheus"

  - source_labels: ["__metrics_path__"]
    target_label: "path"
    replacement: "/metrics"

  - source_labels: ["__address__"]
    target_label: "instance"
```

