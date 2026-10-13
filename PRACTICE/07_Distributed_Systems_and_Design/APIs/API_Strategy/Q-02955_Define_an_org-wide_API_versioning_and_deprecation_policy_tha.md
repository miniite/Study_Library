---
type: "practice"
difficulty: "Expert"
---

# Define an org-wide API versioning and deprecation policy that spans 40 teams. How do you reconcile teams that want URI versioning, header versioning, and no-versioning-ever, and what deprecation-timeline SLAs would you make binding across the whole org?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When crafting an API strategy and governance policy at a large organization with 40+ teams, the key is to establish a consistent yet flexible approach that accommodates different team preferences while ensuring backward compatibility and future-proofing. The overarching goal is to balance innovation with stability across the entire org.

The question revolves around reconciling various versioning strategies (URI versioning, header versioning, and no-versioning) and setting binding deprecation timelines. Here’s a step-by-step approach from first principles:

#### Triage
Understanding the Question
Versioning Approaches: Different teams may have varying opinions on how to manage API versions—some prefer URI-based versioning (e.g., api/v1/endpoint), others might opt for header-based versioning (e.g., X-API-Version: 1), and still, some may argue against any explicit versioning.
Deprecation Policies: Teams need to agree on a timeline or SLA for deprecating old APIs, ensuring that backward compatibility is maintained while promoting new features.

#### Assumptions
All teams use RESTful APIs with well-defined schemas and data formats.
The organization has a common set of security and compliance standards.
There’s existing communication channels (e.g., Slack, Teams) for discussions and approvals.

#### First Principles Solution

Unified Versioning Policy:

Recommend URI Versioning: While it might seem more explicit at first glance, URI versioning is generally easier to implement and manage across a wide range of client types.
Hybrid Approach: Allow header-based versioning for clients that require finer control over API requests (e.g., mobile apps or legacy systems). This can coexist with URI versioning by providing both api/v1/endpoint and X-API-Version: 1.

Deprecation Timeline SLA:

Mandatory Deprecation Window: Set a mandatory deprecation period of at least 6-9 months for old APIs to allow teams time to update their clients.
Grace Periods: For critical systems or long-term dependencies, consider extending the deprecation window up to two years with additional justification from the team.
Communication Channels: Ensure all API consumers are informed about upcoming changes via announcements in common communication channels and versioning documentation.

Governance Body:

Establish a central governance body (e.g., an API Governance Committee) that includes representatives from key teams. This body will make final decisions on API policies, versioning strategies, and deprecation timelines.
Regular meetings to discuss evolving requirements and align all teams.

API Lifecycle Management:

Implement tools for tracking API versions and their states (e.g., draft, beta, stable).
Use a CI/CD pipeline that includes checks for version compliance and deprecation readiness.
Automate notifications for developers when an API is going to be deprecated.

#### Closing Summary

By adopting a hybrid approach, allowing flexibility where needed but maintaining a clear and consistent policy, we can ensure that different teams’ needs are addressed while preserving the overall stability and future scalability of our APIs. The governance body will help in making informed decisions that balance innovation with backward compatibility.

#### Common Mistakes/Follow-Ups
Not Having Clear SLAs: Teams often rush to deprecate older versions without proper planning, leading to client-side issues.
Ignoring User Feedback: Failing to involve key stakeholders and get buy-in from teams can result in resistance and longer implementation times.
Overly Complex Versioning Schemes: While some might argue for intricate versioning strategies, simpler is usually better for maintaining consistency across the org.

**Comparison:**

```
API Versioning Strategy	Description
URI	Explicitly versioned in URLs (e.g., api/v1)
Header	Explicitly versioned via headers
No-version	No explicit versioning, implicit backward compat.
```

**Code:**

```python
# Example of a simple API versioning policy check
def validate_api_version(version_str):
    if not version_str.startswith('v'):
        raise ValueError("API version must start with 'v'")
    return True
```

**Example input / output:**

```text
Input:
- New team requests URI versioning for a new service.
- Existing teams have varying preferences (URI, header, no-version).

Output:
- Unified policy decides to use URI versioning as the default with header-based options where needed.
- Established deprecation window of 6 months is communicated to all relevant parties.
```

**Data:**

```json
{
  "api_versions": {
    "uri": "/api/v1",
    "header": "X-API-Version: 1"
  },
  "deprecation_window": "6-9 months",
  "communication_channels": [
    "Slack channel #api-updates",
    "Company intranet API documentation page"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. API Request
6. Is URI Versioned?
7. Process URI
8. Is Header Versioned?
9. Process Header
10. No Version, Use DefaultHandling

