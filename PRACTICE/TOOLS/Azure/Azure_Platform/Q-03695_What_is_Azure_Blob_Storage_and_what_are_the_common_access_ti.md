---
type: "practice"
difficulty: "Basic"
---

# What is Azure Blob Storage and what are the common access tiers (hot/cool/archive)?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure SQL and App Service Deployments](<../../../../TOOLS/Azure/Azure_Platform/Azure_SQL_and_App_Service_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Azure Blob Storage is a highly scalable, secure, and durable object storage service provided by Microsoft Azure. It allows you to store large amounts of unstructured data like text files, images, videos, etc., in a cloud environment. The common access tiers available for Blob Storage are Hot, Cool, and Archive.

Let's break down what each tier means:

Hot Access Tier: This is the default tier that provides fast and frequent access to your data. It’s suitable for scenarios where you need immediate, high-frequency access to your blobs.
Cool Access Tier: The Cool tier stores rarely accessed data, making it ideal for archival purposes or long-term storage of infrequently used data. Data in this tier is automatically moved from the Hot tier after 30 days and offers a lower cost per GB compared to Hot tier.
Archive Access Tier: The Archive tier is designed for storing extremely large volumes of rarely accessed data, such as backups or historical records. Data stored here is automatically moved from the Cool tier after 180 days. This tier provides the lowest cost per GB but also the slowest access time.

To summarize:

Hot Tier = Fast and frequent access
Cool Tier = Rarely accessed data, lower cost than Hot, faster than Archive
Archive Tier = Extremely large volumes of rarely accessed data, lowest cost

It's important to note that Azure Blob Storage offers different tiers based on the frequency of access required. Choosing the right tier can significantly impact costs and performance.

**Flow:**

1. Hot | Cool
2. Archive
3. User
4. Create Container
5. Access Tier Needed?
6. Choose Hot Tier for HighFrequency Access
7. Choose Archive Tier for RarelyAccessed Data

