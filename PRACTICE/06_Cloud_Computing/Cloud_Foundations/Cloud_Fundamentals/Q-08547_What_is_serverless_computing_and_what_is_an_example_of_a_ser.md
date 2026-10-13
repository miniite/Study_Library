---
type: "practice"
difficulty: "Intermediate"
---

# What is serverless computing and what is an example of a serverless service?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stateless by Default](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Serverless computing is a cloud service model where the cloud provider manages the underlying infrastructure, including server management, scaling, and operation. This allows developers to focus solely on writing application code without worrying about the servers hosting it. An example of a serverless service is AWS Lambda.

To start, I’ll provide a brief overview and then delve into the core concepts before giving an in-depth explanation.

**Comparison:**

```
Function	Event (Input)	Response (Output)
AWS Lambda	{"event": {}, "context": {}}	{ "statusCode": 200, "body": "Hello from Lambda!" }
```

**Code:**

```text
# Example Python function for AWS Lambda
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

**Example input / output:**

```

Given the input:

JSON
Copy
{
  "event": {},
  "context": {
    "functionName": "exampleLambda",
    "invokedFunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:exampleLambda"
  }
}

The output would be:

JSON
Copy
{
  "statusCode": 200,
  "body": "Hello from Lambda!"
}
```

**Data:**

```json
{
  "event": {},
  "context": {}
}
```

**Flow:**

1. User Request
2. API Gateway
3. AWS Lambda Function
4. Response

