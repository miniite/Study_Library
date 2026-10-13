---
type: "practice"
difficulty: "Intermediate"
---

# A flaky E2E test at 8% failure rate blocks merges and engineers just re-run until green; how would you quarantine it, keep the signal, and stop the re-run habit from masking a real intermittent bug?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Flaky Tests](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Flaky_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CI/CD pipelines are critical to ensuring software quality and reliability. However, flaky tests can be a significant pain point, particularly when they occur frequently but not consistently enough to trigger automatic retries or failures. In this scenario, we need to address the issue of an E2E test that fails 8% of the time by quarantining it while keeping its signal intact and discouraging engineers from manually re-running tests just because they get red initially.

TL;DR: We will introduce a quarantine mechanism for flaky tests using a combination of tags, retries with exponential backoff, and notifications. This approach ensures that the test remains part of the CI/CD pipeline but prevents it from causing unnecessary delays or masking real issues.

#### Triaging the Question

The core question revolves around handling flaky tests in a CI/CD environment. Specifically:

The 8% failure rate is too frequent to ignore but not high enough for automatic failures.
Engineers re-run tests until they pass, which can mask underlying bugs.
We need a balanced approach to quarantine the test, maintain its signal, and encourage proper handling.

#### Assumptions
The CI/CD pipeline uses a tagging system for tests (e.g., @flaky).
There is an existing notification mechanism in place.
Engineers are familiar with basic CI/CD concepts and can follow instructions to rerun or skip flaky tests.

#### Solution
Step 1: Tagging the Flaky Test

First, we need a way to identify and tag this test. We'll introduce a custom CI/CD pipeline step that automatically tags any E2E test with an 8% failure rate as @flaky.

BASH
# Example Jenkins Pipeline snippet
pipeline {
    agent any
    stages {
        stage('Run E2E Tests') {
            steps {
                script {
                    def results = sh(returnStdout: true, script: 'mvn verify -Dtest=E2ETests').trim()
                    def failures = results.count('TEST-FAIL')

                    if (failures >= 0.8 * totalTests) {
                        echo "Marking test as flaky"
                        env.FLAKY_TEST_TAG = true
                    }
                }
            }
        }
    }
}
Step 2: Configure Retries with Exponential Backoff

Next, we'll configure retries for the tagged tests. Instead of failing immediately, these tests will be retried multiple times with increasing delays between attempts.

YAML
# Example Jenkins configuration snippet (freestyle job)
post {
    success {
        retry(3) { // Retry up to 3 times
            timeout(time: 10, unit: 'MINUTES') { // Wait for 10 minutes before next retry
                build job: 'E2E.Tests', parameters: [[$class: 'StringParameterValue', name: 'TAGGED_FLAKY_TEST']]
            }
        }
    }
}
Step 3: Introduce Notifications and Documentation

To keep everyone informed about the status of flaky tests, we'll set up notifications for every attempt. Additionally, a detailed documentation guide will be provided to engineers on how to handle these tests.

YAML
# Example Slack notification configuration
post {
    success {
        echo 'Test passed after retries'
        notify parameter: 'SUCCESS', message: "E2E Test Passed After Retries"
    }

    failure {
        echo 'Test failed, retrying...'
        notify parameter: 'FAILURE', message: "E2E Test Failed, Retrying in 10 minutes"
    }
}

#### Closing Summary

By implementing a combination of tagging, retries with exponential backoff, and notifications, we can effectively manage flaky tests without compromising their integrity or masking real bugs. This approach ensures that the signal from these tests is preserved while discouraging unnecessary re-runs.

#### Common Mistakes

Ignoring the tagging step entirely.
Not setting up appropriate retry mechanisms or notification systems.
Failing to document and communicate the new process to all relevant stakeholders.
Code Blocks

**Code:**

```yaml
# Example Jenkins Pipeline snippet for tagging flaky tests
pipeline {
    agent any
    stages {
        stage('Run E2E Tests') {
            steps {
                script {
                    def results = sh(returnStdout: true, script: 'mvn verify -Dtest=E2ETests').trim()
                    def failures = results.count('TEST-FAIL')

                    if (failures >= 0.8 * totalTests) {
                        echo "Marking test as flaky"
                        env.FLAKY_TEST_TAG = true
                    }
                }
            }
        }
    }
}

# Example Jenkins freestyle job configuration for retries with exponential backoff
post {
    success {
        retry(3) { // Retry up to 3 times
            timeout(time: 10, unit: 'MINUTES') { // Wait for 10 minutes before next retry
                build job: 'E2E.Tests', parameters: [[$class: 'StringParameterValue', name: 'TAGGED_FLAKY_TEST']]
            }
        }
    }
}
```

**Data:**

```yaml
# Example Jenkins Pipeline snippet for tagging flaky tests
pipeline {
    agent any
    stages {
        stage('Run E2E Tests') {
            steps {
                script {
                    def results = sh(returnStdout: true, script: 'mvn verify -Dtest=E2ETests').trim()
                    def failures = results.count('TEST-FAIL')

                    if (failures >= 0.8 * totalTests) {
                        echo "Marking test as flaky"
                        env.FLAKY_TEST_TAG = true
                    }
                }
            }
        }
    }
}

# Example Jenkins freestyle job configuration for retries with exponential backoff
post {
    success {
        retry(3) { // Retry up to 3 times
            timeout(time: 10, unit: 'MINUTES') { // Wait for 10 minutes before next retry
                build job: 'E2E.Tests', parameters: [[$class: 'StringParameterValue', name: 'TAGGED_FLAKY_TEST']]
            }
        }
    }
}
```

