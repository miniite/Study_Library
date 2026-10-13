---
type: "practice"
difficulty: "Intermediate"
---

# How would you list running EC2 instances and their states using boto3 in a small inventory script?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll be writing a Python script that uses the boto3 library to list all the running Amazon EC2 instances along with their current states. This involves setting up the AWS credentials, initializing the boto3 client for EC2, and making an API call to fetch the instance information.

Firstly, let's break down the steps involved:

Setup AWS Credentials: Ensure that your AWS credentials are configured properly.
Initialize Boto3 Client: Use boto3 to initialize a client for Amazon EC2.
List Instances: Call the describe_instances method and filter out instances in the 'running' state.
Print Results: Format and print the required information.

#### Triaging the Question

The question is essentially asking you to write a small script that interacts with AWS EC2 via Python's boto3 library. The interviewer might be probing your understanding of:

How to set up and manage AWS credentials securely.
Basic usage of the boto3 library.
Filtering data based on specific states.
Properly handling API responses.

#### Assumptions
You have basic knowledge of Python programming.
You know how to install and configure boto3.
You are familiar with AWS EC2 services.
Step-by-Step Solution
Step 1: Setup AWS Credentials

Ensure that your AWS credentials are configured either using environment variables or by setting up a configuration file at the default location (~/.aws/credentials).

PYTHON
import os
os.environ['AWS_ACCESS_KEY_ID'] = 'your_access_key'
os.environ['AWS_SECRET_ACCESS_KEY'] = 'your_secret_key'
os.environ['AWS_DEFAULT_REGION'] = 'us-west-2'  # or your preferred region
Step 2: Initialize Boto3 Client for EC2
PYTHON
import boto3

ec2_client = boto3.client('ec2')
Step 3: List Running Instances

Use the describe_instances method to get a list of all instances and then filter out only those that are in the 'running' state.

PYTHON
response = ec2_client.describe_instances()
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        if instance['State']['Name'] == 'running':
            print(f"Instance ID: {instance['InstanceId']}, State: {instance['State']['Name']}")
Step 4: Print Results

The output will list each running EC2 instance along with its state.

#### Closing Summary
and Common Mistakes/Follow-Ups

This script effectively lists running instances by filtering the response from describe_instances. Key points to remember:

Ensure your AWS credentials are securely managed.
Handle exceptions for API calls gracefully.

#### Cons
ider pagination if you have a large number of instances.

#### Common mistakes
include:

Forgetting to set up and configure AWS credentials.
Not properly handling exceptions that might occur during API calls.
Filtering the wrong state, which could lead to incorrect results.

#### Example
Output
PYTHON
Instance ID: i-0f123456789abcdef0, State: running
Instance ID: i-0f123456789abcdef1, State: running

**Code:**

```python
import boto3

def list_running_instances():
    os.environ['AWS_ACCESS_KEY_ID'] = 'your_access_key'
    os.environ['AWS_SECRET_ACCESS_KEY'] = 'your_secret_key'
    os.environ['AWS_DEFAULT_REGION'] = 'us-west-2'

    ec2_client = boto3.client('ec2')

    response = ec2_client.describe_instances()

    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] == 'running':
                print(f"Instance ID: {instance['InstanceId']}, State: {instance['State']['Name']}")

list_running_instances()
```

