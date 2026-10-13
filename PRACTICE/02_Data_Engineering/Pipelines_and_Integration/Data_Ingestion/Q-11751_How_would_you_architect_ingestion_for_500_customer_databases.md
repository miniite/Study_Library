---
type: "practice"
difficulty: "Deep"
---

# How would you architect ingestion for 500 customer databases with identical schemas but strict per-tenant isolation requirements?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect data ingestion for 500 customer databases with identical schemas and strict per-tenant isolation requirements, we need to ensure that each tenant’s data is processed independently and securely without any cross-tenant leakage. The core challenge here is managing scalability, security, and the potential complexity of handling a large number of tenants.

The key idea is to leverage Airbyte or Fivetran for their robust integration capabilities while implementing a microservices architecture to handle per-tenant isolation at scale. We will design a system that can manage multiple ingestion pipelines, each tailored to its respective tenant’s database. This involves configuring Airbyte or Fivetran connectors for each customer and orchestrating them through a central management platform.

First, we will define the high-level components of our architecture:

Airbyte/Fivetran Connectors: These will be configured with specific settings for each tenant.
Data Ingestion Pipeline: This pipeline will handle data extraction, transformation, and loading (ETL) operations for each tenant.
Central Management Platform: A system to manage and orchestrate the connectors and pipelines, ensuring security and isolation.

Now let's break down the solution step-by-step:

#### Triage

The question is essentially asking how to build a scalable data ingestion architecture that ensures strict per-tenant isolation while leveraging Airbyte or Fivetran for its integration capabilities. The key aspects include:

Scalability: Handling 500 tenants without compromising performance.
Security: Ensuring no cross-tenant data leakage.
Configuration Management: Managing multiple configurations efficiently.

#### Assumptions
We are using Airbyte, but the solution can be adapted for Fivetran.
Each tenant has an identical schema in their database.
Data needs to be ingested into a central data warehouse (e.g., Snowflake).
The system must support real-time and batch ingestion.

#### First Principles Solution
1. Airbyte/Fivetran Connector Configuration
Per-Tenant Connectors: We will configure one Airbyte/Fivetran connector for each tenant, ensuring that they are isolated from each other.
Tenant-Specific Settings: Each connector will have its own source database connection settings and target data warehouse schema.
2. Data Ingestion Pipeline
Extract Phase: Use the configured connectors to extract data from each tenant’s database.
Transform Phase: Apply necessary transformations (e.g., date standardization, data type conversions) if needed.
Load Phase: Load the transformed data into a central data warehouse with proper schema mappings.
3. Central Management Platform
Configuration Management: A system to manage and deploy configurations for each tenant’s connector. This can be implemented using CI/CD pipelines or Kubernetes deployments.
Orchestration Layer: An orchestration layer that manages the execution of each pipeline, ensuring they run independently and do not interfere with each other.
Detailed Implementation
1. Airbyte/Fivetran Connector Configuration
JSON
{
  "name": "tenant-a-connector",
  "source_type": "database",
  "target_type": "data_warehouse",
  "source_config": {
    "host": "tenant-a-db.example.com",
    "port": 3306,
    "user": "airbyte",
    "password": "airbyte-secret"
  },
  "target_schema": "tenant_a_data"
}
2. Data Ingestion Pipeline
Pipeline Definition: Define a pipeline in Airbyte or Fivetran to handle the ETL process.
Transformation Steps: Use transformation steps like date_conversion, data_type_casting, etc., if needed.
3. Central Management Platform
Kubernetes Deployment: Deploy each connector as a separate Kubernetes pod, ensuring isolation through namespaces and network policies.
CI/CD Pipeline: Automate the deployment of connectors using GitOps principles with Helm charts or Kubernetes manifests.

#### Closing Summary

By leveraging Airbyte/Fivetran for its integration capabilities and implementing a microservices architecture with proper configuration management and orchestration, we can achieve scalable and secure data ingestion from 500 customers. The key is to ensure each tenant’s data is processed independently and securely without any cross-tenant leakage.

#### Common Mistakes
and Follow-Ups
Not Implementing Proper Isolation: Failing to use namespaces or network policies in Kubernetes can lead to cross-tenant data exposure.
Lack of Orchestration Layer: Without an orchestration layer, managing multiple connectors can become complex and error-prone.
Ignoring Security Best Practices: Not implementing secure connection settings or using strong secrets management practices.

**Comparison:**

```
Tenant	Source Database	Target Schema
tenant-a	tenant-a-db.example.com	tenant_a_data
...	...	...
```

**Code:**

```python
# Example Kubernetes Deployment for a Tenant Connector
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tenant-a-connector
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tenant-a-connector
  template:
    metadata:
      labels:
        app: tenant-a-connector
    spec:
      containers:
      - name: airbyte-connector
        image: airbyte/airbyte:latest
        env:
        - name: AIRBYTE_SOURCE_HOST
          value: "tenant-a-db.example.com"
        - name: AIRBYTE_TARGET_SCHEMA
          value: "tenant_a_data"
---
# Example Service for Tenant Connector
apiVersion: v1
kind: Service
metadata:
  name: tenant-a-connector-service
spec:
  selector:
    app: tenant-a-connector
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
# Example Network Policy for Isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-tenant-a
spec:
  podSelector:
    matchLabels:
      app: tenant-a-connector
  policyTypes:
  - Ingress
  - Egress
---
```

**Example input / output:**

```text
Input:
- Tenant configurations stored in a YAML or JSON file

Output:
- Data from each tenant ingested and loaded into the central data warehouse
- Logs and metrics for each pipeline execution
```

**Data:**

```json
{
  "tenant_configs": [
    {
      "name": "tenant-a",
      "source_db": "tenant-a-db.example.com",
      "target_schema": "tenant_a_data"
    },
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Airbyte/Fivetran Connector
4. Isolation?
5. Kubernetes Pod withNamespace
6. Shared Resources withNetwork Policy
7. Ingestion Pipeline
8. Ingestion Pipeline
9. Data Warehouse

