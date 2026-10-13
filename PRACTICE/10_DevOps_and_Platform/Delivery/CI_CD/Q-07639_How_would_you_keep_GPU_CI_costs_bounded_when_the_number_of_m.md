---
type: "practice"
difficulty: "Deep"
---

# How would you keep GPU CI costs bounded when the number of model experiments and PRs grows, without slowing feedback?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ CI Cost Optimization](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage GPU CI costs while accommodating a growing number of model experiments and pull requests (PRs) efficiently, we need to focus on several aspects: optimizing resource allocation, leveraging shared resources, implementing cost-aware scheduling policies, and automating the process. The goal is to minimize idle time and ensure that feedback remains rapid without compromising on the quality or speed of CI/CD cycles.

First, let's break down what the question is asking:

What are we trying to achieve? We need to balance the number of experiments and PRs with the available GPU resources.
What does the interviewer want to see? They are probing our ability to design a scalable system that can handle increasing loads efficiently, while also optimizing for cost.

#### Assumptions

The CI/CD pipeline is set up on a cloud platform where GPUs are provisioned.
There's an ongoing increase in the number of experiments and PRs.
Cost optimization without compromising feedback speed or experiment quality is paramount.

Step-by-step solution:

1. Resource Allocation Optimization
Dynamic Sizing
Implement dynamic sizing for GPU resources based on demand. Use machine learning models to predict the required GPU capacity needed at different times of the day/week.

#### Example
AWS Batch, Google Cloud AI Platform Notebooks, or Azure Machine Learning Services support dynamic scaling.
2. Shared Resources
Shared GPUs
Create a shared pool of GPUs where multiple teams can access them. This reduces the chance of underutilization and ensures that resources are used efficiently.

#### Example
Set up a Kubernetes cluster with multiple GPU nodes, allowing different projects to request specific types and numbers of GPUs as needed.
3. Cost-Aware Scheduling
Priority-Based Scheduling
Prioritize CI/CD jobs based on their importance or urgency. Critical PRs can be given higher priority.

#### Example
Use a scheduling framework like Kubernetes with custom admission controllers to enforce priorities and allocate resources accordingly.
4. Automated Infrastructure Management
Infrastructure as Code (IaC)
Implement IaC using tools like Terraform, CloudFormation, or Pulumi to automate the provisioning of GPU instances.

#### Example
Automate the creation and deletion of GPU instances based on demand patterns.
5. Monitoring and Feedback Loops
Real-Time Monitoring
Set up real-time monitoring for resource usage and job performance. Tools like Prometheus, Grafana, or Datadog can provide insights into when and where resources are being underutilized.

#### Example
Use metrics to identify idle GPU instances that can be shut down during off-peak hours.
6. Cost Optimization Strategies
On-Demand vs Reserved Instances
Utilize both on-demand and reserved GPU instances based on the usage patterns. On-demand instances are cheaper but may incur higher costs when demand spikes unexpectedly.

#### Example
Reserve fewer GPUs for steady-state operations and dynamically allocate additional capacity during peaks.

#### Closing Summary
and Common Mistakes

By focusing on dynamic sizing, shared resources, priority-based scheduling, automated infrastructure management, real-time monitoring, and cost optimization strategies, we can ensure that GPU CI/CD costs are kept under control while maintaining efficient feedback loops. Common mistakes include:

Underutilization: Not leveraging shared resources effectively.
Overprovisioning: Allocating too many resources upfront without proper demand forecasting.
Insufficient Monitoring: Failing to monitor resource usage and adjust strategies in real-time.

**Comparison:**

```
Time	GPU Demand (Units)
09:00	25
17:00	8
23:00	6
```

**Code:**

```python
# Example of dynamic sizing using a hypothetical Python script
def predict_gpu_demand(hour):
    # Machine learning model predicting GPU usage based on historical data
    if hour >= 9 and hour <= 17:
        return 20  # More demand during business hours
    else:
        return 5   # Less demand outside business hours

# Example of priority-based scheduling using a custom Kubernetes controller
def prioritize_jobs(jobs):
    return sorted(jobs, key=lambda job: job.priority, reverse=True)
```

**Example input / output:**

```text
Input:
- Number of GPU jobs increasing over time.
- High demand during business hours and low demand outside.

Output:
- Dynamic allocation of GPUs based on real-time demand.
- Prioritization of critical PRs with faster CI/CD cycles.
- Cost savings through efficient use of shared resources and dynamic scaling.
```

**Data:**

```yaml
# Sample YAML for IaC with Terraform
resource "aws_instance" "gpu_instance" {
  count            = var.gpu_count
  ami              = data.aws_ami.amzn2-ami-gp2.id
  instance_type    = "g4dn.xlarge"
  vpc_security_group_ids = [aws_security_group.default.id]
  key_name         = aws_key_pair.my-key-pair.name
}
```

**Flow:**

1. Yes
2. No
3. CI/CD Request
4. Is PR Critical?
5. Priority Higher, Faster Queue
6. Regular Queue, SchedulingBased on Demand
7. Dynamic Resource AllocationBased on Real-Time Metrics
8. Real-Time Monitoring andFeedback Adjustments

