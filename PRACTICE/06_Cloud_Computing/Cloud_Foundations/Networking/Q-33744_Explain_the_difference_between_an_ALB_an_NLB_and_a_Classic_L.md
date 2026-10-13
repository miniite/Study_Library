---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between an ALB, an NLB, and a Classic LB in terms of the OSI layer they operate at and what that means for TLS termination.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ mTLS and Zero Trust](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/mTLS_and_Zero_Trust.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between Application Load Balancers (ALBs), Network Load Balancers (NLBs), and Classic Load Balancers, it's crucial to consider where they operate in the OSI model and what implications that has for TLS termination.

TL;DR:
Classic LB: Operates at Layer 4 of the OSI model, handling TCP-level load balancing without SSL termination.
NLB: Also operates at Layer 4, but can handle both TCP and UDP traffic and offers more flexibility in configuration.
ALB: Operates at Layer 7, providing HTTP/HTTPS load balancing with built-in SSL termination.

#### Triaging the Question

The question asks us to differentiate these types of load balancers based on their position in the OSI model (Layer 4 vs. Layer 7) and what that means for TLS termination. This requires a deep understanding of how each type processes traffic at its respective layer, as well as the implications of SSL/TLS handling.

#### First-Principles Solution

Classic Load Balancer (Layer 4):

Operates at the Transport Layer (Layer 4) of the OSI model.
Handles only TCP-based load balancing.
Does not perform any SSL termination; it passes encrypted traffic directly to backend instances.

Network Load Balancer (NLB):

Also operates at Layer 4, similar to Classic LB.
Supports both TCP and UDP protocols.
Offers more advanced features like health checks for more robust monitoring of backend instances.

Application Load Balancer (ALB):

Operates at the Application Layer (Layer 7) of the OSI model.
Provides HTTP/HTTPS load balancing with built-in SSL termination capabilities.
Terminates SSL/TLS encryption before passing the decrypted request to the backend servers, allowing for further processing within the application tier.

#### Closing Summary

The primary difference between these load balancers lies in their position in the OSI model and how they handle network traffic. Classic LBs are suitable for simple TCP-based applications, while NLBs offer more flexibility with both TCP and UDP protocols. ALBs, on the other hand, provide advanced features like Layer 7 processing and built-in SSL termination, making them ideal for modern web applications that require HTTP/HTTPS support.

#### Common Mistakes
and Follow-Ups:
Mistake: Assuming all load balancers operate at the same layer in the OSI model. NLBs and ALBs offer different levels of abstraction.

#### Follow-Up
Discussing scenarios where SSL termination might be beneficial or detrimental, such as compliance requirements or performance considerations.
Triaging Summary (continued):

For a deeper understanding, consider these follow-up questions:

What are the implications for security when choosing an ALB over a Classic LB in terms of data exposure?
How does NLB's ability to handle both TCP and UDP impact its use cases compared to other load balancers?

**Comparison:**

| Load Balancer Type | Layer in OSI Model | SSL Termination Capabilities |
|--------------------|-------------------|------------------------------|
| Classic LB         | Layer 4           | No                           |
| NLB                | Layer 4           | Optional                     |
| ALB                | Layer 7           | Built-in                     |

**Code:**

```python
# Example code snippet for configuring SSL termination on an ALB
import boto3

def create_alb_listener(region, certificate_arn):
    client = boto3.client('elbv2', region_name=region)
    response = client.create_listener(
        LoadBalancerArn='arn:aws:elasticloadbalancing:<region>:<account-id>:load_balancer/<lb-name>',
        Protocol='HTTPS',
        Port=443,
        DefaultActions=[{'Type': 'forward', 'TargetGroupArn': '<target-group-arn>'}],
        SslPolicy='ELBSecurityPolicy-2016-08',
        Certificates=[{'CertificateArn': certificate_arn}]
    )
    return response
```

**Example input / output:**

```text
Input:
- Load Balancer ARN: arn:aws:elasticloadbalancing:us-west-2:123456789012:load_balancer/lb-123456
- Certificate ARN: arn:aws:acm:us-west-2:123456789012:certificate/cert-123456

Output:
{
  "Listener": {
    "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-west-2:123456789012:load_balancer/lb-123456",
    "Protocol": "HTTPS",
    "Port": 443,
    "Certificates": [
      {
        "CertificateArn": "arn:aws:acm:us-west-2:123456789012:certificate/cert-123456"
      }
    ],
    "DefaultActions": [
      {
        "Type": "forward",
        "TargetGroupArn": "<target-group-arn>"
      }
    ]
  },
  "ResponseMetadata": {
    "RequestId": "7a62fe4c-974f-11e5-b3d9-c1bfecad0ed8"
  }
}
```

**Data:**

```json
{
  "listeners": [
    {
      "Protocol": "HTTPS",
      "Port": 443,
      "Certificates": [
        {
          "CertificateArn": "arn:aws:acm:<region>:<account-id>:certificate/<certificate-id>"
        }
      ],
      "DefaultActions": [
        {
          "Type": "forward",
          "TargetGroupArn": "<target-group-arn>"
        }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User
4. ALB (Layer 7)
5. Is SSL Termination Required?
6. ALB Terminates SSL
7. ALB Passes Traffic to Backend
8. Backend Server
9. Application Processing

