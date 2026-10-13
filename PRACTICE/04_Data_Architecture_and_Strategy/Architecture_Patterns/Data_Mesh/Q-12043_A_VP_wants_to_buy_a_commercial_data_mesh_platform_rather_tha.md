---
type: "practice"
difficulty: "Expert"
---

# A VP wants to buy a commercial data mesh platform rather than build one on your existing stack; how do you frame the build-versus-buy decision for the exec team?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Mesh Principles](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Mesh_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to present a comprehensive analysis of why investing in a commercial data mesh platform could be beneficial compared to building one from scratch. The key points to consider are cost, time-to-market, scalability, and long-term sustainability.

The core idea is that while building an internal solution might seem like it would allow for full control over the architecture and implementation, leveraging a commercial product can offer significant advantages in terms of resources, expertise, and agility. Let’s break this down from first principles.

**Comparison:**

```
Cost	Customized vs Off-the-Shelf
Build	High customization required; higher costs due to in-house development and maintenance
Buy	Lower initial setup cost; ongoing subscription fees, but potentially lower total ownership cost
Time-to-Market	Learning Curve and Expertise
Build	Higher learning curve for custom architecture; requires in-house expertise
Buy	Faster deployment as platform is ready to use; reduces dependency on skilled resources
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Current Data Stack] --> B1[Cost];
    A --> B2[Time-to-Market];
    A --> B3[Scalability];
    A --> B4[Sustainability];

    C[Commercial Data Mesh Platform] --> D1[Reduced Cost];
    C --> D2[Faster Time-to-Market];
    C --> D3[Enhanced Scalability];
    C --> D4[Better Long-Term Sustainability];

    B1 --> E[Customization and Control];
    B2 --> F[Learning Curve and Expertise];
    B3 --> G[Resource Allocation];
    B4 --> H[Risk of Building In-House]

    D1 --> I[Off-the-Shelf Solutions];
    D2 --> J[Ready-to-Use Features];
    D3 --> K[Managed Infrastructure];
    D4 --> L[Vendor Support and Updates];
```

**Example input / output:**

```text
Input: Current data stack consists of PostgreSQL, Apache Spark for processing, and S3 for storage. VP is considering a commercial platform that offers managed services like AWS Redshift for database, AWS Glue for processing, and S3 with integrated analytics tools.

Output: The commercial platform could reduce costs by about 25% due to the managed nature (skip in-house DBA roles) and faster time-to-market as it comes pre-configured. It also offers enhanced scalability with managed infrastructure and better long-term sustainability through vendor support.
```

**Data:**

```yaml
# Example Data Stack Configuration
current_stack:
  - database: PostgreSQL
  - processing_tool: Apache Spark
  - storage: S3
  - analytics_tool: Tableau

commercial_mesh_platform:
  - database: AWS Redshift
  - processing_tool: AWS Glue
  - storage: S3
  - analytics_tool: Looker
```

**Flow:**

1. Current Data Stack
2. Develop and Maintain
3. High Cost
4. Long Development Time
5. Resource Intensive
6. Vendor Solution
7. Lower Initial Cost
8. Faster Deployment
9. Easier to Scale
10. Build vs Buy Decision

