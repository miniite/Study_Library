---
type: "practice"
difficulty: "Expert"
---

# How do you decide whether the gateway should own request/response transformation as an anti-corruption layer during a big backend rewrite, versus forcing clients to adapt; adjudicate the long-term maintenance cost of transformation-at-the-edge accumulating into an unremovable liability.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Idempotency and Org Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Idempotency_and_Org_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether an API Gateway should own request/response transformation as an anti-corruption layer during a backend rewrite, versus forcing clients to adapt, we must consider several key factors and long-term implications. The decision hinges on balancing immediate development costs with the ongoing maintenance burden.

In summary:

An anti-corruption layer in the gateway can shield clients from changes in internal API designs.
Long-term maintenance of such a transformation logic can become unwieldy if not carefully managed.
We need to weigh these against the cost and complexity of requiring clients to adapt.

Let’s break down the reasoning step by step:

#### Triage

The question is essentially asking us to choose between:

Gateway Transformation: Implementing request/response transformations in the API Gateway, which can act as an anti-corruption layer.
Client Adaptation: Forcing clients to handle any necessary data transformations themselves.

We are being probed on our understanding of the trade-offs involved and the potential long-term liabilities. Assumptions include:

The backend is undergoing a major rewrite, implying significant internal changes.
The API Gateway will be maintained as part of the overall architecture.
Clients have varying levels of development capabilities and may need different degrees of support.
First Principles
Anti-Corruption Layer: An anti-corruption layer in the gateway ensures that changes to the backend do not directly impact client applications, reducing the risk of client code breaking due to internal API modifications.
Gateway Complexity: The gateway will handle more logic and may become complex over time, increasing maintenance costs.
Client Adaptation: Clients must adapt their code when APIs change, but this can be managed through clear documentation and versioning.
Step-by-Step Solution

Assess Internal Changes:

Evaluate the scale and frequency of internal API changes.
Determine if these changes are likely to be frequent enough to justify an anti-corruption layer.

Client Support Needs:

#### Cons
ider the complexity of data transformations required by clients.
Assess whether most clients can handle these transformations without significant additional effort.

Gateway Design and Maintenance:

Plan for a modular and extensible gateway architecture that can accommodate various transformation needs.
Estimate the development and maintenance costs associated with maintaining such logic over time.

Client Training and Documentation:

Develop comprehensive documentation and training materials to assist clients in adapting their code.
Provide clear guidance on how to handle API versioning and backward compatibility.

Risk Analysis:

Assess the risk of client applications breaking due to internal API changes if no anti-corruption layer is implemented.
Weigh this against the risk of increased maintenance costs for the gateway.

Pilot Implementation:

#### Cons
ider implementing a pilot transformation in the gateway to evaluate its effectiveness and cost before full-scale deployment.
Use metrics such as client feedback, error rates, and performance impact to inform final decisions.

#### Closing Summary

Deciding whether an API Gateway should own request/response transformations involves careful consideration of both short-term development costs and long-term maintenance liabilities. An anti-corruption layer can provide significant protection against internal changes but comes with its own set of challenges in terms of complexity and ongoing support. By carefully evaluating the specific context, requirements, and client needs, we can make an informed decision that balances these factors effectively.

#### Common Mistakes/Follow-Ups
Overcomplicating the Gateway: Failing to keep the gateway design simple and modular can lead to increased maintenance costs.
Ignoring Client Needs: Not adequately addressing client support and adaptation requirements can result in suboptimal outcomes.
Lack of Metrics: Failing to gather and analyze metrics on the effectiveness and impact of transformations can hinder informed decision-making.

