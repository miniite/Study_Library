---
type: "practice"
difficulty: "Intermediate"
---

# Your app stores user uploads in object storage and you need to serve them fast globally while blocking hotlinking and abuse. How do you combine CDN, signed URLs, and referer checks?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CDN Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/CDN_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your application serves user-uploaded content quickly from a global location while mitigating issues like hotlinking and abuse, we need to leverage a combination of Cloud Delivery Networks (CDNs), signed URLs, and referer checks. This approach ensures efficient content delivery and robust security.

#### Triaging the Question

The question requires us to design a solution that addresses three key aspects:

Global Content Delivery: Ensure fast access to user-uploaded files from multiple geographic locations.
Hotlinking Prevention: Prevent unauthorized users from accessing your stored objects directly via direct URLs.
Abuse Mitigation: Protect against malicious activities such as excessive requests or DDoS attacks.

#### Assumptions
We are using a cloud provider that supports CDNs and object storage.
The application is stateless, allowing us to leverage cached responses effectively.
User sessions can be managed with session tokens or cookies if necessary.

#### First Principles Solution
Step 1: Set Up Object Storage

First, we need to store user-uploaded files in an object storage service such as AWS S3, Google Cloud Storage, or Alibaba Cloud Object Storage Service (OOS). These services provide robust security features and efficient data distribution capabilities.

Object Storage
CDN
Step 2: Configure a CDN

The CDNs provided by cloud vendors can cache content at edge locations, reducing latency and improving performance. However, they also need to be configured properly to ensure security and usability.

Object Storage
CDN
Step 3: Generate Signed URLs

Signed URLs are time-limited access tokens that grant temporary access to the object storage service. These can be generated on-the-fly in your application backend when a user requests a file.

JAVASCRIPT
// Example of generating a signed URL using AWS S3 SDK

#### cons
t s3 = new AWS.S3();

#### cons
t params = {
    Bucket: 'my-bucket',
    Key: 'path/to/my/file.jpg'
};

s3.getSignedUrl('getObject', params, function(err, url) {
  if (err) console.log(err); // an error occurred
  else     console.log(url); // successful response
});
Step 4: Implement Referer Checks

To prevent hotlinking, we need to configure our signed URLs with referer checks. This ensures that the file can only be accessed from allowed domains.

Object Storage
CDN
Referer Check
Detailed Implementation

Generate Signed URLs on Demand: When a user requests a file, generate a signed URL with a short expiration period (e.g., 5 minutes).

Add Referer Header to Signed URL: Include the x-amz-referrer header in your signed URL request. This header should contain the allowed referrer domain(s).

CDN Configuration: Configure your CDN to serve files from object storage with referer checks enabled. For example, in AWS CloudFront:

Go to the distribution settings.
Navigate to the “Behaviors” tab and edit the behavior associated with the object storage path.
Set up an access control policy that includes allowlisted domains.

Rate Limiting: Implement rate limiting at both the application level (backend) and CDN level to prevent abuse. This can be done using Cloudflare, AWS WAF, or other security services.

Logging and Monitoring: Use logging and monitoring tools provided by your cloud provider to track suspicious activities and block IP addresses if necessary.

#### Closing Summary

By combining CDNs for global content delivery, signed URLs with referer checks for security, and rate limiting to prevent abuse, we can ensure efficient and secure serving of user-uploaded files. This approach provides a robust solution that balances performance, security, and cost-effectiveness.

#### Common Mistakes/Follow-Ups
Mistake: Failing to enable rate limiting or referer checks.

#### Follow-Up
Discuss additional measures like IP blocking, CAPTCHAs, or using a WAF (Web Application Firewall) for more advanced protection.

**Code:**

```javascript
// Example of generating a signed URL with AWS S3 SDK
const s3 = new AWS.S3();
const params = {
    Bucket: 'my-bucket',
    Key: 'path/to/my/file.jpg'
};

s3.getSignedUrl('getObject', { ...params, Expires: 60 }, function(err, url) {
  if (err) console.log(err); // an error occurred
  else     console.log(url); // successful response
});
```

**Data:**

```json
{
    "bucket": "my-bucket",
    "key": "path/to/my/file.jpg"
}
```

