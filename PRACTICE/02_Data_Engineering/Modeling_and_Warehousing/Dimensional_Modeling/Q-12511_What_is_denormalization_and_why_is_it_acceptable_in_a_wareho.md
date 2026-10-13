---
type: "practice"
difficulty: "Basic"
---

# What is denormalization and why is it acceptable in a warehouse but avoided in OLTP?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Denormalization is the process of optimizing data storage by reducing the normalization level, often leading to redundancy. In data warehousing, denormalization is accepted because it improves query performance and simplifies complex queries. However, in online transaction processing (OLTP) systems, denormalization is generally avoided due to its negative impacts on data consistency and maintainability.

To understand this concept fully, let's break down the key aspects and differences between data warehousing and OLTP systems.

**Comparison:**

```
Normalization Level	Data Warehouse Characteristics	OLTP Characteristics
1	High	Low
2	Medium	High
3	Low	Medium
4 and above	Very low	High
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Data Warehouse] --> B{Is denormalization needed?};
4. B --> C{Yes}::No --> D;
5. C --> E[Denormalize data];
6. F[OLTP System] --> G{Is denormalization acceptable?};
7. G --> H{No}::Yes --> I;
8. H --> J[Normalize data];

