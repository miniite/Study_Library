---
type: "practice"
difficulty: "Intermediate"
---

# You're told to keep 7 years of granular transactions for compliance but 99% of queries touch the last 90 days. How would you tier storage so audits over old data are possible without paying hot rates forever?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to design a tiered storage architecture that balances cost and performance while meeting compliance requirements. The core idea is to separate "hot" (recent) data from "cold" (older) data, ensuring rapid access for frequent queries while keeping older data in more economical storage tiers.

The question is probing your understanding of how to implement a tiered storage strategy within a data architecture. Specifically, it tests your ability to make trade-offs between cost and performance based on the access patterns of the data. We need to assume that "hot" data are those accessed frequently (last 90 days), while "cold" data are those rarely accessed but required for compliance purposes.

Firstly, let's define our storage tiers:

Hot Storage: This tier stores the most recent data and is optimized for fast access.
Warm Storage: This tier stores data that is less frequently accessed than hot data but more so than cold data.
Cold Storage: This tier stores the oldest data, which are infrequently accessed.

Let's break down the solution step-by-step:

Identify Access Patterns:

Most queries touch the last 90 days of data.
Compliance requires keeping data for at least 7 years.

Choose Appropriate Storage Solutions:

For hot storage, use an in-memory or SSD-based database to ensure low-latency access.
For warm storage, consider using a more cost-effective storage solution like NVMe SSDs.
For cold storage, leverage cheaper and slower storage options such as HDDs or object storage services.

Implement Data Aging Strategy:

Automatically move data from hot to warm storage after 90 days.
Further migrate data from warm to cold storage after 1 year of in-warm storage.
Finally, archive data to an external storage solution like AWS S3 Glacier or Azure Blob Storage after 7 years.

Design a Data Mesh:

Use a microservices approach where each service owns its data and is responsible for managing its lifecycle.
Ensure that the data mesh supports both hot and cold tiers through dedicated API endpoints and workflows.

Monitor and Optimize:

Continuously monitor access patterns to re-evaluate storage tiering as needed.
Implement automated cost optimization tools to adjust storage policies based on usage trends.

Here's a high-level diagram of the architecture:

Query
Periodic Migrations
Archive Policies
Data Access Patterns
Data Aging Strategy
Data Archiving Policy
Hot Storage
Query Engine
Warm Storage
Cold Migration Service
Cold Storage
Archive Service
External Storage

**Comparison:**

| Tier | Description                    | Technology               |
|------|-------------------------------|--------------------------|
| Hot  | Recently accessed data         | In-memory database       |
| Warm | Less frequently accessed      | NVMe SSDs                |
| Cold | Older than one year, infrequent access | HDDs or object storage   |

**Code:**

```python
# Example code to move data from hot to warm storage
def migrate_to_warm_storage(data):
    # Logic to move data to warm storage
    pass

# Example code to archive cold data
def archive_cold_data(data):
    # Logic to archive data to external storage
    pass
```

**Example input / output:**

```text
Input: A query requesting data from the last 90 days.
Output: The query is processed by the hot storage tier and returned in milliseconds.

Input: An audit request for data older than a year.
Output: Data is fetched from cold storage or archived external service, possibly with increased latency.
```

**Data:**

```yaml
storage_tiers:
  hot:
    - description: Data accessed frequently within the last 90 days
    - technology: In-memory database or SSD-based solution
  warm:
    - description: Data that is less frequently accessed, but still important for recent queries
    - technology: NVMe SSDs
  cold:
    - description: Data older than one year, infrequently accessed
    - technology: HDDs or object storage services like AWS S3 Glacier
```

**Flow:**

1. Data Access
2. Periodic Migrations
3. Aging Strategy
4. HotStorage
5. QueryEngine
6. WarmStorage
7. ColdMigrationService
8. ColdStorage
9. ArchiveService
10. ExternalStorage

