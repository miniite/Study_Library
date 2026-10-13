---
type: "practice"
difficulty: "Deep"
---

# A deploy job hangs and never times out. How do you diagnose and what timeout layers exist between job runner and project?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To diagnose a hanging GitLab CI job, we need to understand where the job execution gets stuck—whether it's within the job script itself or in the interaction with GitLab’s infrastructure. The key layers to check include the job configuration file, the runner environment, and any network or server-side timeouts.

#### Triage
Understanding the Problem

A hanging GitLab CI job that never times out can indicate a variety of issues, including:

A stuck script within the before_script, script, or after_script sections.
Network latency or connectivity problems between the runner and GitLab’s servers.
Issues with the runner environment, such as resource constraints or misconfiguration.
Step-by-Step Solution: From First Principles

Job Configuration Review

Check the Job Script: Inspect the before_script, script, and after_script sections for any long-running commands that might cause the job to hang.
YAML
stages:
  - build

build_job:
  stage: build
  script:
    - echo "Starting build..."
    - make all  # This might be running indefinitely or failing silently
  timeout: 3600  # Timeout set for one hour
Add Debugging Statements: Add echo statements to track the execution flow and identify where it gets stuck.
BASH
build_job:
  script:
    - echo "Starting make all"
    - make all 2>&1 | tee make.log
    - cat make.log

Runner Environment Verification

Resource Constraints: Ensure the runner environment has sufficient resources (CPU, memory) to execute the job.
Network Latency: Check for network issues that might be causing delays in job execution or communication with GitLab’s servers.
BASH
# Example: Check network latency between the runner and GitLab API server
ping gitlab.example.com

Server-Side Timeouts

Runner Configuration: Review the runner configuration to ensure it is set up correctly for the job execution.
Timeout settings in gitlab-runner config.toml:
TOML
[[runners]]
    name = "my_runner"
    url = "https://gitlab.example.com"
    token = "your_token"
    executor = "docker"
    [runners.docker]
        tls_verify = false
        image = "alpine:latest"
        privileged = true
    [runners.cache]
        [runners.cache.s3]
            # Cache settings
GitLab Project-Level Timeouts: Ensure that the project-level timeouts are configured correctly.
YAML
.gitlab-ci.yml:
jobs:
  timeout: 1800  # Default timeout for all jobs in this project

Debugging with Logs

Job Runner Logs: Check the logs of the job runner to identify any issues.
BASH
docker ps -a
docker logs <runner_container_id>
GitLab CI Pipeline Logs: Use GitLab’s pipeline dashboard to view detailed logs and identify bottlenecks.
Navigate to Pipelines -> Running -> Select the job -> View Logs.

Network Tracing

Traceroute or TCPDUMP: Use tools like traceroute or tcpdump to trace network traffic between the runner and GitLab’s servers.
BASH
traceroute gitlab.example.com
tcpdump -i eth0 host 192.168.1.100 and port 443

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically reviewing the job configuration, runner environment, and network conditions, we can identify the root cause of a hanging GitLab CI job. Common mistakes include not setting appropriate timeouts or misconfiguring the runner environment.

Mistake 1: Ignoring the script section for debugging.
Mistake 2: Overlooking network issues that could be causing delays.

#### Follow-Up
Question: How would you handle situations where the job hangs due to a long-running command without proper error handling?

**Comparison:**

| Layer | Details |
|-------|---------|
| Job Script | `before_script`, `script`, `after_script` sections for long-running commands, debugging statements |
| Runner Configuration | Resource constraints, network latency, runner environment misconfiguration |
| Server-Side Timeouts | Project-level timeouts in `.gitlab-ci.yml` and `gitlab-runner config.toml` |

**Code:**

```

stages:

build

build_job: stage: build script: - echo "Starting make all" - make all 2>&1 | tee make.log - cat make.log timeout: 3600 # Timeout set for one hour

CODE
Copy

```toml
[[runners]]
name = "my_runner"
url = "https://gitlab.example.com"
token = "your_token"
executor = "docker"
[runners.docker]
tls_verify = false
image = "alpine:latest"
privileged = true
[runners.cache]
```

**Example input / output:**

```text
# Example traceroute command output
traceroute gitlab.example.com
traceroute to gitlab.example.com (93.184.216.34), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  0.5 ms  0.6 ms  0.7 ms
 2  10.0.0.1 (10.0.0.1)  3.4 ms  3.3 ms  3.2 ms
 3  93.184.216.34 (93.184.216.34)  5.0 ms  5.1 ms  5.2 ms

# Example tcpdump command output
tcpdump -i eth0 host 192.168.1.100 and port 443
```

**Data:**

```yaml
# Example .gitlab-ci.yml snippet with timeout configuration
jobs:
  timeout: 1800  # Default timeout for all jobs in this project
```

**Flow:**

1. Job Script
2. Review script for long-runningcommands
3. Debugging statements added
4. Runner Configuration
5. Resource constraints checked
6. Network latency checked
7. Server-Side Timeouts
8. Project-level timeoutsreviewed
9. Job Runner Logs
10. GitLab CI Pipeline Logs

