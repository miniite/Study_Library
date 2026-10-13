---
type: "practice"
difficulty: "Deep"
---

# A Kubernetes rolling update of a leader-elected controller briefly ran two active leaders and double-processed jobs. Explain how the lease-based leader election in client-go can allow this during a terminationGracePeriod and how you'd fix it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, particularly within Kubernetes, managing the state of multiple instances of a service is critical to ensure consistency and reliability. Leader election mechanisms like lease-based leader election in client-go are designed to handle such scenarios by ensuring only one instance acts as the leader at any time. However, during certain operations, such as rolling updates with terminationGracePeriods, there can be brief periods where both old and new leaders might exist, potentially leading to double-processing of jobs.

#### Triaging the Question

This question focuses on understanding how lease-based leader election works in client-go and identifying why a Kubernetes rolling update could temporarily run two active leaders. The core issue is related to timing and resource handling during transitions between old and new instances. We need to ensure that only one leader exists at any time, even during periods of transition.

#### First-Principles Solution
Understanding Lease-Based Leader Election

Lease-based leader election in client-go works by each instance (referred to as a "leader" or "candidate") requesting a lease from the Kubernetes API server. The candidate that holds the longest lease is considered the leader. This mechanism ensures that only one leader exists at any time, but it relies on correct handling of leases and timely termination.

Rolling Update with TerminationGracePeriod

During a rolling update in Kubernetes, old instances are terminated while new ones are created. The terminationGracePeriod specifies how long an instance has to gracefully shut down before being forcibly killed. If the transition between old and new leaders is not handled correctly, both might hold leases at the same time.

Common Issues During Rolling Updates
Timing of Lease Release: When an old leader detects that it needs to terminate, it should release its lease immediately. However, due to network latency or delays in processing termination requests, the lease may not be released promptly.
Grace Period Handling: If a new instance starts and attempts to acquire a lease before the old one releases its lease, both might hold leases for a brief period.
Fixing the Issue

To prevent double-processing of jobs during a rolling update, we need to ensure that:

Timely Lease Release: Old instances must release their leases as soon as possible when they detect termination.
Lease Renewal Management: New instances should only attempt to acquire a lease after the old instance has had sufficient time to terminate and release its lease.

#### Solution
Details

To address this issue, we can implement a custom leader election mechanism that ensures timely lease releases and proper handling of lease renewals:

GO
func (c *Controller) Run(stopCh <-chan struct{}) {
    // Start the controller loop
    go func() {
        for {
            log.Info("Starting new controller instance")
            c.runOnce()

            select {
            case <-stopCh:
                return
            case <-time.After(time.Second):
                continue
            }
        }
    }()

    // Wait for graceful shutdown
    <-stopCh
}

func (c *Controller) runOnce() {
    if err := c.LeaderElectionLeader(); err != nil {
        log.Error(err, "Failed to acquire lease")
        return
    }

    defer func() { _ = c.LeaderElectionRelease() }()

    // Process jobs here
    for job := range c.jobQueue {
        // Handle job processing
    }
}

func (c *Controller) LeaderElectionLeader() error {
    log.Info("Attempting to acquire lease")
    if err := clientv3.NewLeaseClient(c.kubeClient).AcquireLease(leaseName); err != nil {
        return err
    }

    // Start the controller loop after acquiring lease
    c.Run(stopCh)
    return nil
}

func (c *Controller) LeaderElectionRelease() error {
    log.Info("Releasing lease")
    if err := clientv3.NewLeaseClient(c.kubeClient).ReleaseLease(leaseName); err != nil {
        return err
    }

    // Handle graceful shutdown and release resources
    return nil
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

By ensuring timely lease releases and proper handling of lease renewals, we can prevent double-processing during rolling updates. Key takeaways include:

Proper Lease Management: Ensure old instances release leases promptly.
Graceful Shutdown Handling: Implement robust shutdown logic to avoid race conditions.

#### Common mistakes
include delayed lease releases due to network latency or mismanaged termination signals, leading to periods where both old and new leaders hold leases simultaneously.

**Code:**

```go
func (c *Controller) Run(stopCh <-chan struct{}) {
    // Start the controller loop
    go func() {
        for {
            log.Info("Starting new controller instance")
            c.runOnce()

            select {
            case <-stopCh:
                return
            case <-time.After(time.Second):
                continue
            }
        }
    }()

    // Wait for graceful shutdown
    <-stopCh
}

func (c *Controller) runOnce() {
    if err := c.LeaderElectionLeader(); err != nil {
        log.Error(err, "Failed to acquire lease")
        return
    }

    defer func() { _ = c.LeaderElectionRelease() }()

    // Process jobs here
    for job := range c.jobQueue {
        // Handle job processing
    }
}

func (c *Controller) LeaderElectionLeader() error {
    log.Info("Attempting to acquire lease")
    if err := clientv3.NewLeaseClient(c.kubeClient).AcquireLease(leaseName); err != nil {
        return err
    }

    // Start the controller loop after acquiring lease
    c.Run(stopCh)
    return nil
}

func (c *Controller) LeaderElectionRelease() error {
    log.Info("Releasing lease")
    if err := clientv3.NewLeaseClient(c.kubeClient).ReleaseLease(leaseName); err != nil {
        return err
    }

    // Handle graceful shutdown and release resources
    return nil
}
```

