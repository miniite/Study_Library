---
type: "practice"
difficulty: "Intermediate"
---

# A downstream REST dependency publishes a new major version and will EOL the old one in 90 days. How do you plan the migration for a service that calls it in 40 places, including canary and rollback strategy?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When facing a situation where an external REST API is publishing a new major version with an EOL date of 90 days from now, the key challenge lies in migrating our downstream service to use the new API while ensuring zero downtime and minimal impact on end users. This involves planning a strategy that balances risk, testing, and minimizing disruptions.

#### Triaging the Question

The core question here is about developing a migration plan for an existing dependency with multiple calls (40 places), including a canary deployment and rollback mechanism. The key aspects to consider are:

Assessing the impact of each API change: Understanding what changes were introduced in the new version.
Setting up a canary rollout process: Gradually introducing the new API to a subset of users to detect issues early.
Implementing a rollback strategy: Preparing a plan to revert quickly if any issues arise during the migration.

#### First Principles Solution
Understand the New API Changes:
Review the documentation for the new major version to understand what has changed, which endpoints are deprecated, and how they need to be updated.
Identify and Prioritize Calls:
Catalog all 40 places where our service calls this external API.
Prioritize these based on criticality, usage patterns, and dependencies within the system.
Develop a Migration Plan:
Create a timeline with key milestones: start updating one endpoint at a time.
Allocate sufficient time for testing each updated call before moving to the next one.
Implement Canary Rollout:
Introduce the new API version gradually using A/B testing or load balancer rules.
Monitor these users closely for any unexpected behavior, performance issues, or errors.
Prepare a Rollback Plan:
Ensure that we can revert quickly if something goes wrong during the migration.
Maintain a backup of the old code and configuration in case of rollback.
Detailed Steps
API Documentation Review:
MARKDOWN
- New API version documentation includes new endpoints, changes to existing ones, and deprecation notices.
Dependency Cataloging:
Use version control history or logging tools to identify all places where the old API is called.
Prioritize by criticality (e.g., authentication, payment processing) and impact on user experience.
Incremental Migration:
For each updated endpoint, modify the service code to handle both the new and old versions temporarily.
Identify API calls
Review documentation
Catalog endpoints
Prioritize updates
Update one endpoint at a time
Test in staging
Migrate next endpoint
Canary Rollout:
Use A/B testing or load balancer rules to gradually redirect traffic from the old API to the new.
Configure Load Balancer
Canary Traffic
Monitor performance anderrors
Continue migration
Rollback Plan:
Keep a backup of the old code and configuration.
Test rollback procedures in a controlled environment.

#### Closing Summary

The plan involves understanding the API changes, cataloging dependencies, incrementally updating one endpoint at a time, implementing a canary rollout process, and preparing a robust rollback strategy. This ensures that we minimize risk and maintain service stability during the transition.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring critical endpoints or failing to prioritize them adequately.
Mistake 2: Not having a thorough test plan in place, leading to production issues.
Mistake 3: Forgetting to document the rollback procedure and keep it up-to-date.

**Comparison:**

```
Priority	Endpoint	Criticality
High	Auth API	Critical
Medium	Payment	Critical
Low	Stats API	Non-critical
```

**Flow:**

1. Start
2. Review Docs
3. Catalog Endpoints
4. Prioritize
5. Migrate One at a Time
6. Test Staging
7. Roll Out Canaries
8. Monitor
9. Continue Migration

