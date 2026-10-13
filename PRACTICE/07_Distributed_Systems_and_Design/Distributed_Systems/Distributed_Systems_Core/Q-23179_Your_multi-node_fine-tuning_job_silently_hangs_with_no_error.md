---
type: "practice"
difficulty: "Deep"
---

# Your multi-node fine-tuning job silently hangs with no error using NCCL. Walk me through isolating a mismatched collective, a single crashed rank the others wait on, or a deadlocked all-reduce, and the timeouts and watchdogs you'd add to fail fast.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gossip and Deduplication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why your multi-node fine-tuning job using NCCL (NVIDIA Collective Communications Library) is hanging without any error messages, we need to carefully isolate potential issues such as a mismatched collective, a single crashed rank that the others are waiting on, or a deadlock in an all-reduce operation. We will reason through this from first principles and then propose robust watchdogs and timeouts.

Triaging the Problem

The primary issue here is identifying whether the hang is due to communication failures between ranks or within a specific collective operation. NCCL's silent failure can be particularly tricky because it doesn't always report detailed errors, making debugging challenging. Here are some common scenarios:

Mismatched Collective: A mismatch in collective operations could arise if the number of processes participating in an operation is not correctly set.
Crashed Rank Blocking Others: If a single rank has crashed or is stuck, the other ranks may wait indefinitely for it to complete.
Deadlock in All-Reduce: An all-reduce operation might deadlock due to resource contention or incorrect configurations.
First Principles Approach
Check Collective Operations:
Verify that the number of processes participating in each collective matches the expected count.
Monitor Process Statuses:
Use tools like nvidia-smi and ncu (NVIDIA CUDA Profiler) to monitor process statuses and identify any ranks that are not active or have crashed.
Examine Communication Patterns:
Analyze the communication patterns using detailed profiling tools such as NCCL's profiling utilities.
Step-by-Step Solution

Collective Operations Mismatch:

Ensure NCCL_NOXFAIL is enabled in your environment, which allows NCCL to report errors during collective operations.
Use ncclCommCreateGroup() and ncclCommInitRank() functions correctly to initialize groups with the correct number of ranks.

Crashed Rank Detection:

Implement a watchdog function that periodically checks the status of each process.
Use signal handling mechanisms (e.g., SIGALRM) to trigger a graceful shutdown if a rank fails or hangs indefinitely.

Deadlock Detection in All-Reduce:

Add timeouts for collective operations and ensure that an operation does not block indefinitely.
Implement a timeout mechanism using ncclCommStartTimeout() and ncclCommWaitTimeout() functions to detect deadlocks.
Watchdogs and Timeouts

Process Watchdog:

C
void watchdog() {
    pid_t pid = getpid();
    alarm(30);  // Set an initial timeout of 30 seconds
    while (true) {
        if (kill(pid, 0) == -1) {  // Check process status using SIGKILL
            printf("Rank %d has crashed\n", pid);
            exit(EXIT_FAILURE);
        }
        sleep(1);  // Wait for one second before checking again
    }
}

Timeouts in NCCL:

C
ncclResult_t result = ncclCommStartTimeout(comm, &timeoutHandle, timeoutMs);
if (result != ncclSuccess) {
    printf("NCCL operation timed out after %d ms\n", timeoutMs);
    exit(EXIT_FAILURE);
}

#### Closing Summary
and Common Mistakes

By implementing these watchdogs and timeouts, we can effectively identify and address the root causes of hangs in multi-node fine-tuning jobs. Common mistakes include not enabling NCCL_NOXFAIL, missing process status checks, or insufficient timeout mechanisms.

Common Follow-Ups
How would you integrate these changes into an existing deployment?
What other tools and diagnostics could be useful for this scenario?
How do these solutions scale with increasing numbers of nodes?

**Code:**

```
C
Copy
#include <nccl.h>
#include <signal.h>

void watchdog() {
    pid_t pid = getpid();
    alarm(30);  // Set an initial timeout of 30 seconds
    while (true) {
        if (kill(pid, 0) == -1) {  // Check process status using SIGKILL
            printf("Rank %d has crashed\n", pid);
            exit(EXIT_FAILURE);
        }
        sleep(1);  // Wait for one second before checking again
    }
}

ncclResult_t checkCollectiveOperations(int worldSize, int rank) {
    ncclComm_t comm;
    if (ncclCommInitRank(&comm, worldSize, ncclGetGroupList(worldSize), rank) != ncclSuccess) {
        return ncclFailed;
    }

    // Example: Start an all-reduce operation with a timeout
    float data[1024];
    ncclResult_t result = ncclCommStartTimeout(comm, &timeoutHandle, 5000);
    if (result != ncclSuccess) {
        printf("NCCL operation timed out after 5000 ms\n");
        exit(EXIT_FAILURE);
    }

    // Complete the timeout check
    ncclCommWaitTimeout(comm, timeoutHandle);

    return result;
}
```

**Example input / output:**

```text
Input:
- World size = 4
- Rank = 1

Output:
- No error messages initially
- Watchdog detects and logs "Rank 2 has crashed"
```

**Flow:**

1. Initialize NCCL
2. Check collective operations
3. Watchdog process status
4. Timeout for all-reduceoperation
5. Graceful shutdown or errorreporting

