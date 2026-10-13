---
type: "practice"
difficulty: "Expert"
---

# You must define what 'event ownership' means org-wide: who owns a topic's schema, its retention, its SLA, and its consumers-of-record. Draft the ownership contract and the escalation path when a producer wants a breaking change that N consumers depend on.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choreography vs Orchestration](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Choreography_vs_Orchestration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In an event-driven architecture, 'event ownership' refers to defining clear responsibilities within an organization for various aspects of topics, such as schema management, retention policies, service-level agreements (SLAs), and consumer management. This ensures consistency, reliability, and traceability in the system. The contract defines roles and responsibilities while the escalation path outlines a structured approach to handle breaking changes requested by producers.

#### Triaging the Question

The question is probing our understanding of event ownership within an organization and how it impacts various critical aspects like schema management, retention policies, SLAs, and consumer management. We need to define a clear contract that outlines roles and responsibilities for each aspect and establish a structured escalation path when a producer requests a breaking change affecting multiple consumers.

#### Assumptions
The system uses message queues and an event-driven architecture.

#### Cons
umers are registered with the queue service as 'consumers-of-record'.
Breaking changes refer to significant modifications that may impact existing consumers.
The organization has well-defined roles, such as DevOps engineers, application owners, and support teams.
Step-by-Step Solution
1. Defining Event Ownership

Schema Management:

Owner: Application Owner
Responsibilities:
Define and maintain the schema of each topic.
Ensure compatibility with existing consumers.
Communicate any planned changes to the DevOps team.

Retention Policies:

Owner: Operations Team
Responsibilities:
Set and adjust retention policies based on business needs.
Monitor storage usage and optimize as necessary.
Report storage usage to relevant stakeholders.

Service-Level Agreements (SLAs):

Owner: DevOps Engineer
Responsibilities:
Define SLAs for each topic, including latency targets, delivery guarantees, and error rates.
Monitor SLA compliance and take corrective actions as needed.
Communicate SLA status to application owners.

#### Cons
umer Management:

Owners: Application Owners and Consumer Teams
Responsibilities:
Register consumers with the queue service as 'consumers-of-record'.
Ensure that all consumers are aware of changes in schema or retention policies.
Provide feedback on breaking changes impacting their services.
2. Ownership Contract

The ownership contract should be formalized and documented, detailing responsibilities, communication protocols, and escalation procedures.

Ownership Contract:

CODE
1. Application Owner
   - Schema Management: Define and maintain the schema of each topic.
   - Communication: Inform DevOps team about planned changes.

2. Operations Team
   - Retention Policies: Set and adjust retention policies based on business needs.
   - Reporting: Monitor storage usage and report to relevant stakeholders.

3. DevOps Engineer
   - SLA Management: Define, monitor, and enforce SLAs for each topic.
   - Compliance: Take corrective actions if SLA targets are not met.
   - Communication: Report SLA status to application owners.

4. Application Owners and Consumer Teams
   - Consumer Registration: Register consumers as 'consumers-of-record'.
   - Feedback: Provide feedback on breaking changes impacting their services.
3. Escalation Path

When a producer requests a breaking change affecting multiple consumers, the following steps should be followed:

Initial Request:

The producer submits a formal request to the DevOps team, detailing the proposed change and its impact on existing consumers.

Impact Assessment:

DevOps Engineers assess the impact of the change on current consumers.
They communicate their findings to the application owners and consumer teams involved.

Approval Process:

A joint approval process is initiated involving application owners, DevOps engineers, and possibly business stakeholders.
Any necessary approvals are obtained before proceeding with the change.

Change Implementation:

The change is implemented by the DevOps team while ensuring minimal disruption to ongoing operations.

#### Cons
umers-of-record are notified of the upcoming changes, providing time for them to prepare or migrate if needed.

Post-Implementation Review:

Post-implementation review is conducted to ensure that the change met its SLA targets and did not introduce any new issues.
Feedback from consumers is collected to improve future processes.
4. Closing Summary and Common Mistakes

Summary: The ownership contract clearly defines roles and responsibilities for managing topics within an event-driven architecture, ensuring consistency and reliability. The structured escalation path minimizes disruptions by involving relevant stakeholders in the decision-making process.

#### Common Mistakes/Follow-Ups

Failing to involve all affected parties in the approval process.
Overlooking the need for detailed impact assessments before implementing changes.
Not having a clear post-implementation review and feedback loop, leading to recurring issues.

#### Closing Summary

By defining event ownership through a well-documented contract and establishing a structured escalation path, we ensure that critical aspects of topics are managed efficiently and effectively. This approach minimizes disruptions, enhances reliability, and maintains the trust of consumers in the system.

**Comparison:**

```
Role	Responsibilities
Application Owner	Schema Management, Communication with DevOps Team
Operations Team	Retention Policies, Storage Usage Monitoring
DevOps Engineer	SLA Management, Compliance Monitoring
Application Owners	Consumer Registration, Feedback on Breaking Changes
```

**Code:**

```python
# Example code for registering consumers as 'consumers-of-record'
def register_consumer(topic_name, consumer_id):
    # Implement registration logic here
    pass

register_consumer("example-topic", "consumer-123")
```

**Example input / output:**

```text
Input:
- Producer requests a breaking change in the schema of 'example-topic'.

Output:
1. DevOps team receives and assesses the request.
2. Application owners and consumer teams are notified of potential impacts.
3. Joint approval process initiated.
4. Change implemented by DevOps engineers with minimal disruption.
5. Post-implementation review conducted to ensure SLA compliance.
```

**Data:**

```json
{
  "topics": [
    {
      "name": "example-topic",
      "owner": "application-owner-team@acme.com",
      "retention_policy_owner": "operations-team@acme.com",
      "sla_owner": "devops-engineer@acme.com"
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Initial Request
5. Approval Required?
6. Joint Approval Process
7. Change Implementation
8. Post-Implementation Review?
9. Feedback Loop

