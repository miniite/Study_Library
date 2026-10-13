---
type: "practice"
difficulty: "Deep"
---

# A legacy monolith can only read secrets from a file at boot and can't hot-reload. Design a rotation strategy that still achieves near-continuous rotation without unbounded restart storms across the fleet.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Secret Rotation](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Shared_Secret_Rotation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we need to design a secrets management solution for a legacy monolith application that can only read secrets from a static file at boot and cannot hot-reload them. The goal is to achieve near-continuous rotation of secrets without causing unbounded restart storms across the entire fleet.

#### Triage
The key challenges are:

Static Secrets Reading: The application reads its secrets from a file at startup, but does not support dynamic reloading.
Continuous Rotation: We need frequent updates to secrets while minimizing disruption in the application's operation.
Unbounded Restart Storms: Frequent restarts of all instances could lead to downtime and performance issues.

#### Solution

Introduce a Secrets Proxy Service: Deploy a lightweight proxy service that fetches secrets from a secure secrets management system (e.g., Hashicorp Vault) and serves them to the monolith application.
Use a Caching Mechanism: The proxy can cache the secrets for a short period, allowing the application to use the cached secrets until the next update without needing to restart.
Scheduled Secret Updates: Schedule periodic updates of the secrets via the proxy service while ensuring minimal disruption by staggering the updates across instances.

Step-by-Step Solution:

Deploy Vault and Proxy Service:

Set up Hashicorp Vault as the secrets management backend.
Deploy a lightweight proxy service (e.g., using Node.js or Python) that communicates with Vault to fetch secrets.

Caching Mechanism:

Configure the proxy to cache secrets for a short duration, say 5-10 minutes.
This allows the application to continue running without needing an immediate restart during the secret rotation period.

Scheduled Secret Updates:

Schedule periodic updates of the secrets at regular intervals (e.g., every hour).
Ensure that these updates are staggered across instances to minimize simultaneous changes and avoid a "wave" effect.
Implement a small window for updates, e.g., 5-10 minutes, during which only a subset of instances are updated.

Health Checks:

Implement health checks on the proxy service to ensure it is properly caching and serving secrets.
Monitor the application for any issues that might arise due to secret changes.

#### Closing Summary
This strategy leverages a lightweight proxy service to fetch secrets from Vault, cache them locally, and provide near-continuous rotation with minimal disruption. By staggering updates across instances and implementing health checks, we can ensure smooth operation while maintaining security best practices.

#### Common Mistakes/Follow-Ups

Not Scheduling Updates: Not properly scheduling secret updates can lead to uncontrolled restarts.
Failing Health Checks: Poorly implemented health checks can result in incorrect caching or serving of secrets, leading to application issues.
Staggering Too Aggressively: Staggering too aggressively might not provide sufficient coverage for all instances during a single update cycle.

