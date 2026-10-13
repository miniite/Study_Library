---
type: "practice"
difficulty: "Basic"
---

# What is Snowflake and how is it different from a traditional on-premise data warehouse?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Snowflake Compute and Caching](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Compute_and_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Snowflake is a cloud-based, multi-tenant, shared-nothing architecture designed to provide scalable and secure data warehousing services. It differs significantly from traditional on-premise data warehouses in terms of its deployment model, scalability, and cost structure.

To start with, Snowflake operates entirely in the cloud, eliminating the need for expensive hardware investments and physical server maintenance. This cloud-first approach allows for dynamic resource allocation and easy scaling up or down based on demand, which is a major advantage over traditional data warehouses that require pre-provisioned resources and can be costly to scale.

The key concept of Snowflake's architecture involves separating compute and storage, allowing them to scale independently. In contrast, traditional data warehouses typically bundle these functionalities together, limiting their scalability. By decoupling storage and computing layers, Snowflake ensures optimal performance without the need for large upfront investments in hardware.

Moreover, Snowflake supports multi-tenancy, meaning multiple users can share the same infrastructure but operate on isolated data environments. This feature reduces costs compared to traditional solutions where each organization would need its own dedicated environment.

Let's break down these concepts step by step:

Cloud-Based Infrastructure: Unlike traditional data warehouses that require you to buy and maintain your own servers, Snowflake runs on Amazon Web Services (AWS) or Microsoft Azure. This means no physical hardware is required, reducing initial setup costs.
Multi-Tenant Architecture: Snowflake uses a multi-tenant model where different users can use the same infrastructure but have their data isolated from others. This is achieved through logical partitions called "warehouses" and "databases." Each organization gets its own set of compute resources to process queries, ensuring isolation and security.
Separate Compute and Storage: In Snowflake, storage (where your data resides) and computing (processing power for running queries) can be scaled independently. This flexibility means you pay only for the resources you use when running queries. Traditional data warehouses often require you to over-provision compute resources to handle peak loads.
Shared-Nothing Architecture: Each node in Snowflake's architecture handles a specific task, and nodes are completely independent of each other. This design ensures high availability and reduces bottlenecks common in shared-everything architectures.

In summary, Snowflake offers a flexible, cost-effective, and scalable solution compared to traditional data warehouses by leveraging cloud infrastructure, multi-tenancy, separation of storage and compute, and a shared-nothing architecture. Understanding these aspects will help you appreciate how Snowflake can transform the way organizations manage their data warehousing needs.

#### Common mistakes
often include confusing the multi-tenant nature with single-tenant environments or misunderstanding that Snowflake's scalability is independent for compute and storage.

**Code:**

```python
# Example code snippet to illustrate creating a new database in Snowflake
import snowflake.connector

def create_database():
    conn = snowflake.connector.connect(
        user='your_user',
        password='your_password',
        account='your_account'
    )
    cursor = conn.cursor()

    # Create a new database named "analytics_db"
    cursor.execute("CREATE DATABASE analytics_db")
    print("Database created successfully.")

create_database()
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `CREATE DATABASE analytics_db;`<br>`USE WAREHOUSE prod_wh;`<br>`CREATE TABLE sales_data (id INTEGER, date DATE, amount FLOAT);` | Database `analytics_db` created successfully.<br>Warehouse `prod_wh` is ready for use.<br>Table `sales_data` created in schema `sales`. |

**Data:**

```json
{
  "database": "analytics_db",
  "warehouses": {
    "dev_wh": {
      "type": "compute-2xlarge",
      "auto_sizing": true,
      "min_compute_units": 10
    },
    "prod_wh": {
      "type": "compute-xlarge",
      "auto_sizing": false,
      "min_compute_units": 5
    }
  },
  "tables": [
    {
      "name": "sales_data",
      "schema": "sales",
      "columns": [
        {"name": "id", "data_type": "INTEGER"},
        {"name": "date", "data_type": "DATE"},
        {"name": "amount", "data_type": "FLOAT"}
      ]
    }
  ]
}
```

