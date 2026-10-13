---
type: "practice"
difficulty: "Deep"
---

# Why might a managed serverless function be cheaper for a rarely-used internal tool than an always-on VM?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Services vs Self-hosting](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are comparing the cost implications of using a managed serverless function versus an always-on virtual machine (VM) for a rarely-used internal tool. The key idea is that serverless functions offer lower operational costs by scaling down or even stopping execution when no requests are made, whereas VMs must run continuously at a fixed cost.

#### Triage

The question revolves around understanding the economic and operational differences between managed serverless functions and always-on VMs for a sporadically used application. The interviewer is probing your comprehension of cloud fundamentals, specifically how resource utilization impacts costs in serverless architectures versus traditional computing models.

Step-by-Step Solution from First Principles

Cost Structure of Serverless Functions:

Scaling Down: In a managed serverless environment like AWS Lambda or Azure Functions, the execution cost is proportional to the number of invocations and the duration of each invocation.
Zero Downtime: These services can scale down to zero instances when no requests are made, reducing idle costs significantly.

Cost Structure of VMs:

Fixed Cost: Always-on VMs incur a fixed cost for operating systems, hardware resources, and network usage regardless of whether they are actively running or not.
Resource Utilization: Even if the VM is mostly idle, it continues to consume resources, leading to higher costs over time.

Comparison:

For an application that is rarely used, serverless functions can significantly reduce costs by only incurring execution charges when there are actual requests. This contrasts with VMs where you pay for a fixed allocation of resources even during periods of low or no activity.

#### Closing Summary

Managed serverless functions can be more cost-effective than always-on VMs for applications that are sporadically used because they offer dynamic scaling and only charge based on execution time. This makes them ideal for internal tools, batch jobs, and other use cases where resources are not continuously utilized.

#### Common Mistakes/Follow-Ups
Misunderstanding Scalability: One common mistake is assuming that serverless functions require significant upfront configuration or maintenance compared to VMs.
Ignoring Warm-Up Time: For applications with cold starts, there can be an initial delay when the function first executes. This needs to be considered in performance and cost trade-offs.
Triaging Further
Warm-Up Strategy: Discuss how warm-up strategies like pre-warming functions or using auto-scaling groups can mitigate cold start issues.
Resource Allocation: Compare the resource allocation differences between serverless functions (e.g., memory, CPU) and VMs to understand what might be optimal for a given use case.

**Comparison:**

```
Invocation	Serverless Cost	VM Cost
0 requests	$0.00	$0.10
1 request	$0.20	$0.25
10 requests	$2.00	$2.50
```

**Code:**

```python
# Example of invoking a Lambda function in AWS using boto3
import boto3

def invoke_lambda_function(function_name):
    client = boto3.client('lambda')
    response = client.invoke(
        FunctionName=function_name,
        InvocationType='RequestResponse'
    )
    return response['Payload'].read().decode()
```

**Example input / output:**

```text
Input:
- `function_name`: MyRarelyUsedTool

Output:
- Function execution result (assuming successful)
```

**Data:**

```json
{
  "FunctionName": "MyRarelyUsedTool",
  "InvocationType": "RequestResponse"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. VM
6. Is the server busy?
7. Always incurs fixed cost
8. Can scale resources up anddown based on demand
9. Serverless Function
10. Is there an incoming request?
11. No execution, zero cost
12. Execute function, pay perinvocation and duration

