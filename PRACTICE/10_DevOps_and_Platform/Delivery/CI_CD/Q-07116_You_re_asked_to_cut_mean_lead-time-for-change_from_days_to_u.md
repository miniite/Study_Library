---
type: "practice"
difficulty: "Expert"
---

# You're asked to cut mean lead-time-for-change from days to under an hour across the org; decompose where the time actually goes (review, CI, queueing, approvals, deploy) and prioritize interventions by leverage rather than by what's easiest.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ CI Performance Investment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/CI_Performance_Investment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To cut mean lead-time-for-change from days to under an hour across the organization, we first need a thorough understanding of where time is being spent in the CI/CD pipeline. This involves decomposing the current process into its components and measuring the time taken at each stage. The key areas are review, continuous integration (CI), queueing, approvals, and deployment. We then prioritize interventions based on leverage rather than ease, focusing on high-impact changes that can significantly reduce lead times.

TL;DR
Decompose the current pipeline to identify bottlenecks.
Prioritize by leverage, not just ease of implementation.
Implement targeted improvements where they will have the most significant impact.

#### Triage
The Question and What's Being Probed

The question is essentially asking us to optimize a CI/CD pipeline for faster delivery cycles. It’s probing our ability to:

Understand the current state of the CI/CD process.
Identify bottlenecks that are causing delays.
Prioritize interventions based on their potential impact.
Step-by-Step Solution: Decomposing and Prioritizing
1. Gather Data on Current Pipeline Performance
Collect metrics such as average lead time, cycle time (time from code commit to deployment), and wait times for each stage of the pipeline.
Identify specific stages where delays are most significant.
2. Identify Bottlenecks
Review: Time spent waiting for feedback or approvals after a pull request is submitted.
CI: Build and test time, which can be lengthy if there's a large codebase or complex build processes.
Queueing: Time spent in the CI queue when multiple changes are being processed concurrently.
Approvals: Manual approval steps that introduce delays.
Deployment: Time taken for automated deployments versus manual intervention.
3. Prioritize by Leverage
Focus on high-leverage areas first, such as reducing wait times in the review and CI stages.
For example:
Review: Reduce average time to initial feedback from 2 days to 1 hour.
CI: Optimize build and test processes to reduce pipeline execution time by 50%.
Queueing: Implement a smarter queue management system that prioritizes urgent changes.
Detailed Steps
Review
Current State: Assume average wait time for initial feedback is 2 days (48 hours).
Optimization Goal: Reduce this to 1 hour.
Introduce automated code quality checks and static analysis tools.
Implement a parallel review system where multiple reviewers can comment on the same PR simultaneously.
CI
Current State: Assume average build and test time is 6 hours.
Optimization Goal: Reduce this by 50% to 3 hours.
Optimize build scripts for parallel execution.
Implement caching mechanisms for pre-built dependencies.
Use faster testing frameworks that can run in parallel.
Queueing
Current State: Assume the average queue time is 24 hours due to high concurrent builds.
Optimization Goal: Reduce this to a maximum of 30 minutes.
Implement a smarter queuing system with prioritization rules based on urgency and criticality.
Use state-of-the-art container orchestration tools that can handle more concurrent builds efficiently.
Approvals
Current State: Assume manual approvals take up an average of 12 hours per PR.
Optimization Goal: Reduce this to a maximum of 1 hour.
Automate as many approval steps as possible using self-service portals and automated checks.
Implement a role-based access control system to streamline the approval process.
Deployment
Current State: Assume manual deployments take up an average of 4 hours per deployment.
Optimization Goal: Reduce this to 30 minutes or less.
Automate deployments using CI/CD tools like Jenkins, GitLab CI, or Kubernetes.
Implement blue-green deployment strategies that allow for zero-downtime deployments.

#### Closing Summary

By focusing on high-leverage areas first and implementing targeted improvements, we can significantly reduce the mean lead-time-for-change from days to under an hour. The key is not just identifying bottlenecks but also prioritizing interventions based on their potential impact.

#### Common Mistakes/Follow-Ups
Mistake: Focusing solely on ease of implementation rather than leverage.

#### Follow-Up
Explain the importance of focusing on high-leverage areas even if they require more effort initially.
Mistake: Ignoring data collection and analysis.

#### Follow-Up
Highlight the critical role of data in making informed decisions about where to focus optimization efforts.

**Comparison:**

```
Stage	Current Time	Target Time
Review	24 hours	1 hour
CI	6 hours	3 hours
Queueing	30 minutes	30 minutes (max)
Approvals	12 hours	1 hour
Deployment	4 hours	30 minutes
```

**Code:**

```python
# Example Code for Automated Build Scripts Optimization

def optimize_build_scripts(scripts):
    # Implement caching logic
    cache_hits = 0
    total_requests = len(scripts)

    for script in scripts:
        if is_cacheable(script):
            result = fetch_from_cache(script)
            cache_hits += 1
        else:
            result = execute_script(script)

        log_build_result(result)

def is_cacheable(script):
    # Logic to determine if a build can be cached
    return "dependency" in script

def fetch_from_cache(script):
    # Fetch from local or remote cache
    pass

def execute_script(script):
    # Execute the build script and record time taken
    start_time = time.time()
    result = run_command(script)
    end_time = time.time()
    total_time = end_time - start_time
    return {"status": "success", "time": total_time}
```

**Example input / output:**

```text
Input:
- Current lead time is 2.5 days.
- Cycle times for review, CI, queueing, and deployment are 24 hours, 6 hours, 30 minutes, and 12 hours respectively.

Output:
- Optimized lead time to under an hour by focusing on high-leverage areas like reducing review wait time to 1 hour, CI execution time by 50%, implementing smarter queuing for concurrent builds, automating approvals, and fully automating deployments.
```

**Data:**

```json
{
  "current_pipeline_data": {
    "lead_time": 2.5, // Days
    "cycle_time": [0.8, 1.2, 2.3, 4.0], // Hours for each stage: review, CI, queueing, deployment
    "wait_times": [24, 6, 30, 12] // Hours in each stage
  }
}
```

**Flow:**

1. Review
2. Automated Feedback
3. 1 Hour
4. CI
5. Optimize Build Scripts
6. 3 Hours
7. Queueing
8. Smarter Queuing
9. 30 Minutes (max)
10. Approvals
11. Automate Approvals
12. 1 Hour
13. Deployment
14. Auto Deployments
15. 30 Minutes

