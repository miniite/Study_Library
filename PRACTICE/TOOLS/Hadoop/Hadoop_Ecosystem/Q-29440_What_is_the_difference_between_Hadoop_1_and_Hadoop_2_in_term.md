---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between Hadoop 1 and Hadoop 2 in terms of resource management and scalability?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Hadoop and Batch Processing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hadoop_and_Batch_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this discussion, we'll explore the key differences between Hadoop 1 (also known as Hadoop MapReduce) and Hadoop 2 (which introduced YARN), focusing on their resource management mechanisms and scalability capabilities.

TL;DR: Hadoop 1 uses a single ResourceManager, which is both the scheduler and the web server. It has limitations in terms of efficient resource utilization and isolation. In contrast, Hadoop 2 introduces YARN (Yet Another Resource Negotiator), providing a more scalable architecture with separate ResourceManager for scheduling resources and NodeManager for managing containers on each node.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Resource Manager in Hadoop 1] --> B{Schedule resources?};
    B -->|Yes| C[NodeManagers];
    A --> D{Handle web requests?};
    D -->|Yes| E/WebAPI Server;

    F[YARN ResourceManager in Hadoop 2] --> G{Schedule resources?};
    G -->|Yes| H[ApplicationMaster for each application];
    F --> I{Handle web requests?};
    I -->|No| J[Separate NodeManagers for resource management];
```

**Flow:**

1. ResourceManager in Hadoop 1
2. NodeManagers
3. Web Server
4. Web API Requests
5. YARN ResourceManager
6. ApplicationMaster for eachapplication
7. NodeManager
8. Resource Management

