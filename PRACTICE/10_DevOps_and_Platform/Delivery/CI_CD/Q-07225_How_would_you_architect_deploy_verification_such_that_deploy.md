---
type: "practice"
difficulty: "Expert"
---

# How would you architect deploy verification such that 'deploy done' provably means 'the new version is serving correct traffic in every region', while keeping that verification fast enough not to become the new bottleneck?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a deployment has completed successfully and that the new version of an application is serving the correct traffic across all regions, we need to architect our CI/CD pipeline in such a way that it not only handles the deployment but also verifies its success comprehensively. The challenge lies in ensuring that this verification process remains efficient enough to avoid becoming a bottleneck.

#### Triaging the Question

The question asks for an architecture that ensures 'deploy done' means "the new version is serving correct traffic in every region" while keeping the verification fast enough not to slow down the deployment pipeline. This involves several key components: understanding the scope of regions and applications, ensuring consistent verification across environments, and optimizing the verification process.

Key Assumptions
Multiple Regions: The application needs to be deployed and verified in multiple geographic regions.
Version Control: Each version of the application is uniquely identifiable.
Traffic Monitoring: We need a mechanism to monitor traffic and ensure it's directed correctly post-deployment.
Performance Constraints: Verification must not significantly delay the deployment process.

#### Solution
Step-by-Step Approach

Environment Setup:

Ensure that each region has an identical environment setup for the application.
Use consistent configuration management tools (e.g., Ansible, Terraform) to manage environments across regions.

Deployment Strategy:

Implement a blue-green deployment strategy where new versions are deployed in a separate environment and traffic is directed there gradually.
This ensures that we can roll back immediately if issues arise without affecting production users.

Traffic Splitting:

Use load balancers or cloud-native services (e.g., AWS ALB, Kubernetes Ingress) to split traffic between old and new versions of the application.
Implement a health check mechanism at the load balancer level to ensure only healthy instances receive traffic.

Verification Mechanism:

Create a verification job that runs after each deployment in every region.
This job should validate that the correct version is serving requests by checking logs, metrics, or custom assertions.
Use automated tests (e.g., API calls, load testing) to ensure functionality remains stable.

Orchestration:

Utilize CI/CD tools like Jenkins, GitLab CI, or CircleCI to orchestrate the deployment and verification pipeline.
Ensure that these tools can trigger both deployments and verification jobs seamlessly across all regions.

Monitoring and Logging:

Set up comprehensive monitoring for each region using tools like Prometheus, Grafana.
Implement logging mechanisms (e.g., ELK stack) to capture detailed logs during deployment and verification phases.

Optimization:

Optimize the verification process by parallelizing tasks where possible.
Use caching or memoization techniques if applicable to reduce redundant checks.

#### Example
Architecture
Pass
Pass
Pass
Code Repository
CI/CD Pipeline
Deployment to Staging inRegion 1
Health Check and LogVerification
Deployment to Staging inRegion 2
Health Check and LogVerification
Deployment to Staging inRegion 3
Health Check and LogVerification
Mark Deployment as Done

#### Closing Summary

By following this architecture, we ensure that each deployment is verified in all regions before marking it as complete. This approach uses consistent setup across environments, leverages modern CI/CD practices, and includes comprehensive monitoring to guarantee correctness.

#### Common Mistakes
and Follow-Ups
Ignoring Health Checks: Failing to implement proper health checks can lead to undetected issues.
Lack of Parallelism: Not parallelizing verification tasks can slow down the entire process.
Overly Complex Verification Logic: Over-engineering the verification logic without considering performance impacts.

By addressing these points, we ensure that our deployment verification is robust yet efficient.

