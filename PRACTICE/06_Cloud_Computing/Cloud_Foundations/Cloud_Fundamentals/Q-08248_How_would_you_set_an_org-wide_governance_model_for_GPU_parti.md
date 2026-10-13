---
type: "practice"
difficulty: "Deep"
---

# How would you set an org-wide governance model for GPU partitioning (MIG, time-slicing, fractional GPUs) so small inference workloads share silicon efficiently without one tenant's kernel starving another's SLO?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ GPU Quota Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/GPU_Sharing/GPU_Quota_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of setting an org-wide governance model for GPU partitioning (MIG, time-slicing, fractional GPUs) in a way that small inference workloads can share the silicon efficiently without one tenant's kernel starving another’s service level objective (SLO), we need to design a robust and scalable solution. This involves understanding the underlying principles of MIG, defining appropriate policies for GPU allocation, and implementing effective monitoring and enforcement mechanisms.

The key idea is to leverage MIG technology along with time-slicing and fractional GPUs to create a dynamic and fair resource-sharing environment. By doing so, we ensure that smaller workloads can run efficiently while maintaining high service levels for critical tasks.

**Comparison:**

```
MIG Settings	Value
Enable MIG	True
Mig Levels	[1, 2]
Partition Size	4GB
Time-Slicing Policies	Value
Max Time Slice Duration	60 seconds
Min Reserved Time	15 seconds
```

**Code:**

```python
# Example Python script to configure GPU partitioning policies
def configure_gpu_partitioning():
    # Define MIG configuration settings
    mig_settings = {
        "enable_mig": True,
        "mig_levels": [1, 2],  # Enable multiple levels of MIG
        "partition_size": "4GB"  # Allocate each partition with 4GB of memory
    }

    # Define time-slicing policies
    time_slicing_policies = {
        "max_time_slice_duration": 60,  # Time slice duration in seconds
        "min_reserved_time": 15         # Minimum reserved time for critical tasks
    }

    return mig_settings, time_slicing_policies
```

**Example input / output:**

```text
Input:
- Define a policy for MIG with two levels and partitions of 4GB.
- Set the maximum time slice duration to 60 seconds.
- Reserve at least 15 seconds as minimum critical task time.

Output:
- GPU configurations are updated to enable MIG with appropriate partition sizes.
- Time-slicing policies are set up to ensure fair resource allocation and SLO compliance.
```

**Data:**

```json
{
  "mig_config": {
    "enable_mig": true,
    "mig_levels": [1, 2],
    "partition_size": "4GB"
  },
  "time_slice_policy": {
    "max_time_slice_duration": 60,
    "min_reserved_time": 15
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Define MIG Settings
6. Enable MIG?
7. MIG Levels & Partition Sizes
8. No MIG Configuration
9. Set Time-Slicing Policies
10. Critical Task Reserved Time?
11. Update Minimum Critical TaskTime
12. Set Maximum Time SliceDuration
13. GPU Configurations Updated
14. Time Slicing Policies Set Up
15. Governance Model Established
16. Governance Model Skipped
17. Monitor and Enforce SLOCompliance
18. Monitor and Enforce FairResource Allocation
19. Ensure Efficient WorkloadSharing
20. Skip Configuration forNon-MIG Users

