---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a source and a destination (connector) in Airbyte or Fivetran?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Data Engineering, especially when dealing with tools like Airbyte or Fivetran, understanding the roles of sources and destinations is crucial for building robust data pipelines. Let’s break down these concepts:

TL;DR:

Sources in Airbyte/Fivetran are where your data comes from, such as databases, APIs, or file systems. Destinations are where that data gets stored, like warehouses or databases.

#### Triage

The question is asking for the distinction between source and destination connectors in tools like Airbyte or Fivetran. These terms refer to the parts of a data integration pipeline responsible for data extraction and loading respectively.

#### First-Principles Solution

Sources (Connectors):

These are the components that extract data from various sources.
They can be configured with specific parameters to connect to different types of systems, such as databases, file servers, or APIs.

#### Example
s include MySQL, PostgreSQL, Salesforce, Dropbox, and Google Drive.

Destinations (Connectors):

These are the components that ingest data from sources and load it into target storage locations.
Destinations can be configured to write data to various types of databases, file systems, or cloud warehouses.

#### Example
s include PostgreSQL, Redshift, BigQuery, and Snowflake.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, the primary difference between a source and destination connector lies in their role within the pipeline. Sources are responsible for fetching data from various origins, while destinations focus on storing or transforming that data into a desired format. A common mistake is not properly configuring these connectors to ensure data integrity and consistency.

#### Common Mistakes
Inadequate Configuration: Not setting up proper authentication or connection details can lead to failures.
Incorrect Mapping: Mismatching the structure of source and destination data models might result in incomplete or erroneous data loading.
Performance Issues: Choosing an inappropriate connector for high-volume data sources may cause performance bottlenecks.

#### Follow-Ups
Can you describe a scenario where using Fivetran instead of Airbyte would be more advantageous?
How do these connectors handle data transformation if required?

**Comparison:**

```
Sources	Destinations
MySQL	BigQuery
PostgreSQL	Redshift
Salesforce	Snowflake
Dropbox	S3
```

**Code:**

```text
# Example configuration snippets
source_config = {
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "password"
}

destination_config = {
    "type": "bigquery",
    "project_id": "my-project-id",
    "dataset_id": "my-dataset",
}
```

**Example input / output:**

```text
Input:
- Source connector for MySQL with table `orders`
- Destination connector for BigQuery

Output:
- Data from orders in MySQL is successfully loaded into the `orders` table in BigQuery.
```

**Data:**

```json
{
  "sources": [
    {
      "name": "MySQL Source",
      "type": "mysql",
      "connection_details": {
        "host": "localhost",
        "port": 3306,
        "user": "root",
        "password": "password"
      }
    },
    {
      "name": "Salesforce Source",
      "type": "salesforce",
      "connection_details": {
        "username": "admin@example.com",
        "password": "password!234567890",
        "security_token": "token123"
      }
    }
  ],
  "destinations": [
    {
      "name": "BigQuery Destination",
      "type": "bigquery",
      "connection_details": {
        "project_id": "my-project-id",
        "dataset_id": "my-dataset"
      }
    },
    {
      "name": "PostgreSQL Destination",
      "type": "postgresql",
      "connection_details": {
        "host": "localhost",
        "port": 5432,
        "user": "admin",
        "password": "password123"
      }
    }
  ]
}
```

**Data:**

```
Data Storage
Source Connector
Data Extraction
Deduplication
Destination Connector
Target Storage
```

