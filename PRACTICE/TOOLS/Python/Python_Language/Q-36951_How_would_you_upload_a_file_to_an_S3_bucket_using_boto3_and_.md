---
type: "practice"
difficulty: "Intermediate"
---

# How would you upload a file to an S3 bucket using boto3, and how does boto3 find your AWS credentials?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Credential Handling](<../../../../TOOLS/Python/Python_Language/Credential_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To upload a file to an S3 bucket using boto3 in Python, we first need to understand how boto3 finds our AWS credentials and then walk through the steps of setting up and executing the file upload.

Boto3 uses several mechanisms to find your AWS credentials. These include:

Environment Variables: You can set environment variables like AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.
Shared Configuration File: Boto3 looks for a configuration file at ~/.aws/credentials or within the shared folder /etc/aws/credentials.
Instance Roles (EC2 Instances): If running on an EC2 instance, boto3 can use IAM roles attached to that instance.
Boto Configuration File: Additional settings like region and endpoint URL are stored in ~/.aws/config or /etc/aws/config.

To upload a file using boto3, we need to create a session object and use it to interact with the S3 client.

#### Triaging the Question

The question is asking about two main aspects:

How to upload a file to an S3 bucket using boto3.
How boto3 finds AWS credentials.

#### First-Principles Solution
Step 1: Install Boto3

First, ensure you have boto3 installed in your Python environment:

BASH
pip install boto3
Step 2: Set Up Your Environment Variables

If using environment variables for authentication, set them up as follows:

PYTHON
import os
os.environ['AWS_ACCESS_KEY_ID'] = 'YOUR_AWS_ACCESS_KEY'
os.environ['AWS_SECRET_ACCESS_KEY'] = 'YOUR_AWS_SECRET_KEY'

Alternatively, you can use the boto3.Session method to create a session object with your credentials explicitly defined.

Step 3: Create an S3 Client

Create a client for interacting with the S3 service:

PYTHON
import boto3

# Using the default configuration
s3 = boto3.client('s3')

# Or using explicit credentials if needed
# s3 = boto3.client(
#     's3',
#     aws_access_key_id='YOUR_AWS_ACCESS_KEY',
#     aws_secret_access_key='YOUR_AWS_SECRET_KEY'
# )
Step 4: Upload a File to S3

Now, let’s upload a file:

PYTHON
def upload_file(file_name, bucket, object_name=None):
    """Upload a file to an S3 bucket

    :param file_name: File to upload
    :param bucket: Bucket to upload to
    :param object_name: S3 object name. If not specified then file_name is used
    :return: True if file was uploaded, else False
    """
    # If S3 object_name was not specified, use file_name
    if object_name is None:
        object_name = os.path.basename(file_name)

    try:
        response = s3.upload_file(file_name, bucket, object_name)
    except ClientError as e:
        print(e)
        return False
    return True

# Example usage
bucket_name = 'your-bucket-name'
file_path = '/path/to/your/file.txt'

if upload_file(file_path, bucket_name):
    print(f"File {file_path} uploaded to S3.")
else:
    print("Upload failed.")

#### Closing Summary
and Common Mistakes/Follow-Ups

This example demonstrates how to upload a file using boto3 in Python. Key points include setting up environment variables or using explicit credentials, creating an S3 client, and performing the file upload. Common mistakes might include not handling exceptions properly, incorrect bucket names, or misconfigured permissions.

For follow-ups, the interviewer might ask about:

Handling large files.
Using multipart uploads for very large files.
Setting metadata on uploaded objects.
Ensuring secure storage and usage of AWS credentials.

