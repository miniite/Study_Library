---
type: "practice"
difficulty: "Basic"
---

# What does 'zero-downtime deploy' mean and why do users care about it?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Zero-downtime deployment (ZDT) in DevOps is a crucial practice where new code or updates are seamlessly integrated into production without causing any disruption to end-users. This means that during deployments, users can continue to access and use the application as if nothing was happening behind the scenes. Users care about ZDT for several reasons: it ensures high availability, improves user experience, and minimizes potential business impacts.

To understand why zero-downtime deployment is important, let's break down the concept from first principles.

**Comparison:**

```
Deployment Method	Description
Zero-downtime	Ensures no service disruption during deployment
Blue-Green	Deploys to a separate environment and swaps traffic later
Canary Release	Tests on a small user base before full rollout
```

**Code:**

```python
# Example of a simple function to illustrate deployment without downtime
def process_request(request):
    return f"Request processed at {datetime.now()}"

def deploy_new_version(current_function):
    # Simulate switching to new code version
    old_function = current_function
    def new_function(request):
        if random.random() < 0.5:
            return old_function(request)
        else:
            return "Processing new version..."
    return new_function

current_function = process_request
new_version = deploy_new_version(current_function)

# Example usage
print(new_version("Sample request"))  # May print either the old or new processing logic
```

**Example input / output:**

```text
Input:
- Old version: process_request(request)
- New version: new_function(request)

Output (example):
1. Request processed at 2023-10-01 14:00:00
2. Processing new version...
```

**Data:**

```json
{
  "deployment_method": "zero-downtime",
  "features": [
    { "name": "Failover", "description": "Switch traffic to a backup system if primary fails" },
    { "name": "Blue-Green Deployment", "description": "Deploy new code to a separate environment and swap traffic when ready" },
    { "name": "Canary Release", "description": "Deploy to a small subset of users and monitor before full rollout" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. User Request
6. Is Service Running?
7. Continue Using Old Code
8. New Version Deployed
9. Return Response
10. Deploy New Version?
11. Monitor Performance
12. Switch Traffic Gradually to NewCode
13. User ContinuesUninterruptedly

