---
type: "practice"
difficulty: "Deep"
---

# How would you architect the company's approach to 'availability zone as a failure domain' so that teams stop accidentally coupling their reliability to a single AZ they didn't even know they depended on?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Availability Zone Resilience](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of ensuring that teams do not inadvertently couple their systems to a specific Availability Zone (AZ) while maintaining high availability, we need to architect our infrastructure with robust principles and automated checks. The goal is to ensure that critical components are designed to be AZ-agnostic, thereby reducing the risk of outages due to single points of failure. This involves several steps: designing microservices architecture, leveraging cloud-native services, implementing monitoring and alerting, and ensuring failover mechanisms.

At a high level, the solution revolves around these key principles:

Microservices Design: Ensuring that each service can operate independently within any AZ.
Load Balancing and Auto-scaling: Utilizing managed load balancers to distribute traffic across multiple AZs.
Database Resiliency: Implementing multi-AZ databases or read replicas for high availability.
Monitoring and Alerting: Setting up comprehensive monitoring to detect issues early and automated alerts to notify relevant teams.

**Comparison:**

| Service | Availability Zones | Auto-scaling Enabled? |
|---------|-------------------|----------------------|
| App1    | us-east-1a, us-east-1b | Yes                  |
| DB      | Multi-AZ           | Yes                  |
| Cache   | Single AZ          | No                   |

**Code:**

```python
# Example of a simple microservice function in Python
def process_request(request_data):
    # Simulate processing logic
    processed_data = transform_data(request_data)
    return save_to_dynamodb(processed_data)

def transform_data(data):
    # Transformation logic here
    return data

def save_to_dynamodb(item):
    # Connect to a multi-AZ DynamoDB table and insert the item
    dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    table = dynamodb.Table('MyTable')
    response = table.put_item(Item=item)
    return response
```

**Example input / output:**

```text
Request to process_data:
Input: {"data": "sample input"}
Output: {"status": "success", "processed_data": "transformed sample input"}

Monitor alert for high CPU usage on a specific instance in an AZ:
Request: CheckCPUUsage("i-0abcdef1234567890")
Response: Alert: High CPU Usage detected; instance i-0abcdef1234567890 is overloaded.
```

**Data:**

```json
{
  "db_config": {
    "region": "us-east-1",
    "table": "MyTable"
  },
  "load_balancer": {
    "listeners": [
      { "protocol": "HTTP", "port": 80 }
    ],
    "instances": ["i-0abcdef1234567890"]
  }
}
```

**Flow:**

1. User Request
2. Load Balancer Route53
3. [Send to us-east-1a or -1b]
4. D
5. Process Data in us-east-1a
6. E
7. Process Data in us-east-1b
8. F,G
9. Store Data in Multi-AZ DB
10. Auto-scaling Triggered forIncreased Capacity

