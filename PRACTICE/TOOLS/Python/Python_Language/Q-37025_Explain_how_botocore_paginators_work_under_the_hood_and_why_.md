---
type: "practice"
difficulty: "Intermediate"
---

# Explain how botocore paginators work under the hood and why calling describe_instances without a paginator silently truncates results at the first page in a large account.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how botocore paginators work, we need to delve into the mechanics of AWS SDKs and Python's dynamic nature. When dealing with large-scale AWS accounts that contain numerous resources like EC2 instances, using a paginator is crucial because the API limits the number of results returned in a single call. In this scenario, if you attempt to fetch all instances without a paginator, botocore silently limits your request to just one page, which can lead to missing data.

#### Triaging the Question

The question is probing our understanding of how pagination works with AWS SDKs and Python libraries like botocore. It also tests whether we recognize potential pitfalls in such operations. We assume that the user has a basic knowledge of AWS services, specifically EC2, and an intermediate level of experience with Python.

#### First-Principles Solution

Botocore is the low-level, data-driven, service-specific SDK for Amazon Web Services (AWS). It serves as the foundation for higher-level abstractions like boto3. When dealing with paginated operations in botocore, such as listing EC2 instances, each call to an AWS API method returns a limited number of results. To retrieve all results, you need to use a paginator object that handles subsequent requests automatically.

Here’s how it works under the hood:

Initialization: You initialize the paginator for a specific operation (e.g., describe_instances).
First Call: The first call returns an initial page of results and metadata about the total number of pages.
Subsequent Calls: If there are more pages, the paginator automatically sends additional requests to fetch them until all pages have been retrieved.

#### Example
Code

Let's look at a simplified example using botocore:

PYTHON
import boto3

# Initialize an EC2 client and paginator
ec2_client = boto3.client('ec2')
paginator = ec2_client.get_paginator('describe_instances')

# Use the paginator to iterate over all pages of results
response_iterator = paginator.paginate()

for page in response_iterator:
    for reservation in page['Reservations']:
        for instance in reservation['Instances']:
            print(instance['InstanceId'])
Why describe_instances Without a Paginator Truncates Results

The AWS API limits the number of instances returned per request. By default, without specifying a paginator, your call to describe_instances only returns the first page of results, which is often just one page due to this limit.

This behavior can be problematic in large accounts where you need all the data. To ensure you retrieve all instances, you should explicitly use the paginator as shown above.

#### Closing Summary
and Common Mistakes

In summary, botocore paginators are essential for working with large datasets on AWS services. They handle the complexity of making multiple API calls automatically, ensuring that you don't miss any data. The key takeaway is to always use a paginator when dealing with potentially large-scale operations like listing EC2 instances.

#### Common mistakes
include:

Ignoring the default page limit and assuming describe_instances will return all results.
Not using pagination in scripts or applications where completeness of data is crucial.
Common Follow-Ups
Handling Pagination Efficiently: How can you optimize the use of paginators for large-scale operations?
Error Handling with Paginators: What are best practices for handling errors and retries when using paginators?

**Comparison:**

| Instance ID | State       |
|-------------|------------|
| i-1234567890abcdef0 | running   |
| i-0987654321fedcba0  | stopped    |

**Code:**

```python
import boto3

# Initialize an EC2 client and paginator
ec2_client = boto3.client('ec2')
paginator = ec2_client.get_paginator('describe_instances')

# Use the paginator to iterate over all pages of results
response_iterator = paginator.paginate()

for page in response_iterator:
    for reservation in page['Reservations']:
        for instance in reservation['Instances']:
            print(instance['InstanceId'])
```

**Example input / output:**

```text
# Example Output
i-1234567890abcdef0
i-0987654321fedcba0
```

**Data:**

```json
{
  "Reservations": [
    {
      "Instances": [
        {
          "InstanceId": "i-1234567890abcdef0",
          "State": { "Name": "running" }
        },
        {
          "InstanceId": "i-0987654321fedcba0",
          "State": { "Name": "stopped" }
        }
      ]
    }
  ]
}
```

**Flow:**

1. Initiate Describe Instances Call
2. Check Page Limit
3. Return First Page of Results
4. Check for More Pages
5. Request Next Page
6. Process All Pages

