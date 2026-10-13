---
type: "practice"
difficulty: "Basic"
---

# What is an event-driven trigger and how might a file being uploaded to storage start a function?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Push Communication Patterns](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Push_Communication_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An event-driven trigger in cloud computing is a mechanism that automatically initiates a process or action when specific events occur. For example, a file upload event can trigger a function to perform certain actions on that file. In this context, we'll explore how an uploaded file might initiate a function using AWS S3 and AWS Lambda as our primary examples.

#### Triage
Restating the Question

The question is asking us to understand the concept of event-driven triggers specifically in the context of cloud storage (AWS S3) and serverless computing (AWS Lambda). The core idea here is that an uploaded file should trigger a function to process it, which could involve anything from simple logging to complex data transformation.

#### Assumptions
We are using AWS services for this example.
The function will perform some action based on the uploaded file's contents or metadata.
The function could be written in Python, Node.js, or any other supported language by AWS Lambda.
Step-by-Step Solution
1. Setting Up an S3 Bucket

First, create an S3 bucket to store your files. This will serve as the storage location where uploaded files are stored.

BASH
aws s3 mb s3://my-bucket-name --region us-west-2
2. Creating a Lambda Function

Next, we need to create a Lambda function that can be triggered by an event from S3. This function will process the uploaded file.

Write the Code for the Lambda Function: Here’s a simple Python example:

PYTHON
import json

def lambda_handler(event, context):
    # Extract the bucket and key name from the event
    s3_event = json.loads(json.dumps(event))
    bucket_name = s3_event['Records'][0]['s3']['bucket']['name']
    object_key = s3_event['Records'][0]['s3']['object']['key']

    print(f"Processing file: {object_key} in bucket: {bucket_name}")

    # Here you can perform actions such as downloading the file, processing it, and storing results

    return {
        'statusCode': 200,
        'body': json.dumps('File processed successfully')
    }

Deploy the Lambda Function: Use the AWS Management Console or CLI to deploy this function.

3. Configuring an Event Trigger

To configure the S3 bucket to trigger a Lambda function, follow these steps:

Go to the AWS Management Console and navigate to your S3 bucket.
In the Properties section, click on Event notifications.
Add a new notification configuration:
Event: s3:ObjectCreated:* (or s3:ObjectCreated:Put for just uploads)
Lambda Function: Select the Lambda function you created earlier
Save the settings.
4. Uploading Files to S3

Now, when you upload a file to your S3 bucket, the configured event trigger will automatically invoke the associated Lambda function.

#### Closing Summary
and Common Mistakes
Common Mistake: Not specifying the exact event type in the S3 notification configuration (e.g., s3:ObjectCreated:* instead of s3:ObjectCreated:Put).
Best Practice: Always test your setup by uploading a file to ensure that the Lambda function is triggered as expected.

#### Conclusion

By following these steps, you can set up an event-driven architecture where the upload of a file to an S3 bucket automatically triggers a Lambda function. This approach leverages serverless computing and cloud storage services for scalable, automated processing tasks.

