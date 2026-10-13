---
type: "practice"
difficulty: "Basic"
---

# In simple terms, what is a rolling deployment?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ CI CD Pipeline Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_CD_Pipeline_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling deployments are a strategy used in CI/CD pipelines to update applications in production environments without causing service disruptions or downtime. Essentially, updates are rolled out gradually, one server at a time, allowing you to monitor and manage any issues that may arise.

To understand rolling deployments, we first need to break down the core concepts involved: continuous integration (CI) and continuous delivery (CD). CI ensures that code changes are continuously tested and integrated into the main branch of your repository. CD takes this a step further by automating the process of deploying those changes in a controlled manner.

Now let's dive into why rolling deployments matter. Imagine you have a web application running on 10 servers, each serving a portion of traffic. In a traditional "big bang" deployment, all servers are updated simultaneously, which can lead to downtime if something goes wrong. Rolling deployments mitigate this risk by updating one server at a time, monitoring the outcome, and only moving on to the next server once the previous one is confirmed to be stable.

Here’s how it works in practice:

Initiate Deployment: The CI/CD pipeline starts the deployment process.
Update One Server: A new version of the application is deployed to a single server.
Monitor for Issues: The system monitors this server closely for any errors or performance issues.
Test and Verify: Once the new version passes all checks, traffic is gradually redirected to it from other servers.
Repeat Until Complete: This process continues until all servers are updated.

Rolling deployments offer several benefits:

Reduced Downtime: By updating one server at a time, you minimize the risk of service interruptions.
Easier Rollbacks: If something goes wrong, you can quickly revert changes by stopping the deployment on the affected servers and rolling back to the previous version.
Improved User Experience: Users are less likely to notice any hiccups as traffic is smoothly redirected.

However, there are also potential downsides:

Increased Complexity: Managing a rolling deployment requires careful planning and monitoring.
Resource Intensive: Updating multiple servers one by one can take more time compared to a big bang approach.

In conclusion, rolling deployments are an essential part of modern CI/CD practices, providing a balanced solution between stability and agility. They allow teams to push updates confidently while maintaining high availability for end-users.

#### Common mistakes
to avoid include not thoroughly testing the application in staging environments before going live or not having adequate monitoring tools in place to catch issues early.

**Comparison:**

```
Deployment Phase	Description
Initiate	Start the CI/CD pipeline and prepare for deployment.
Update One Server	Deploy a new version to a single server.
Monitor	Closely watch this server for any issues.
Test	Verify that the application is functioning correctly.
Redirect Traffic	Gradually transfer traffic from old versions to the updated one.
```

**Example input / output:**

```text
This section is typically used for more complex scenarios, but we can illustrate a simple example of how the deployment might look in practice:

| Server | Old Version | New Version |
|--------|-------------|-------------|
| S1     | 1.0         | 2.0         |
| S2     | 1.0         | -           |
| S3     | 1.0         | -           |
| ...    | ...         | ...         |
| S10    | 1.0         | -           |

In this example, only `S1` has been updated to version `2.0`, while the rest remain on version `1.0`. Traffic would then be gradually redirected from older versions to new ones.
```

**Flow:**

1. Initiate Deployment
2. Update One Server
3. Monitor for Issues
4. Test and Verify
5. Redirect Traffic
6. Repeat Until Complete
7. Rollback if Needed

