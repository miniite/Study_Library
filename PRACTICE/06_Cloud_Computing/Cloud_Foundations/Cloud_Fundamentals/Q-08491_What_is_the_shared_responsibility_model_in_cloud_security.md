---
type: "practice"
difficulty: "Intermediate"
---

# What is the shared responsibility model in cloud security?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The shared responsibility model in cloud security is a fundamental concept that clarifies how cloud service providers (CSPs) and their customers share responsibilities to secure a cloud-based environment. In essence, while CSPs are responsible for securing the underlying infrastructure, the customer is responsible for securing their own applications, data, and configurations. This division of responsibilities ensures both parties have clear lines of accountability.

#### Triaging the Question

The interviewer is likely probing your understanding of how cloud security works, particularly in terms of dividing responsibilities between the CSP and the user. Key assumptions include:

Understanding that cloud services are multi-tenant environments.
Recognizing that different components of a cloud deployment (e.g., hardware, network, applications) may require different levels of security oversight.

#### First Principles Solution

CSP Responsibility:

Infrastructure Security: The CSP is responsible for securing the underlying physical and virtual infrastructure, including data centers, networks, firewalls, and hypervisors.
Platform Management Services: This includes managed services like load balancers, databases, and storage systems that are provided by the CSP. They also manage updates and patches to these services.
Customer Support and Security Operations: The CSP provides tools and support for customers to monitor their environments and address security incidents.

Customer Responsibility:

Application and Data Security: Customers must secure their applications, data, and configurations within the cloud environment. This includes implementing access controls, encryption, and other security measures.
Network Security: Customers are responsible for securing network traffic that moves in and out of their cloud resources.
Operational Controls: Ensuring compliance with regulatory requirements and adhering to best practices is also a customer responsibility.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the shared responsibility model ensures that both the CSP and the user collaborate effectively to secure the cloud environment. The key takeaway is that while the CSP provides robust security measures for the underlying infrastructure, customers must still take active steps to secure their applications and data.

#### Common mistakes
include:

Over-relying on the CSP's security mechanisms without implementing additional customer-side controls.
Failing to properly configure security settings within managed services provided by the CSP.
Ignoring the need for ongoing monitoring and compliance checks.

#### Follow-up
questions might explore specific examples of CSP vs. customer responsibilities, such as handling data breaches or managing access control policies.

**Comparison:**

|Component           | CSP Responsibility             | Customer Responsibility               |
|--------------------|--------------------------------|--------------------------------------|
| Hardware Security  | √                               | X                                    |
| Network Security   | √                               | √                                    |
| Data Center        | √                               | X                                    |
| Platform Services  | √                               | √                                    |
| Application Layer  | X                               | √                                    |

**Code:**

```python
# Example: Configuring network security groups (NSGs)
def configure_nsg_rules(nsg_id, rules):
    # This function would interact with the cloud provider's API to apply NSG rules
    pass

# Example: Implementing encryption for data at rest and in transit
def implement_encryption(data, encryption_key):
    encrypted_data = encrypt(data, encryption_key)
    return encrypted_data

def decrypt(encrypted_data, encryption_key):
    # Decryption logic would be implemented here
    pass
```

**Example input / output:**

```text
Input:
nsg_id: nsgh-123456
rules: [
  {
    "direction": "Inbound",
    "protocol": "TCP",
    "source_addresses": ["0.0.0.0/0"],
    "destination_ports": [80, 443],
    "actions": [{"type": "Allow"}]
  },
  {
    "direction": "Outbound",
    "protocol": "UDP",
    "source_addresses": ["192.168.1.0/24"],
    "destination_ports": [53, 67],
    "actions": [{"type": "Deny"}]
  }
]

Output:
nsg_id: nsgh-123456
rules: [
  {
    "direction": "Inbound",
    "protocol": "TCP",
    "source_addresses": ["0.0.0.0/0"],
    "destination_ports": [80, 443],
    "actions": [{"type": "Allow"}]
  },
  {
    "direction": "Outbound",
    "protocol": "UDP",
    "source_addresses": ["192.168.1.0/24"],
    "destination_ports": [53, 67],
    "actions": [{"type": "Deny"}]
  }
]
```

**Data:**

```json
{
  "nsg_id": "nsgh-123456",
  "rules": [
    {
      "direction": "Inbound",
      "protocol": "TCP",
      "source_addresses": ["0.0.0.0/0"],
      "destination_ports": [80, 443],
      "actions": [{"type": "Allow"}]
    },
    {
      "direction": "Outbound",
      "protocol": "UDP",
      "source_addresses": ["192.168.1.0/24"],
      "destination_ports": [53, 67],
      "actions": [{"type": "Deny"}]
    }
  ]
}
```

**Flow:**

1. Manages Infrastructure
2. Configures and Secures
3. Cloud Service Provider
4. (Hardware, Network, Platform)
5. Customer
6. (Application, Data,AccessControls)

