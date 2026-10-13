---
type: "practice"
difficulty: "Basic"
---

# What is a presigned URL and how could it let a user upload a document straight to object storage for your RAG app?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Presigned URLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Presigned_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A presigned URL allows users to perform actions on an object stored in cloud-based object storage, such as uploading or downloading files, without needing API credentials directly. In the context of a Retrieval-Augmented Generation (RAG) application, this URL can enable a user to upload documents directly to the object storage bucket used by your RAG system.

To understand presigned URLs, let’s break down the core concepts and steps involved in generating such a URL. We'll start with the basics before diving into its practical implementation for uploading files to an object storage service like AWS S3 or Azure Blob Storage.

#### Triage
The question is asking you to explain what a presigned URL is and how it can facilitate direct file uploads from users to object storage, specifically in the context of a RAG application. The interviewer is likely probing your understanding of security practices, authentication mechanisms, and cloud storage concepts. Assumptions include that you are familiar with RESTful APIs and common cloud services like AWS S3 or Azure Blob Storage.

#### First Principles Solution

Understanding Presigned URLs: A presigned URL is a temporary URL that has been signed by your application using an access key (either an IAM user's or an IAM role’s). This signature ensures that the URL can only be used to perform specific actions, such as uploading a file, within a specified time frame.

Generating the Presigned URL:

In AWS S3, you would use the generatePresignedUrl method provided by the SDK. For example:
PYTHON
import boto3

s3 = boto3.client('s3')
bucket_name = 'your-bucket-name'
key_name = 'document.pdf'

url = s3.generate_presigned_url(
    'put_object',
    Params={'Bucket': bucket_name, 'Key': key_name},
    ExpiresIn=3600  # URL expires after one hour
)
print(url)
In Azure Blob Storage, you would use the GenerateSasUri method. For example:
CSHARP
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;

CloudBlobContainer container = new CloudBlobContainer(new Uri("https://your-account.blob.core.windows.net/your-container"), new StorageCredentialsAccountAndKey("accountname", "accountkey"));
CloudBlockBlob blockBlob = container.GetBlockBlobReference("document.pdf");
var sasToken = blockBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
{
    Permissions = SharedAccessBlobPermissions.Write,
    Start = DateTime.UtcNow,
    Expiry = DateTime.UtcNow.AddHours(1)
});
string url = blockBlob.Uri + sasToken;

#### Cons
ole.WriteLine(url);

Using the Presigned URL:

Once generated, this URL can be shared with users who need to upload files. They can use their web browsers or a custom application to make an HTTP PUT request to the presigned URL, which will store the file in your object storage bucket.

Security Considerations:

Ensure that the presigned URLs are short-lived (e.g., one hour) to minimize potential misuse.
Restrict access to the S3 or Blob Storage buckets via IAM policies and only grant necessary permissions.

#### Closing Summary
+ Common Mistakes/Follow-Ups: In summary, a presigned URL is a secure method for users to upload files directly to object storage without needing direct API credentials. This approach enhances security while providing flexibility in file management within your RAG application. Common mistakes include:

Using overly long expiration times on the URLs, which increases risk if they are compromised.
Not properly restricting IAM policies and permissions associated with the presigned URLs.

For follow-ups, you might be asked about alternative methods for secure file uploads or how to integrate this approach with other cloud services or security practices.

**Code:**

```

import boto3

s3 = boto3.client('s3') bucket_name = 'your-bucket-name' key_name = 'document.pdf'

url = s3.generate_presigned_url( 'put_object', Params={'Bucket': bucket_name, 'Key': key_name}, ExpiresIn=3600 ) print(url)

CODE
Copy

```csharp
using Microsoft.Azure.Storage;
using Microsoft.Azure.Storage.Blob;

CloudBlobContainer container = new CloudBlobContainer(new Uri("https://your-account.blob.core.windows.net/your-container"), new StorageCredentialsAccountAndKey("accountname", "accountkey"));
CloudBlockBlob blockBlob = container.GetBlockBlobReference("document.pdf");
var sasToken = blockBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
{
    Permissions = SharedAccessBlobPermissions.Write,
    Start = DateTime.UtcNow,
    Expiry = DateTime.UtcNow.AddHours(1)
});
string url = blockBlob.Uri + sasToken;
Console.WriteLine(url);
```

