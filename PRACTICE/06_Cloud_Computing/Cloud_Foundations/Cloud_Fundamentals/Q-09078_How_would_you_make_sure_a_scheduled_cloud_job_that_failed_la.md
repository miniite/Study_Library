---
type: "practice"
difficulty: "Intermediate"
---

# How would you make sure a scheduled cloud job that failed last night actually retries instead of silently skipping?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Exactly Once Scheduled Jobs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Exactly_Once_Scheduled_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a scheduled cloud job that failed last night actually retries rather than silently skipping, we need to implement robust error handling and logging mechanisms along with a retry strategy. This approach involves understanding the nature of the job, identifying potential failure points, and setting up automated notifications and monitoring.

#### Triaging the Question

The core of this question is about ensuring reliability in scheduled cloud jobs by addressing both technical and operational aspects:

Technical Aspects: We need to ensure that the job retries when it fails.
Operational Aspects: Monitoring and logging must be set up to detect failures, and notifications should be configured so that relevant teams are alerted.

#### First Principles Solution
Identify the Job Failure Point:
Determine where the job is failing by reviewing logs or using monitoring tools like CloudWatch (AWS), Stackdriver (GCP), or Azure Monitor.
Implement Retry Logic:
Use cloud-native features to configure retries. For example, in AWS Lambda, you can set up event source mappings that retry failed invocations.
Logging and Monitoring:
Ensure detailed logging for the job's execution. Cloud providers offer robust logging mechanisms such as CloudTrail (AWS), Logging V2 (GCP), or Application Insights (Azure).
Automated Notifications:
Set up alerts using notification services like SNS (AWS) or Pub/Sub (GCP) to alert stakeholders when a job fails.
Review and Validate:
Test the retry logic in a staging environment before deploying it to production.
Step-by-Step Solution
Determine Job Failure Point
Review logs for errors: aws logs filter-log-events --log-group-name /your-log-group (AWS).
Identify the specific step or task that is failing.
Configure Retry Logic in AWS Lambda:
JSON
{
  "FunctionName": "exampleLambda",
  "Description": "Example Lambda function with retry logic.",
  "Role": "arn:aws:iam::123456789012:role/service-role/lambda-executeRole",
  "Handler": "index.handler",
  "Runtime": "nodejs14.x",
  "Timeout": 30,
  "MemorySize": 128,
  "DeadLetterConfig": {
    "TargetArn": "arn:aws:sqs:us-east-1:123456789012:exampleQueue"
  },
  "RetryAttempts": 3
}
RetryAttempts field ensures the Lambda function retries up to three times.
Set Up Detailed Logging:
JSON
{
  "logLevel": "INFO",
  "appenders": [
    { "type": "console" },
    { "type": "awslogs", "category": "your-log-group-name" }
  ]
}
Configure Automated Notifications in AWS SNS:
Create an SNS topic and subscribe it to relevant email addresses or webhooks.
BASH
aws sns create-topic --name example-topic
aws sns subscribe --topic-arn arn:aws:sns:us-east-1:123456789012:example-topic --protocol email --endpoint user@example.com
Validate and Test:
Simulate a failure in the job to ensure that retries and notifications work as expected.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By setting up these mechanisms, we can ensure that our scheduled cloud jobs are resilient and reliable. Common mistakes include not configuring proper logging or retry logic, which can lead to undetected failures. Additionally, failing to set up automated notifications can result in unaddressed issues.

#### Follow-ups

How would you integrate this with a CI/CD pipeline?
What other monitoring tools would you consider for a comprehensive solution?

**Code:**

```json
{
  "FunctionName": "exampleLambda",
  "Description": "Example Lambda function with retry logic.",
  "Role": "arn:aws:iam::123456789012:role/service-role/lambda-executeRole",
  "Handler": "index.handler",
  "Runtime": "nodejs14.x",
  "Timeout": 30,
  "MemorySize": 128,
  "DeadLetterConfig": {
    "TargetArn": "arn:aws:sqs:us-east-1:123456789012:exampleQueue"
  },
  "RetryAttempts": 3
}
```

**Example input / output:**

```text
# Create SNS Topic
aws sns create-topic --name example-topic

# Subscribe to the SNS topic via email
aws sns subscribe --topic-arn arn:aws:sns:us-east-1:123456789012:example-topic --protocol email --endpoint user@example.com
```

