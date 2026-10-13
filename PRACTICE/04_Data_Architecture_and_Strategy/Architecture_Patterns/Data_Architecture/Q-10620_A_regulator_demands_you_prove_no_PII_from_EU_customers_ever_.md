---
type: "practice"
difficulty: "Expert"
---

# A regulator demands you prove no PII from EU customers ever entered a US-hosted dataset. Given a sprawling lake, how do you architect for provable data-flow containment rather than after-the-fact scanning?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to design a robust architecture that ensures personal identifiable information (PII) from EU customers never enters US-hosted datasets. The key concept here is "provable data-flow containment" rather than relying on post-facto scanning for compliance. This involves careful segmentation of data flows and employing robust security mechanisms.

First, let's triage the question:

What the Question is Asking: We need to ensure that no PII from EU customers ever enters US-hosted datasets.
What the Interviewer is Probing: The interviewer is probing our understanding of data governance, secure data flow, and compliance mechanisms in a distributed environment.

#### Assumptions
The lake contains diverse data sources including those with potential PII from EU customers.
We need to ensure that this data does not cross the US-EU border.
Compliance is critical for regulatory reasons.

The solution involves designing an architecture that enforces strict data flow controls and employs cryptographic techniques for secure transmission. Here's a step-by-step approach:

Data Segmentation: Segment the lake into distinct regions or tiers based on geographic origin to ensure that data from EU customers remains in dedicated, isolated zones.
Data Encryption at Rest and In Transit: Use strong encryption mechanisms to protect PII both when it is stored and when it is being transferred between nodes or regions.
Access Controls and Identity Management: Implement fine-grained access controls using identity and access management (IAM) systems that enforce strict rules on who can access data from which region.
Data Movement Control: Employ network segmentation techniques to control the flow of data, ensuring that it only moves along predefined secure pathways.

The architecture would look something like this:

Data Flow
No
Yes
Pass
Fail
US Lake
Is Data EU Originated?
Drop Non-EU Data
EU Lake
Compliance Check
Store PII Securely
Reject Data
Data Isolation Violation Alert

#### First-Principles Solution
Geographic Segmentation: The lake is divided into two primary segments: one for US-hosted data and another for EU-hosted data. Each segment is managed by its own set of controls.
Encryption: All data in transit between the lakes, as well as at rest, is encrypted using industry-standard encryption algorithms such as AES-256. This ensures that even if data leaks, it cannot be accessed without proper decryption keys.
Access Controls: IAM systems enforce strict rules based on geographic origin. Only authorized EU entities can access EU-hosted PII, and similarly, only US entities can interact with US-hosted data.
Compliance Monitoring: A compliance monitoring layer is implemented to continuously check for any violations of data flow policies. This layer generates alerts if unauthorized data movement is detected.

#### Closing Summary

This architecture ensures that no PII from EU customers ever enters the US-hosted dataset by employing robust geographic segmentation, encryption, access controls, and continuous monitoring. The design is scalable and can be extended to include additional compliance requirements as needed.

#### Common Mistakes/Follow-Ups

Not properly segmenting data based on origin.
Failing to implement strong encryption mechanisms.
Overlooking the importance of fine-grained access control.
Ignoring the need for continuous monitoring and alerting systems.

**Comparison:**

```
Region	Data Type	Origin
US	PII	US
US	Non-PII	US
EU	PII	EU
EU	Non-PII	EU
```

**Code:**

```python
# Example Code for Access Control Rules
import json

def check_access(data_origin, user_role):
    if data_origin == "EU" and user_role != "EU_Access_Role":
        raise Exception("Access Violation")
    return True

# Example Usage
user = {"role": "US_Admin"}
data_origin = "EU"
try:
    check_access(data_origin, user["role"])
except Exception as e:
    print(e)
```

**Example input / output:**

```text
Input:
- User with role "EU_Access_Role" tries to access a dataset in the US Lake.

Output:
- Access is denied due to mismatched roles and origin.
```

**Data:**

```yaml
# Sample Data Schema for EU and US Lakes
lakes:
  us_lake:
    - data_type: PII
      origin: US
    - data_type: Non-PII
      origin: US
  eu_lake:
    - data_type: PII
      origin: EU
    - data_type: Non-PII
      origin: EU
```

**Flow:**

1. Data Flow
2. No
3. Yes
4. Pass
5. Fail
6. US Lake
7. Is Data EU Originated?
8. Drop Non-EU Data
9. EU Lake
10. Compliance Check
11. Store PII Securely
12. Reject Data
13. Data Isolation Violation Alert

