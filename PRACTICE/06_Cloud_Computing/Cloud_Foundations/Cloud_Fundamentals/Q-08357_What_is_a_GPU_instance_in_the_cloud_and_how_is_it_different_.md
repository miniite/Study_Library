---
type: "practice"
difficulty: "Basic"
---

# What is a GPU instance in the cloud and how is it different from a normal CPU instance for AI workloads?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ GPU Sharing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the realm of cloud computing, a GPU instance stands out as a specialized type of virtual machine designed to handle computationally intensive tasks efficiently, particularly those involving graphics and artificial intelligence (AI). A GPU instance is different from a standard CPU-based instance in several key ways that are crucial for AI workloads.

**Comparison:**

|Instance Type | CPU | GPU | Memory (GiB) |
|--------------|-----|-----|-------------|
|p3.2xlarge     | 1   | 8  | 61          |

**Code:**

```text
# Example of initializing a GPU-enabled EC2 instance on AWS
aws ec2 run-instances \
    --image-id ami-0c55b1593example \
    --instance-type p3.2xlarge \
    --placement AvailabilityZone=us-west-2a \
    --count 1 \
    --key-name MyKeyPair \
    --security-group-ids sg-0123456789abcdef0
```

**Example input / output:**

```text
# Sample input and output for launching a GPU instance on AWS
Input:
aws ec2 run-instances \
    --image-id ami-0c55b1593example \
    --instance-type p3.2xlarge \
    --placement AvailabilityZone=us-west-2a \
    --count 1 \
    --key-name MyKeyPair \
    --security-group-ids sg-0123456789abcdef0

Output:
{
    "Instances": [
        {
            "InstanceId": "i-0eef1b4c8d1f0b1c9",
            "InstanceType": "p3.2xlarge",
            "State": {
                "Code": 16,
                "Name": "running"
            },
            "LaunchTime": "2023-05-17T14:38:00Z"
        }
    ]
}
```

**Data:**

```text
{
  "InstanceType": "p3.2xlarge",
  "Placement": {
    "AvailabilityZone": "us-west-2a"
  },
  "Count": 1,
  "KeyName": "MyKeyPair",
  "SecurityGroupIds": ["sg-0123456789abcdef0"]
}
```

**Flow:**

1. ami-0c55b1593example
2. us-west-2a
3. MyKeyPair
4. sg-0123456789abcdef0
5. Launch GPU Instance
6. Image ID?
7. Select p3.2xlarge
8. Placement Zone?
9. Key Pair?
10. Security Group IDs?
11. Run Instances
12. Launching Instance
13. Instance Running

