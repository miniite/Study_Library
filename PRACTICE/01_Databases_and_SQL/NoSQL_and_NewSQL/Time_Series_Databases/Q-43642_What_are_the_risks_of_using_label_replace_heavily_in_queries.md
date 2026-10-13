---
type: "practice"
difficulty: "Intermediate"
---

# What are the risks of using label_replace() heavily in queries, and how can it hurt performance or correctness in a correlation rule?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ PromQL Pitfalls](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/PromQL_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Prometheus or InfluxDB, label_replace() is an extremely useful function for manipulating labels within your time-series data. However, using this function heavily can introduce significant risks to both performance and correctness in correlation rules. This answer will break down the key issues and provide strategies to mitigate these risks.

**Comparison:**

```
Query Example	Potential Risk
label_replace(label_replace(foo{job="bar"}, "old_label1", "$2", "old_label2", "$(2)"), "new_label", "$3", "old_label3", "$(3)")	High complexity, potential for incorrect results and performance degradation
```

**Code:**

```
PROMQL
Copy
# Example of heavy use of label_replace()
label_replace(
  label_replace(
    foo{job="bar"},
    "new_label",
    "$2",
    "old_label1", "$(2)"
  ),
  "another_new_label",
  "$3",
  "another_old_label", "$(3)"
)
```

