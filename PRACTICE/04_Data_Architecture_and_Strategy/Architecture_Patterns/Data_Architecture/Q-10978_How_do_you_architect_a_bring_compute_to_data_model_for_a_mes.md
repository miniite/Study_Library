---
type: "practice"
difficulty: "Deep"
---

# How do you architect a 'bring compute to data' model for a mesh where domains keep their data products in their own accounts/regions, and consumers query cross-account? What are the IAM, cost-attribution, and network-egress traps that make this harder than it looks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-domain Joins in a Mesh](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Cross-domain_Joins_in_a_Mesh.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a 'bring compute to data' model within a mesh architecture, where domains maintain their own data products in separate accounts/regions, and consumers query across these boundaries, the key is to balance local processing capabilities with cross-account coordination while addressing complex IAM, cost attribution, and network egress challenges. The core idea is to leverage serverless or managed compute services that can execute queries locally before aggregating results. Let’s break down how this architecture works from first principles.

#### Triage

The question is asking for a detailed data architecture design that supports distributed data processing where domains (e.g., teams, departments) keep their data products in their own AWS accounts/regions. The challenge lies in enabling consumers to query and aggregate this data across multiple accounts while ensuring security, cost efficiency, and network optimization.

#### Assumptions
Each domain has its own AWS account and regions.
There is a central control plane or orchestrator that manages cross-account operations.

#### Cons
umers are typically part of different domains but need access to aggregate data from other domains.
Step-by-Step Solution
Data Storage Layer:
Use Amazon S3, Redshift Spectrum, or Glue Data Catalogs for storing and indexing data across regions/accounts. These services can be used in each domain’s account.
Compute Layer:
Utilize AWS Lambda or managed compute services like Amazon Athena, Glue, or custom ETL jobs to process and aggregate data locally within each region/account. This reduces network latency and cost of data movement.
Cross-Account Access and Orchestration:
Implement cross-account roles using AWS IAM and STS (Security Token Service) for secure data access between domains.
Cost Attribution and Management:
Use AWS Cost Explorer or budgeting services to track costs across multiple accounts and regions.
Network Egress Optimization:
Minimize network egress by processing as much logic locally as possible, reducing the need for frequent cross-account data transfers.

#### Implementation
Example

Data Products in Separate Accounts:

PLAINTEXT
Account A: Domain 1 (Data Product 1)
Account B: Domain 2 (Data Product 2)

Central Orchestration and IAM Design:

Create a central AWS account for orchestrating cross-account queries.
Use AWS STS to assume roles in each domain’s account for secure data access.

Local Processing with Lambda or Managed Services:

PLAINTEXT
Account A:
  - Lambda function 1 (Process Data Product 1)
Account B:
  - Lambda function 2 (Process Data Product 2)

Cross-Account Query Execution:

Use the central orchestrator to initiate queries in each domain’s account.
Each domain’s Lambda function aggregates and returns processed data to the orchestrator.

Cost Tracking:

PLAINTEXT
Account A: $100 (Lambda, S3)
Account B: $80 (Lambda, S3)
Central Orchestrator: $20 (API Gateway, Lambda)

#### Closing Summary

By structuring the architecture to 'bring compute to data,' you minimize network latency and optimize costs. The key is to leverage local processing capabilities in each domain while ensuring secure cross-account access through IAM roles and STS. This design also simplifies cost management by tracking expenses at an account level.

#### Common Mistakes/Follow-Ups
Traps in IAM: Forgetting to set up the right IAM policies and roles can lead to security breaches or unauthorized access.
Cost Overruns: Not properly managing costs through budgeting and optimization tools can result in unexpected charges.
Network Egress: Failing to optimize network egress by processing as much data locally can increase data transfer costs.

**Comparison:**

| Account | Regions        | Data Products                |
|---------|----------------|------------------------------|
| A       | us-east-1, eu-west-1 | Data Product 1               |
| B       | ap-southeast-1   | Data Product 2               |

**Code:**

```python
# Example Lambda function for local processing
def process_data(event, context):
    # Process data from S3 bucket
    s3 = boto3.client('s3')
    object_key = event['Records'][0]['s3']['object']['key']
    file_content = s3.get_object(Bucket='data-bucket', Key=object_key)['Body'].read()

    # Local processing logic
    processed_data = process_file(file_content)

    return {
        'statusCode': 200,
        'body': processed_data
    }
```

**Example input / output:**

```text
Input:
Account A -> Lambda function 1 (Process Data Product 1)
Account B -> Lambda function 2 (Process Data Product 2)

Output:
Central Orchestrator -> Aggregated processed data from both domains
```

**Data:**

```json
{
  "accounts": [
    {"id": "A", "regions": ["us-east-1", "eu-west-1"]},
    {"id": "B", "regions": ["ap-southeast-1"]}
  ],
  "data_products": [
    {"account_id": "A", "name": "Data Product 1"},
    {"account_id": "B", "name": "Data Product 2"}
  ]
}
```

**Flow:**

1. Data Product Processing
2. Process Data
3. Aggregated
4. Cross-Account Query
5. Initiate Queries
6. Process Data
7. Central Orchestrator
8. Lambda B
9. Output
10. Lambda A
11. Output
12. Central Store/Orchestrator

