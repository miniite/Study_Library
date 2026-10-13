---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a presigned-URL upload flow so a mobile client uploads directly to storage, and what expiry, content-type, and size constraints would you bake into the signature?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Presigned URLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Presigned_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a presigned-URL upload flow where a mobile client can upload files directly to cloud storage, we need to focus on several key aspects: security, efficiency, and usability. The core idea is to generate temporary URLs with specific permissions and constraints that the mobile client uses to initiate an upload request directly to the cloud storage service.

#### Triage

The question asks us to design a presigned-URL upload flow. This means understanding how temporary access can be granted to the mobile client, ensuring data security while maintaining ease of use. We'll need to consider factors such as URL expiry times, content type constraints, and file size limitations. The interviewer is probing our knowledge of cloud storage services, secure authentication mechanisms, and best practices for handling user-generated content.

#### Assumptions
Cloud Storage Service: AWS S3 or similar.
Mobile Client: A standard mobile app using HTTP/HTTPS.
Security Level: Moderate to high—protecting against unauthorized access while allowing easy upload from the client.

#### Solution
Step-by-Step from First Principles

Generate Presigned URL

Use AWS S3's generatePresignedUrl function to create a temporary URL with specified parameters.
The URL will include an expiration time, content type, and file size constraints.

Expiration Time

Set the URL to expire after 5-10 minutes for security reasons. This prevents unauthorized access if the link is intercepted or lost.

Content-Type Constraints

Define allowed content types (e.g., image/jpeg, video/mp4). This helps in rejecting inappropriate uploads and ensures compliance with file format expectations.

File Size Limits

Set a maximum file size limit to prevent overloading the storage system or network. For example, 10MB per file.

Signature Parameters

Include parameters like Content-Type, x-amz-acl (bucket ACL), and any other necessary headers.

Mobile Client Implementation

The mobile app requests a presigned URL from the server.
Uses the URL to initiate an upload directly to S3, sending metadata if needed.

#### Example
Workflow

Server-Side Request Handling

PYTHON
import boto3

# Initialize S3 client
s3 = boto3.client('s3')

def generate_presigned_url(bucket_name, object_name):
    """Generate a presigned URL for the specified file."""
    try:
        url = s3.generate_presigned_url(
            'put_object',
            Params={
                'Bucket': bucket_name,
                'Key': object_name,
                'ContentType': 'image/jpeg',  # Example content type
                'ACL': 'private'  # Set appropriate ACL based on security needs
            },
            ExpiresIn=600  # URL expires in 10 minutes
        )
    except Exception as e:
        return None, str(e)

    return url

Mobile Client Request

SWIFT
// Swift example using URLSession
func requestPresignedUrl(bucketName: String, objectName: String) async throws -> URL {
    let url = try await withCheckedThrowingContinuation { continuation in
        guard let baseUrl = URL(string: "https://your-cloud-service.com/generate-presigned-url") else {
            continuation.resume(throwing: NSError(domain: "", code: 1, userInfo: nil))
            return
        }

        var request = URLRequest(url: baseUrl)
        request.httpMethod = "POST"
        let bodyData = try JSONSerialization.data(withJSONObject: [
            "bucket_name": bucketName,
            "object_name": objectName
        ])
        request.httpBody = bodyData

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let response = response as? HTTPURLResponse,
                      response.statusCode == 200,
                      let responseData = data {
                do {
                    let result = try JSONSerialization.jsonObject(with: responseData) as? [String: String]
                    guard let url = URL(string: result?["url"] ?? "") else { return }
                    continuation.resume(returning: url)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }.resume()
    }
}

#### Closing Summary
and Common Mistakes

The presigned-URL upload flow ensures secure, direct file uploads from mobile clients to cloud storage. Key constraints like URL expiry, content type, and file size help maintain data security and compliance.

#### Common Mistakes

Expiry Time Too Long: URLs should not be valid for longer than necessary.
No Content Type Enforcement: Allowing any content type can lead to security vulnerabilities or incorrect processing.
Ignoring File Size Limits: Large files may cause performance issues or storage overuse.
Triaging Input
Ensure the input validation on both server and client sides is robust.
Regularly monitor usage patterns to detect potential abuse.

**Comparison:**

```
Parameter	Description
Bucket Name	The name of the S3 bucket
Object Name	The name of the object being uploaded
Expiry Time	URL expiration time in seconds (e.g., 600)
Content-Type	Allowed content types
ACL	Access control level
```

**Code:**

```

import boto3

def generate_presigned_url(bucket_name, object_name): s3 = boto3.client('s3') try: url = s3.generate_presigned_url( 'put_object', Params={ 'Bucket': bucket_name, 'Key': object_name, 'ContentType': 'image/jpeg', # Example content type 'ACL': 'private' # Set appropriate ACL based on security needs }, ExpiresIn=600 # URL expires in 10 minutes ) except Exception as e: return None, str(e)

CODE
Copy
return url
CODE
Copy

```swift
import Foundation

func requestPresignedUrl(bucketName: String, objectName: String) async throws -> URL {
    let url = try await withCheckedThrowingContinuation { continuation in
        guard let baseUrl = URL(string: "https://your-cloud-service.com/generate-presigned-url") else {
            continuation.resume(throwing: NSError(domain: "", code: 1, userInfo: nil))
            return
        }

        var request = URLRequest(url: baseUrl)
        request.httpMethod = "POST"
        let bodyData = try JSONSerialization.data(withJSONObject: [
            "bucket_name": bucketName,
            "object_name": objectName
        ])
        request.httpBody = bodyData

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else if let response = response as? HTTPURLResponse,
                      response.statusCode == 200,
                      let responseData = data {
                do {
                    let result = try JSONSerialization.jsonObject(with: responseData) as? [String: String]
                    guard let url = URL(string: result?["url"] ?? "") else { return }
                    continuation.resume(returning: url)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }.resume()
    }
}
```

**Example input / output:**

```
Server Request
JSON
Copy
{
    "bucket_name": "example-bucket",
    "object_name": "image.jpg"
}
Server Response (Example)
JSON
Copy
{
    "url": "https://s3.amazonaws.com/example-bucket/image.jpg?Expires=1672545600&AWSAccessKeyId=ASIAIOSFODNN7EXAMPLE&Signature=7KXQHw4iR5YvzjY2tTgJZf8Sd9o%3D"
}
```

**Data:**

```json
{
  "bucket_name": "example-bucket",
  "object_name": "image.jpg"
}
```

**Flow:**

1. Yes
2. No
3. Generate Presigned URL
4. Valid Input?
5. Generate and Return URL
6. Error Handling
7. Upload via Presigned URL
8. Return Error

