---
type: "practice"
difficulty: "Deep"
---

# How do you version and promote connector configuration changes across dev, staging, and prod without manual clicking in the UI?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Sync and Connector Migration](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Incremental_Sync_and_Connector_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure smooth and automated versioning and promotion of Airbyte or Fivetran connector configurations from development (dev) to staging to production (prod), we need a well-defined CI/CD pipeline that handles configuration changes without manual UI interaction. The core idea is to manage these changes through code, leveraging tools like Git for version control and Jenkins or similar orchestration systems to automate the deployment process.

**Code:**

```python
# Example Python script using Airbyte SDK to update connector configurations
import airbyte_api_client
from airbyte_api_client.models import ConfiguredAirbyteConnector

def update_airbyte_config(connector_id, config_dict):
    client = airbyte_api_client.ApiClient()
    connectors_api = airbyte_api_client.ConnectorsApi(client)

    # Fetch and update the existing configuration
    connector = connectors_api.get_connector_by_id(connector_id)
    updated_config = ConfiguredAirbyteConnector(
        id=connector.id,
        name=connector.name,
        type=connector.type,
        destination_definition=connector.destination_definition,
        source_definition=connector.source_definition,
        config=config_dict  # Updated configuration
    )

    response = connectors_api.update_connector_by_id(connector_id, updated_config)
    return response

def promote_configs_to_environment(env):
    """Promote connector configurations to a specified environment."""
    env_configs = fetch_env_configs(env)  # Assume this function fetches the config from Git or DB

    for config in env_configs:
        update_airbyte_config(config["id"], config["config"])

# Example usage
promote_configs_to_environment("staging")
```

**Example input / output:**

```text
# Example input and output of promoting configurations to staging
input_configs = {
    "dev_configs": [
        {"id": "12345", "config": {"field1": "value1", "field2": "value2"}},
        {"id": "67890", "config": {"fieldA": "valueA", "fieldB": "valueB"}}
    ]
}

promote_configs_to_environment("staging")
output_configs = {
    "staging_configs": [
        {"id": "12345", "config": {"field1": "updated_value1", "field2": "updated_value2"}},
        {"id": "67890", "config": {"fieldA": "updated_valueA", "fieldB": "updated_valueB"}}
    ]
}
```

**Data:**

```json
{
  "dev_configs": [
    {
      "id": "12345",
      "config": {"field1": "value1", "field2": "value2"}
    },
    {
      "id": "67890",
      "config": {"fieldA": "valueA", "fieldB": "valueB"}
    }
  ],
  "staging_configs": [
    {
      "id": "12345",
      "config": {"field1": "updated_value1", "field2": "updated_value2"}
    },
    {
      "id": "67890",
      "config": {"fieldA": "updated_valueA", "fieldB": "updated_valueB"}
    }
  ],
  "prod_configs": [
    {
      "id": "12345",
      "config": {"field1": "final_value1", "field2": "final_value2"}
    },
    {
      "id": "67890",
      "config": {"fieldA": "final_valueA", "fieldB": "final_valueB"}
    }
  ]
}
```

