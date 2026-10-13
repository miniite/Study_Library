---
type: "practice"
difficulty: "Basic"
---

# In simple terms, what is 'configuration as code' and how does it apply to observability config like alert thresholds?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Configuration as code (CaaS) is a practice where software configurations are stored in version-controlled text files, typically using languages like YAML or JSON. This approach allows developers and operators to manage configuration settings in a more systematic and repeatable manner, similar to how code is managed.

In the context of CI/CD, CaaS helps streamline observability by treating alert thresholds, logging levels, and other monitoring configurations as first-class citizens alongside application code. This means that these configurations can be versioned, automated, and tested just like your application code, ensuring consistency and reliability across different environments.

**Code:**

```yaml
# Example of a CaaS file for observability settings in YAML
observability:
  alerting_thresholds:
    - metric: cpu_usage
      threshold: 80
      action: warn
    - metric: memory_usage
      threshold: 95
      action: critical
  logging_levels:
    application: info
    database: error
```

**Flow:**

1. Yes
2. No
3. Configuration as Code
4. Is the configuration versionedand automated?
5. Consistent acrossenvironments
6. Configuration drift may occur

