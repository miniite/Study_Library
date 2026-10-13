---
type: "practice"
difficulty: "Expert"
---

# How would you design deploy verification for an edge/CDN-compute platform (Workers/Lambda@Edge running in 300 PoPs) where propagation is eventually consistent, you can't drain a PoP like a pod, and 'deploy done' must mean 'every PoP serving the new version correctly'?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design deploy verification for an edge/CDN-compute platform with eventual consistency, we need to ensure that every point of presence (PoP) is serving the new version of our code or configuration correctly before declaring a deployment as "done." Given the constraints—where PoPs cannot be drained like pods and propagation is eventually consistent—we must adopt a strategy that continuously checks each PoP for correctness until all have been verified. This involves a combination of monitoring, validation techniques, and potentially rolling out deployments in stages.

The key idea here is to implement a mechanism that can repeatedly check the health and state of each PoP until we are confident that every instance is serving the updated code or configuration correctly. We'll use monitoring tools and possibly custom scripts to automate this process. The solution involves setting up verification steps, potentially using a rolling deployment strategy, and continuously polling or proactively checking each PoP.

**Comparison:**

```
Pop ID	Status
1	Verified
2	Failed
3	Verified
```

**Code:**

```python
import requests

def verify_pop(pop_id):
    # Example URL for fetching the version information from the PoP
    response = requests.get(f"https://api.example.com/pops/{pop_id}/version")
    if response.status_code == 200:
        return response.json()['version']
    else:
        raise Exception(f"Failed to retrieve version info for POP {pop_id}")
```

**Example input / output:**

```

To implement the verification, we can use a simple script that iterates over each PoP and checks its version:

PYTHON
Copy
pop_ids = [1, 2, 3]  # Example list of PoP IDs

verified_pops = []

for pop_id in pop_ids:
    try:
        current_version = verify_pop(pop_id)
        print(f"POP {pop_id} is serving version: {current_version}")
        verified_pops.append(pop_id)
    except Exception as e:
        print(f"Failed to verify POP {pop_id}: {str(e)}")

print("Verification complete.")
```

**Data:**

```yaml
pops:
  - id: 1
    name: PoP-A
  - id: 2
    name: PoP-B
  - id: 3
    name: PoP-C
```

**Flow:**

1. Yes
2. No
3. Start
4. Is all POP verified?
5. Declare deployment done
6. Wait and retry verification forunverified POPs

