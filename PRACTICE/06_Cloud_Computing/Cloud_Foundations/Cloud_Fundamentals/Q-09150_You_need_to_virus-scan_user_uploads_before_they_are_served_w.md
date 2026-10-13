---
type: "practice"
difficulty: "Intermediate"
---

# You need to virus-scan user uploads before they are served, without blocking the upload response. Walk me through an event-driven design using storage triggers and a quarantine bucket.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of scanning user uploads with anti-virus software while ensuring that the upload process is not blocked, we can leverage serverless computing and event-driven architectures. This approach uses storage triggers to monitor uploads in real-time, allowing us to initiate a virus scan without affecting the initial response time for the client.

#### Triage
Understanding the Requirements
Core Problem: Upload files need to be scanned for viruses before being served.

#### Cons
traints: The upload process should not be blocked while the virus scan is performed.

#### Assumptions
We are using a cloud-based storage service (e.g., S3, GCS).
Anti-virus scanning is available as an external service or can be integrated through a function.

#### First-Principles Solution

We will design this solution using a combination of event-driven triggers and a quarantine bucket. Here’s how it works:

User Uploads to Storage Bucket: When a user uploads a file, the file is saved directly to an "upload" bucket.
Storage Trigger Activation: A trigger (e.g., AWS Lambda function) is set up to listen for new objects in this upload bucket.
File Quarantine and Scan: The triggered function moves the uploaded files to a quarantine bucket and initiates a virus scan.
Results Handling:
If the file passes the scan, it is moved from the quarantine bucket to a "trusted" bucket where it can be served.
If the file fails the scan, it remains in the quarantine bucket (or deleted) with appropriate logging.
Detailed Steps

Storage Bucket Configuration:

PLAINTEXT
[Upload Bucket]
└───[User Uploads]  # Object event triggers here
    ├───file1.txt
    ├───file2.docx
    ├───...

Triggers and Functions:

Event Trigger: When a new object is created in the upload bucket, this trigger activates.
Lambda Function (upload-trigger-lambda): This function handles the logic of moving files to the quarantine bucket and initiating the scan.

Quarantine Bucket Setup:

PLAINTEXT
[Quarantine Bucket]
    ├───[Scanning In Progress]  # For tracking current scans
        ├───file1.txt
        └───file2.docx

Scan Function (scan-virus-lambda): This function performs the actual virus scan and updates the status in the quarantine bucket.

Results Handling:

Passing Files: Once a file passes the scan, it is moved to a "trusted" bucket.
PLAINTEXT
[Trusted Bucket]
    ├───file1.txt  # Passes scanning criteria
    └───file2.docx  # Passes scanning criteria
Failing Files: If a file fails the scan, it is either quarantined or deleted and logged.
PLAINTEXT
[Quarantine Bucket]
    ├───[Scanning In Progress]  # For tracking current scans
    └───file3.exe  # Fails scanning criteria

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The event-driven design using storage triggers and a quarantine bucket ensures that the upload process is not blocked while virus scans are performed. This approach leverages serverless functions to handle the background tasks efficiently.

#### Common Mistakes
Failing to properly configure event triggers, leading to missed file processing.
Not handling scan results correctly, causing files to get stuck in the quarantine bucket or not being removed when they should be.
Insufficient logging and monitoring can lead to undetected issues.

#### Follow-Ups
How do you ensure the security of the storage buckets?
What are your thoughts on using a different service for virus scanning (e.g., antivirus software)?
How would you handle large files that may exceed the Lambda function execution limits?

**Comparison:**

```
Bucket	Purpose
user-uploads	For user uploads before scanning.
quarantine-bucket	For temporarily storing files during the scan process.
trusted-bucket	For storing files that passed the virus scan.
```

**Code:**

```python
# Example Lambda Function to move files to quarantine and initiate scan
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')

    # Get the file from the upload bucket
    source_bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    # Move to quarantine bucket
    s3.copy_object(CopySource={'Bucket': source_bucket, 'Key': key},
                   Bucket='quarantine-bucket',
                   Key=f'scanning-in-progress/{key}')

    # Initiate virus scan (this would be a custom function or service integration)
    initiate_virus_scan(key)

def initiate_virus_scan(file_key):
    print(f"Initiating virus scan for {file_key}")
    # Simulate scan result
    if "malware" in file_key:
        handle_failing_file(file_key)
    else:
        handle_passing_file(file_key)

def handle_failing_file(file_key):
    s3.delete_object(Bucket='quarantine-bucket', Key=f'scanning-in-progress/{file_key}')
    print(f"File {file_key} failed scan, deleted from quarantine")

def handle_passing_file(file_key):
    # Move file to trusted bucket
    s3.copy_object(CopySource={'Bucket': 'quarantine-bucket', 'Key': f'scanning-in-progress/{file_key}'},
                   Bucket='trusted-bucket',
                   Key=file_key)
    print(f"File {file_key} passed scan, moved to trusted bucket")
```

**Example input / output:**

```text
Input:
- User uploads a file to 'user-uploads' bucket

Output:
1. File is moved from 'user-uploads' to 'quarantine-in-progress'
2. Virus scan initiated for the file.
3. If pass, file moved to 'trusted-bucket'; if fail, it remains in 'quarantine-in-progress' or deleted.
```

**Data:**

```json
{
  "upload_bucket": "user-uploads",
  "quarantine_bucket": "quarantine-bucket",
  "trusted_bucket": "trusted-files"
}
```

**Flow:**

1. Yes
2. Pass
3. Fail
4. User Uploads File
5. New Object?
6. Trigger Lambda Function
7. Move to Quarantine Bucket
8. Scan Results?
9. Move to Trusted Bucket
10. Handle Failing File(Delete/Log)

