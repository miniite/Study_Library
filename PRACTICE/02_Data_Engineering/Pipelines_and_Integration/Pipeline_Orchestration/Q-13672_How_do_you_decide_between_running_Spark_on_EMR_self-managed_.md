---
type: "practice"
difficulty: "Deep"
---

# How do you decide between running Spark on EMR self-managed clusters versus EMR Serverless or Databricks jobs compute for a bursty daily batch workload?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Profiling Spark Stages](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between running Spark on EMR self-managed clusters, EMR Serverless, or Databricks jobs for a bursty daily batch workload, you need to weigh multiple factors including cost, scalability, ease of management, and performance. In this scenario, the primary goal is to handle a variable but predictable load with a focus on efficiency and cost-effectiveness.

First, let's break down what each option offers:

EMR Self-managed Clusters: You have full control over cluster creation, termination, and configuration. This includes flexibility in customizing the environment.
EMR Serverless: A fully managed service that automatically scales based on job demand without manual intervention for cluster management.
Databricks Jobs Compute: Part of Databricks' platform, offering a highly scalable compute environment with automated cluster management.

#### Triage
What Are We Probing?

The interviewer is likely probing your understanding of:

Cost Management: Bursty workloads can be expensive if over-provisioned.
Scalability and Performance: Ensuring the system can handle peak loads without compromising performance.
Operational Complexity: Managing clusters versus leveraging managed services.

#### First-principles Solution
1. Cost Considerations
EMR Self-managed Clusters: You need to manually provision and manage resources, which can lead to higher upfront costs and operational overhead. However, you have full control over resource allocation.
EMR Serverless: Predictable pricing with no idle time charges when the job is not running, making it cost-effective for bursty workloads.
Databricks Jobs Compute: Also offers auto-scaling but integrates tightly with Databricks' broader platform, which might come at a premium.
2. Scalability and Performance
EMR Self-managed Clusters: High control over the environment means you can optimize for specific workloads, but managing clusters can be complex.
EMR Serverless: Highly scalable by design with automatic resource adjustments based on job demands, ensuring efficient use of compute resources.
Databricks Jobs Compute: Also highly scalable and integrates seamlessly with Databricks' ecosystem, making it easy to manage and monitor jobs.
3. Operational Complexity
EMR Self-managed Clusters: Requires detailed cluster management, monitoring, and optimization. This can be time-consuming.
EMR Serverless: Reduces operational complexity significantly as AWS manages the underlying infrastructure.
Databricks Jobs Compute: Offers a more integrated approach with Databricks, simplifying job execution and management.

#### Example Scenario

Assuming you have a daily batch workload that varies in size but is generally predictable:

If your team has experience managing EMR clusters and prefers full control, EMR self-managed clusters might be the best choice.
For a smaller team or if cost efficiency is paramount, EMR Serverless would likely offer better value with minimal setup overhead.
If you are already using Databricks for other projects and want to leverage existing tools, Databricks Jobs Compute could provide a seamless experience.

#### Closing Summary

In conclusion, the decision between EMR self-managed clusters, EMR Serverless, or Databricks jobs compute depends on your specific needs, team expertise, and cost considerations. Each option has its trade-offs in terms of control, scalability, and operational complexity. For bursty workloads, EMR Serverless often stands out due to its cost-effectiveness and ease of use.

#### Common Mistakes/Follow-ups
Overlooking the total cost of ownership (TCO) by focusing solely on initial setup costs.
Ignoring the learning curve and potential support issues with less managed services like EMR self-managed clusters.
Underestimating the operational burden in environments with limited technical expertise.

**Comparison:**

```
Cluster Type	Cost Management	Scalability	Operational Complexity
EMR Self-managed Clusters	High (manual management)	High (flexible configuration)	High (resource monitoring and optimization required)
EMR Serverless	Low (auto-scaling, no idle time charges)	High (auto-scaling)	Low (minimal setup and management)
Databricks Jobs Compute	Moderate (integration with Databricks platform)	High (auto-scaling)	Low (seamless job execution and monitoring)
```

**Code:**

```python
# Example code for initializing an EMR Serverless job
from botocore.client import Config

session = boto3.Session(
    aws_access_key_id='YOUR_ACCESS_KEY',
    aws_secret_access_key='YOUR_SECRET_KEY',
    region_name='us-west-2'
)

emr_client = session.client('emr', config=Config(signature_version='s3v4'))

response = emr_client.create_job_flow(
    Name='DailyBatchJob',
    ReleaseLabel='emr-5.30.1',
    Steps=[
        {
            'Name': 'Spark Job',
            'ActionOnFailure': 'CONTINUE',
            'HadoopJarStep': {
                'Jar': 'command-runner.jar',
                'Args': ['spark-submit', '/path/to/spark-job.py']
            }
        }
    ],
    Instances={
        'InstanceGroups': [
            {
                'Name': 'Master node',
                'Market': 'ON_DEMAND',
                'InstanceType': 'm5.xlarge',
                'Role': 'MASTER'
            },
            {
                'Name': 'Core nodes',
                'Market': 'SPOT',
                'InstanceType': 'm5.2xlarge',
                'Role': 'CORE'
            }
        ],
        'KeepJobFlowAliveWhenNoSteps': True,
        'TerminationProtected': False
    }
)
print(response['JobFlowId'])
```

**Example input / output:**

```text
Input:
- A daily batch job that processes a varying amount of data each day.
- The job runs on EMR Serverless and utilizes Spark for processing.

Output:
- Job execution logs indicating successful completion or errors.
- Data processed and stored in S3 or another target location.
```

**Data:**

```json
{
  "job_flow_id": "j-1234567890abcdef",
  "name": "DailyBatchJob",
  "release_label": "emr-5.30.1",
  "instance_groups": [
    {
      "name": "Master node",
      "market": "ON_DEMAND",
      "instance_type": "m5.xlarge",
      "role": "MASTER"
    },
    {
      "name": "Core nodes",
      "market": "SPOT",
      "instance_type": "m5.2xlarge",
      "role": "CORE"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify WorkloadCharacteristics
6. Is the workload highlyvariable?
7. Consider EMR Serverless
8. Does your team have expertisein managing clusters?
9. Choose EMR Self-managedClusters
10. Consider Databricks JobsCompute
11. Verify Cost-Effectiveness andScalability
12. Monitor Performance andCosts Regularly
13. Utilize Databricks' Integrationsand Support

