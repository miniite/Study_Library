---
type: "practice"
difficulty: "Intermediate"
---

# Your Salesforce connector fell behind by two days because of a schema change. Walk me through recovery so downstream SLAs are met.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To recover from the delay caused by a schema change in the Salesforce connector, we need to address both the immediate resynchronization and the long-term reliability of data ingestion. At an intermediate level, this involves understanding the current state of our pipeline, communicating effectively with stakeholders, and implementing robust recovery steps.

Firstly, the question is probing your ability to handle unexpected issues in data pipelines, particularly focusing on how you manage schema changes that can disrupt ongoing processes. The core idea here is to ensure that we restore the integrity of the data flow as quickly as possible while maintaining the reliability of our downstream systems.

Triaging the Problem

#### Assumptions

We are using Airbyte or Fivetran for Salesforce ingestion.
There has been a recent schema change in Salesforce that caused delays.
The delay is two days and affects the entire data pipeline.
Downstream systems have SLAs that must be met.

Steps to recovery involve several key actions:

Understand the Impact: Identify which tables or schemas were affected by the delay.
Resynchronize the Data Ingestion Pipeline: Ensure data is pulled up-to-date from Salesforce.
Update Schema Mappings: Adjust mappings in Airbyte/Fivetran to reflect the latest schema changes.
Implement Monitoring and Alerts: Set up proactive monitoring to prevent future issues.

#### First-Principles Solution

Identify Affected Data:

Use logs from Airbyte or Fivetran to pinpoint which tables or schemas have not been updated in two days.
Check the Salesforce schema version history for when the changes occurred.

Resynchronize Data Ingestion:

Re-run the Airbyte/Fivetran job that was paused due to the delay.
Ensure the job pulls data from the point of change, effectively backfilling the missing days.

Update Schema Mappings:

Review and update the schema mappings in the Airbyte configuration or Fivetran catalog view.
Use the latest schema definitions to ensure accurate data transformation and storage.

Implement Monitoring and Alerts:

Set up monitoring for any changes in Salesforce schema going forward.
Integrate with Slack, email, or other alerting systems to notify relevant stakeholders of potential issues early.

#### Closing Summary
+ Common Mistakes

To ensure that downstream SLAs are met after a schema change delay, it’s crucial to take proactive steps. The key is not just to recover the lost data but also to improve processes to prevent future delays. Common mistakes include failing to update mappings correctly, overlooking dependent systems, or neglecting to implement robust monitoring.

#### Example
Workflow

Let's walk through an example:

Suppose a SalesforceOpportunity schema change occurred.
Re-run the Airbyte job to backfill data for those two days.
Update the schema mapping in Airbyte to reflect the new fields (e.g., add new columns).
Monitor the Salesforce schema version and set up alerts for future changes.

By following these steps, we can ensure that our data pipeline remains reliable and meets SLAs even when faced with unexpected disruptions like schema changes.

**Comparison:**

| Schema Change | Timestamp       | Affected Tables     |
|---------------|-----------------|--------------------|
| Opportunity   | 2023-10-05      | SalesforceOpportunity, SalesTeam |
| Task          | 2023-10-06      | SalesforceTask      |

**Code:**

```python
# Example Python code to re-run Airbyte job
def run_airbyte_job():
    import airbyte_api_client as airbyte

    # Initialize Airbyte client
    client = airbyte.Client(api_token="YOUR_API_TOKEN")

    # Re-run the specific job that was paused
    job_id = "PAUSED_JOB_ID"
    response = client.run_job(job_id)

    return response.status

# Example function to update schema mappings in Fivetran
def update_fivetran_mapping():
    import fivetran_api_client as fivetran

    # Initialize Fivetran client
    client = fivetran.Client(api_token="YOUR_API_TOKEN")

    # Update the mapping for SalesforceOpportunity
    job_id = "SALESFORCE_OPPORTUNITY_JOB_ID"
    new_schema = {"new_field": "INTEGER"}
    response = client.update_mapping(job_id, new_schema)

    return response.status
```

**Example input / output:**

```text
Input:
- Job ID for Airbyte: PAUSED_JOB_ID
- New schema mapping details: {"table_name": "SalesforceOpportunity", "new_field": "INTEGER"}

Output:
- Airbyte job status updated to RUNNING.
- Fivetran job successfully updated with new schema mappings.
```

**Data:**

```json
{
  "job_status": {
    "job_id": "PAUSED_JOB_ID",
    "status": "RUNNING"
  },
  "schema_update": {
    "table_name": "SalesforceOpportunity",
    "new_field": "INTEGER"
  }
}
```

**Flow:**

1. Identify Affected Data
2. Resynchronize Airbyte Job
3. Update Fivetran SchemaMappings
4. Implement Monitoring andAlerts
5. Monitor for Future Changes

