---
type: "practice"
difficulty: "Deep"
---

# You maintain a shared alert-correlation-rules library consumed by eight service teams via a Git ref, and a breaking correlation-group change reached all of them because consumers tracked main. Design a versioning and pinning scheme so a rules change can't silently fan out to every team's paging.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge effectively, we need to ensure that changes in the alert-correlation-rules library are communicated clearly and controlled among the eight service teams. The core issue is avoiding a scenario where a breaking change can silently propagate to all consumers tracked on main. We'll design a versioning and pinning scheme that ensures transparency and control over updates.

#### Triaging the Question

The question is essentially about designing a robust versioning strategy for a shared library that multiple teams depend on. The key aspects include:

Version Control: Ensure that changes are properly tagged with versions.
Pinning Mechanism: Allow consumers to specify which version they are currently tracking and update only when necessary.
Communication Protocol: Implement a process for announcing breaking changes.

#### Assumptions
Each service team is using the library from main.
The library uses semantic versioning (SemVer).
A breaking change is defined as any change that could affect the behavior of existing rules without requiring additional context or configuration.

#### First-Principles Solution
Version Incrementing: Use SemVer to increment minor versions for new features and major versions for breaking changes.
Pinning Mechanism: Implement a pinning mechanism where consumers can specify their current version, ensuring they only update when necessary.
Communication Protocol: Establish a communication protocol for announcing breaking changes.
Step 1: Version Incrementing

To use SemVer effectively:

Major versions: Indicate incompatible API changes (breaking changes).
Minor versions: Indicate backwards-compatible feature additions or bug fixes.
Patch versions: Indicate only backwards-compatible bug fixes.

For example, if the current version is 3.4.2, a breaking change might increment to 4.0.0.

Step 2: Pinning Mechanism

Each service team will maintain a file (e.g., .gitmodules or a custom configuration) that specifies their pinned version of the library:

PLAINTEXT
[repository "https://github.com/org/alert-correlation-rules.git"]
    branch = main
    tag = v4.0.0  # This is pinned to v4.0.0

This ensures that teams only pull updates explicitly allowed by their pinning configuration.

Step 3: Communication Protocol
Announce Breaking Changes: When a breaking change occurs, the maintainers of the library should create an announcement in a common communication channel (e.g., Slack).
Notify Consumers: The announcement should include details like the new version number and any necessary documentation or migration steps.
Update Configuration: Service teams will update their pinning configuration to the latest compatible version after reviewing the documentation.

#### Implementation
Example

Suppose we have an alert-correlation-rules library at v3.4.2 and a breaking change is introduced in v4.0.0. The steps would be:

Announce the breaking change on Slack.
Update the .gitmodules file of each service team to include:
PLAINTEXT
[repository "https://github.com/org/alert-correlation-rules.git"]
    branch = main
    tag = v4.0.0  # Pinned to v4.0.0
Service teams will pull the changes and ensure they are compatible with v4.0.0.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Not properly tagging versions or not updating the pinning mechanism can lead to silent propagation of breaking changes.

#### Follow-Up
Consider implementing a more robust automation process for pinning updates, such as using Git hooks or CI/CD pipelines.

This approach ensures that all service teams are aware of and can control their versioning, thereby reducing the risk of silent, disruptive changes.

**Code:**

```plaintext
[repository "https://github.com/org/alert-correlation-rules.git"]
    branch = main
    tag = v4.0.0  # Pinned to v4.0.0
```

**Flow:**

1. Announce Breaking Change
2. Publish SemVer Tag v4.0.0
3. Service Teams Update.gitmodules
4. Pull Changes and VerifyCompatibility
5. Notify Maintenance

