---
type: "practice"
difficulty: "Expert"
---

# How do you handle the 'catalog of catalogs' problem — a global data-discovery experience over Unity Catalog + Glue + BigQuery + Purview — without building a fragile crawler that's always stale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cloud Portability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Multi-cloud_Portability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When addressing the "catalog of catalogs" problem in a multi-cloud environment, the goal is to create a unified global data-discovery experience across different data lakes (Unity Catalog, Glue, BigQuery) and metadata repositories (Purview). This involves maintaining a consistent view of all datasets without relying on a fragile crawler that can become stale. The key challenge here is ensuring real-time or near-real-time synchronization of metadata while keeping the architecture robust.

The "catalog of catalogs" problem arises because each data lake and metadata repository has its own schema for storing and organizing metadata, which can lead to discrepancies and inconsistencies if not managed properly. This complexity requires a strategic approach that leverages both automation and governance frameworks to ensure data visibility and control across cloud environments.

To solve this problem, we need to design an architecture that integrates these disparate systems through a centralized metadata management layer while maintaining separation of concerns and minimizing the reliance on single points of failure.

**Comparison:**

```
Metadata Source	Table Name	Schema
Unity Catalog	sales	[id, amount]
Glue	customer	[id, email]
BigQuery	revenue	[month, amount]
```

**Code:**

```python
# Example of a simplified Python script for initial synchronization
import boto3
from google.cloud import bigquery
from azure.purview.catalog import CatalogResourceClient

def sync_metadata():
    # Initialize clients for Unity Catalog, Glue, BigQuery, and Purview
    unity_catalog_client = boto3.client('lakeformation')
    glue_client = boto3.client('glue')
    bigquery_client = bigquery.Client()
    purview_client = CatalogResourceClient("https://your-purview-account-name.azuredatabricks.net")

    # Synchronize metadata from Unity Catalog
    unity_catalog_response = unity_catalog_client.get_tables(DatabaseName='default')
    for table in unity_catalog_response['TableList']:
        purview_client.create_table(name=table['name'], type='hive', schema=[field['name'] for field in table['columns']])

    # Synchronize metadata from Glue
    glue_tables = glue_client.get_tables(DatabaseName='default')
    for table in glue_tables['Tables']:
        purview_client.create_table(name=table['Name'], type='hive', schema=[column['Name'] for column in table['StorageDescriptor']['Columns']])

    # Synchronize metadata from BigQuery
    bigquery_datasets = bigquery_client.list_datasets()
    for dataset in bigquery_datasets:
        for table in bigquery_client.list_tables(dataset.dataset_id):
            purview_client.create_table(name=table.table_id, type='bigquery', schema=[field.name for field in table.schema])

# Call the function to start synchronization
sync_metadata()
```

**Example input / output:**

```text
Input:
- Unity Catalog metadata for tables in the 'default' database.
- Glue catalog metadata for tables in the 'default' database.
- BigQuery dataset and table schema information.

Output:
- Purview metadata repository with synchronized entries from all sources.
```

**Data:**

```json
{
  "unity_catalog": {
    "databaseName": "default",
    "tables": [
      {"name": "sales", "columns": [{"name": "id"}, {"name": "amount"}]},
      {"name": "inventory", "columns": [{"name": "sku"}, {"name": "quantity"}]}
    ]
  },
  "glue": {
    "databaseName": "default",
    "tables": [
      {"name": "customer", "columns": [{"name": "id"}, {"name": "email"}]},
      {"name": "orders", "columns": [{"name": "order_id"}, {"name": "customer_id"}]}
    ]
  },
  "bigquery": {
    "datasets": [
      {"datasetId": "sales_data", "tables": [{"tableId": "revenue", "schema": ["month", "amount"]}]}
    ]
  }
}
```

**Flow:**

1. Unity Catalog Sync
2. Purview
3. Glue Sync
4. BigQuery Sync
5. Catalog Synchronization Layer

