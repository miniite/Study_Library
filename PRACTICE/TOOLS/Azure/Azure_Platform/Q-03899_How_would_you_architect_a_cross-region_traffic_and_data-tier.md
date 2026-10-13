---
type: "practice"
difficulty: "Deep"
---

# How would you architect a cross-region traffic and data-tiering strategy so a global product serves users at low latency while keeping the Cosmos DB/Azure SQL bill sane?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Azure Networking and Global Traffic](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Networking_and_Global_Traffic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a cross-region traffic and data-tiering strategy in Azure that ensures global service availability with low latency, while maintaining cost efficiency, we need to consider multiple layers of design principles. This involves understanding Azure's regions and services like Azure Traffic Manager, Azure Cosmos DB, and Azure SQL Database, along with their respective features such as multi-master writes, automatic failover, and data distribution.

Firstly, the core idea is to leverage Azure's global network capabilities to route traffic efficiently while ensuring that data is stored in a cost-effective manner. The strategy will involve distributing read operations across multiple regions, optimizing write operations for consistency, and using intelligent data replication patterns to minimize costs.

**Comparison:**

```
Region	Cosmos DB Throughput (RU/s)	SQL Database Throughput (DTU)
East US	10000	2000
West Europe	10000	2000
Asia Pacific	10000	1500
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Global Application] --> B{Does User Require Low Latency?}
    B --> |Yes| C[Regional Traffic Manager]
    B --> |No| D[Azure App Service with Geo-Redundant Storage]
    C --> E[Azure Cosmos DB with Multi-Master Writes]
    C --> F[Azure SQL Database for Transactional Workloads]
```

**Example input / output:**

```text
Request for data from user in AsiaPacific region:
- Traffic Manager routes request to nearest regional endpoint (likely WestEurope).
- Data is retrieved from Azure Cosmos DB at low latency.
- Write operations are replicated across multiple regions with eventual consistency.
- Periodic batch jobs sync writes between regions to maintain consistency levels.

Response time: 150ms
Cost: $2.50 per month for each region's throughput (EastUS, WestEurope)
```

**Data:**

```yaml
regions:
  - EastUS
  - WestEurope
  - AsiaPacific

trafficManagerProfiles:
  - id: GlobalTrafficManager
    endpoints:
      - name: EuropeEndpoint
        type: AzureEndpoints
        hostName: example-app-west-europe.azurewebsites.net
        endpointType: Web
      - name: USEndpoint
        type: AzureEndpoints
        hostName: example-app-east-us.azurewebsites.net
        endpointType: Web

azureCosmosDB:
  databases:
    - id: globalDataStore
      throughputs:
        - region: EastUS
          throughput: 10000
        - region: WestEurope
          throughput: 10000
```

**Flow:**

1. Yes
2. No
3. User Request
4. Azure Traffic Manager
5. Closest Regional Endpoint
6. Azure App Service
7. Azure Front Door
8. Azure Cosmos DB Read-onlyGateway
9. Azure SQL Database Proxy

