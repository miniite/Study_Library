---
type: "practice"
difficulty: "Basic"
---

# What is cloud computing and how does it differ from traditional on-premises hosting?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cloud Deployment and Service Models](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Deployment_and_Service_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cloud computing refers to the delivery of computing resources, including servers, storage, databases, software, and networking, over the internet. Unlike traditional on-premises hosting, where these services are hosted in-house by a company, cloud computing offers scalability, flexibility, and cost-effectiveness.

Let's break down how it works and what makes it different from traditional setups:

Definition: Cloud computing involves using remote servers to store, manage, and process data, as opposed to local servers or personal computers.
Scalability: One of the key differences is scalability. In cloud environments, resources can be easily scaled up or down based on demand without the need for significant capital investment in hardware. Traditional setups often require physical infrastructure upgrades, which are more costly and time-consuming.
Cost-Effectiveness: Cloud services typically operate on a pay-as-you-go model, allowing businesses to avoid upfront costs of purchasing and maintaining servers. This can significantly reduce initial expenses and operational costs.
Disaster Recovery: Cloud providers often offer built-in disaster recovery solutions, ensuring that data is backed up regularly and can be quickly restored in case of any issues. Traditional setups may require additional investments in backup systems.

Now let's delve deeper into the technical aspects:

Resource Allocation: In a cloud environment, resources are abstracted from physical hardware through virtualization technologies. This means that multiple users or applications can share the same physical server without interfering with each other.
Service Models:
IaaS (Infrastructure as a Service): Provides bare-metal servers and storage over the internet. Users manage the operating system, middleware, and application software.
PaaS (Platform as a Service): Offers a platform for developers to build, run, and manage applications without worrying about infrastructure details.
SaaS (Software as a Service): Delivers complete software applications that are hosted on remote servers. Users access these applications via web browsers.

By leveraging cloud computing, businesses can focus more on their core operations rather than IT infrastructure management.

In conclusion, the primary difference between cloud computing and traditional on-premises hosting lies in scalability, cost-effectiveness, disaster recovery capabilities, and resource allocation methods.

**Comparison:**

```
Service Model	Description
IaaS	Users manage the operating system, middleware, and application software.
PaaS	Provides a platform for developers to build, run, and manage applications without worrying about infrastructure details.
SaaS	Delivers complete software applications that are hosted on remote servers. Users access these applications via web browsers.
```

**Code:**

```plaintext
// Example of a simple cloud setup using AWS S3 for storing static files

const aws = require('aws-sdk');

aws.config.update({
  accessKeyId: 'YOUR_ACCESS_KEY',
  secretAccessKey: 'YOUR_SECRET_KEY',
  region: 'us-east-1'
});

const s3 = new aws.S3();

// Upload a file
async function uploadFile(fileContent, fileName) {
  const params = {
    Bucket: 'your-bucket-name',
    Key: fileName,
    Body: fileContent
  };

  await s3.upload(params).promise();
}
```

**Example input / output:**

```text
Input:
- fileContent: Base64 encoded image data of an example.jpg
- fileName: example.jpg

Output:
- Successfully uploaded to S3 bucket 'your-bucket-name'
```

**Data:**

```json
{
  "bucketName": "your-bucket-name",
  "fileName": "example.jpg",
  "fileContent": "Base64 encoded image data"
}
```

**Flow:**

1. On-Premises Hosting
2. High Initial Costs
3. Maintenance and UpgradesRequired
4. Limited Scalability
5. Cloud Computing
6. Pay-As-You-Go Pricing
7. Elasticity/Scalability
8. Disaster Recovery Solutions

