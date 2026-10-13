---
type: "practice"
difficulty: "Basic"
---

# How would you estimate the monthly cost of an app before you build it on the cloud?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To estimate the monthly cost of an app before building it in the cloud, we need to consider several factors related to our application’s infrastructure requirements and usage patterns. The core idea is to break down the costs into manageable components, such as compute, storage, databases, networking, and any additional services like monitoring or security. By understanding these components, we can make informed decisions about scaling, optimization, and cost control.

The question is essentially asking you to outline a systematic approach for estimating cloud costs based on the app’s requirements. The interviewer may be probing your ability to think through various cost factors, your familiarity with different AWS services (or other providers), and your understanding of how cloud pricing models work. Some key assumptions include knowing the expected traffic volume, storage needs, data transfer volumes, and any specific features or integrations required.

Let’s walk through this from first principles:

Identify Key Components: List out all the necessary components for your application. For a basic app, you might need:

Compute (EC2 instances)
Storage (S3 buckets)
Databases (RDS, DynamoDB)
Networking and Load Balancing
Monitoring and Logging

Estimate Requirements: Determine the necessary resources for each component based on your application’s expected usage.

Check Pricing Models: Understand the pricing models of these services (hourly, per GB storage, etc.). AWS, for example, offers different pricing tiers depending on usage.

Calculate Costs: Use online calculators or tools provided by cloud providers to get cost estimates.

#### Cons
ider Optimization and Scaling: Think about how you might scale your application and what that could mean in terms of increased costs.

For a basic app, let’s break it down step-by-step:

Compute: Assume an average monthly usage of 20 hours per day with 4 EC2 instances.

Cost Calculation: AWS T2 Micro instance at $0.0068 per hour. [ \text{Total Cost} = 20 , \text{hours/day} \times 30 , \text{days/month} \times 4 , \text{instances} \times $0.0068 \approx $16.32 ]

Storage: Estimate 50 GB of S3 storage for backups.

Cost Calculation: AWS S3 Standard at $0.023 per GB per month. [ \text{Total Cost} = 50 , \text{GB} \times $0.023 \approx $1.15 ]

Database: Use DynamoDB with an estimated write capacity of 10,000 writes per second.

Cost Calculation: AWS DynamoDB at $0.75 per 1 million writes. [ \text{Total Cost} = \frac{10,000 \times 30 \times 30}{1,000,000} \times $0.75 \approx $2.25 ]

Networking: Estimate 10 GB of data transfer per day.

Cost Calculation: AWS Data Transfer at $0.09 per GB for egress (outbound). [ \text{Total Cost} = 10 , \text{GB/day} \times 30 , \text{days/month} \times $0.09 \approx $27 ]

Monitoring and Logging: Use CloudWatch for basic monitoring.

Cost Calculation: AWS CloudWatch at $0.0004 per metric per second (very low cost for basic use). [ \text{Total Cost} = 30 , \text{days/month} \times 24 , \text{hours/day} \times 60 , \text{minutes/hour} \times 60 , \text{seconds/minute} \times $0.0004 \approx $1.73 ]

By summing these up, the total estimated monthly cost for a basic app is approximately: [ $16.32 + $1.15 + $2.25 + $27 + $1.73 = $48.45 ]

This is a simplified example, and real-world costs will vary based on actual usage patterns.

**Comparison:**

| Component    | Usage                      | Estimated Cost |
|--------------|----------------------------|----------------|
| EC2          | 20 hours/day, 30 days/month, 4 instances | $16.32        |
| S3           | 50 GB storage              | $1.15         |
| DynamoDB     | 10,000 writes/second       | $2.25         |
| Data Transfer| 10 GB egress/day          | $27           |
| CloudWatch   | Basic monitoring          | $1.73         |
| **Total**    |                            | **$48.45**    |

**Code:**

```python
# Example Python code to estimate monthly cloud cost

def calculate_ec2_cost(hours_per_day=20, days_in_month=30, instances=4):
    cost = hours_per_day * days_in_month * instances * 0.0068
    return round(cost, 2)

def calculate_s3_cost(storage_gb=50):
    cost = storage_gb * 0.023
    return round(cost, 2)

def calculate_dynamodb_cost(writes_per_second=10000):
    cost = (writes_per_second * days_in_month * hours_per_day * minutes_per_hour * seconds_per_minute) / 1_000_000 * 0.75
    return round(cost, 2)

def calculate_data_transfer_cost(egress_gb=10):
    cost = egress_gb * 0.09
    return round(cost, 2)

def calculate_cloudwatch_cost():
    cost = (days_in_month * hours_per_day * minutes_per_hour * seconds_per_minute) * 0.0004
    return round(cost, 2)
```

**Example input / output:**

```text
Input:
- EC2 Usage: 20 hours/day, 30 days/month, 4 instances
- S3 Storage: 50 GB
- DynamoDB Writes: 10,000 writes/second
- Data Transfer: 10 GB egress/day
- CloudWatch Monitoring

Output:
- EC2 Cost: $16.32
- S3 Cost: $1.15
- DynamoDB Cost: $2.25
- Data Transfer Cost: $27
- CloudWatch Cost: $1.73
- Total Monthly Cost: $48.45
```

**Data:**

```json
{
  "ec2": {
    "hours_per_day": 20,
    "days_in_month": 30,
    "instances": 4,
    "cost_per_hour": 0.0068,
    "total_cost": 16.32
  },
  "s3": {
    "storage_gb": 50,
    "cost_per_gb": 0.023,
    "total_cost": 1.15
  },
  "dynamodb": {
    "writes_per_second": 10000,
    "cost_per_write": 0.75 / 1_000_000,
    "total_cost": 2.25
  },
  "data_transfer": {
    "egress_gb": 10,
    "cost_per_gb": 0.09,
    "total_cost": 27
  },
  "cloudwatch": {
    "days_in_month": 30,
    "hours_per_day": 24,
    "minutes_per_hour": 60,
    "seconds_per_minute": 60,
    "cost_per_second": 0.0004,
    "total_cost": 1.73
  }
}
```

**Flow:**

1. Start
2. Identify Key Components
3. Compute
4. Storage
5. Databases
6. Networking and LoadBalancing
7. Monitoring and Logging
8. Calculate Costs for EachComponent
9. Sum Up All Costs
10. Total Monthly Cost Estimate
11. End

