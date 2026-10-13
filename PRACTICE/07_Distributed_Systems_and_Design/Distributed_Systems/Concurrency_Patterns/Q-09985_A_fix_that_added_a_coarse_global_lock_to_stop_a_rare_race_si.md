---
type: "practice"
difficulty: "Expert"
---

# A "fix" that added a coarse global lock to stop a rare race silently halved the service's throughput and no one connected the two for weeks. As principal, how do you build the review and observability guardrails so a correctness fix can't quietly become a scalability regression?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a Backend Architect/Principal, it's crucial to ensure that any architectural changes, especially those addressing concurrency issues like race conditions, are not only correct but also don’t inadvertently degrade performance. The recent incident with the coarse global lock is a stark reminder of how such changes can go unnoticed for weeks if proper review and observability practices aren't in place.

TL;DR:

To prevent correctness fixes from becoming scalability regressions, we need to implement robust review processes that include thorough testing, automated performance monitoring, and detailed logging. By doing so, any adverse effects on throughput or latency can be detected early.

#### Triaging the Question

The question is essentially asking how to build guardrails in a system that ensures that changes intended for correctness (like resolving race conditions) don't inadvertently introduce scalability issues. The key aspects being probed are:

How to design and enforce review processes.
What types of observability tools should be used.
How to set up automated testing frameworks.

#### Assumptions
Race Condition: A rare race condition was identified, leading to the addition of a coarse global lock to resolve it.
Throughput Impact: The introduction of the global lock reduced throughput by 50%, but no one noticed until weeks later.
Review Process: Current review processes are not sufficient to catch such issues.
System Complexity: The system is complex, with multiple microservices and threads.

#### First-Principles Solution

Refactor and Modularize the Code:

Step 1: Refactor the codebase around fine-grained locks or semaphores instead of a coarse global lock.
Refactoring intoFine-Grained Semaphores
Original Coarse Lock
Fine-Grained Semaphores
Increased Granularity
Improved Scalability
Step 2: Use semaphores or other concurrency primitives that can be scoped to specific operations or services.

Implement Automated Performance Testing:

Step 1: Set up a continuous integration/continuous deployment (CI/CD) pipeline with automated performance tests.
Step 2: Integrate load testing tools like JMeter, LoadRunner, or Gatling to run stress tests on the code changes.
BASH
# Example CI Pipeline Configuration
jobs:
  - job: PerformanceTesting
    steps:
      - script:
          - apt-get update && apt-get install -y jmeter
      - script:
          - java -jar /path/to/jmeter.jar -n -t /path/to/test-plan.jmx -l /path/to/results.jtl

Enhance Observability:

Step 1: Implement distributed tracing using tools like Jaeger or Zipkin.
Step 2: Set up centralized logging with Elasticsearch, Logstash, and Kibana (ELK stack) to monitor logs in real-time.
YAML
# ELK Stack Configuration
output.elasticsearch:
  hosts: ["localhost:9200"]

Review and Code Ownership:

Step 1: Establish a code ownership model where each microservice or module is owned by specific engineers who are responsible for reviewing changes.
Step 2: Implement peer review sessions to ensure that every change goes through at least two sets of eyes.

Performance Metrics and Alarms:

Step 1: Define key performance indicators (KPIs) such as response time, throughput, and error rates.
Step 2: Set up threshold alarms for these metrics to trigger alerts when they cross predefined limits.
YAML
# Example Alert Configuration
thresholds:
  - name: ResponseTimeAlert
    value: 500ms
    severity: WARNING
    duration: 1m

Post-Mortem and Lessons Learned:

Conduct thorough post-mortems after issues like the one with the global lock.
Document the lessons learned in a shared knowledge base to prevent similar incidents.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By implementing these guardrails, we can ensure that future correctness fixes do not inadvertently lead to scalability regressions. The key is to have a robust review process, thorough testing, and detailed observability tools. Common mistakes include:

Overlooking the need for fine-grained concurrency controls.
Not integrating performance testing into CI/CD pipelines.
Insufficient logging or tracing mechanisms.

Common follow-ups might involve discussing specific tools and technologies used in each step, potential trade-offs between different solutions, and how to balance developer productivity with comprehensive testing and monitoring.

**Comparison:**

```
2023-11-15
2023-11-17
2023-11-19
2023-11-21
2023-11-23
2023-11-25
2023-11-27
2023-11-29
2023-12-01
2023-12-03
2023-12-05
ResponseTimeAlert
ThroughputThreshold
Alerts
Thresholds
Performance Metrics and Alarms Configuration
```

**Code:**

```bash
# Example CI Pipeline Configuration
jobs:
  - job: PerformanceTesting
    steps:
      - script:
          - apt-get update && apt-get install -y jmeter
      - script:
          - java -jar /path/to/jmeter.jar -n -t /path/to/test-plan.jmx -l /path/to/results.jtl
```

**Example input / output:**

| Test Case | Input | Expected Output |
|-----------|-------|----------------|
| LoadTest1 | 1000 requests | ResponseTime: <500ms, Throughput: >1000 req/s |
| LoadTest2 | 5000 requests | ResponseTime: <1000ms, Throughput: >5000 req/s |

**Data:**

```yaml
# ELK Stack Configuration
output.elasticsearch:
  hosts: ["localhost:9200"]
```

**Flow:**

1. Code Change
2. Peer Review
3. Automated Testing
4. Performance Monitoring
5. Alerts & Notifications
6. Post-Mortem Analysis

