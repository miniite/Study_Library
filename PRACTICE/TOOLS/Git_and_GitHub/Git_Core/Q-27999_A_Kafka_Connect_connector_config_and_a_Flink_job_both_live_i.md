---
type: "practice"
difficulty: "Deep"
---

# A Kafka Connect connector config and a Flink job both live in Git, and a merged PR changed a partition key in one but not the other, causing serving skew. How would you couple the two configs in one atomic commit and add a train/serve-style parity check in CI?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Kafka Connect connector configurations and Flink jobs are consistently updated together to avoid serving skew, we need to couple their configurations within an atomic commit and implement a parity check during the Continuous Integration (CI) pipeline. This involves ensuring both configurations are managed in Git, and leveraging CI tools to enforce consistency.

#### Triage

The question requires us to manage and synchronize two separate but interdependent configurations: a Kafka Connect connector configuration and a Flink job. The goal is to ensure that any changes made to the Kafka Connect connector's partition key are reflected in the corresponding Flink job within a single commit, thereby avoiding serving skew. Additionally, we need to implement a parity check during CI to verify the consistency of these configurations.

#### First-Principles Solution
Step 1: Repository Structure

We will maintain both configurations in a Git repository and use a consistent naming convention for easier reference and management. For instance:

CODE
├── kafka-connect-connector
│   └── config.properties
└── flink-job
    └── job-config.json
Step 2: Atomic Commit Mechanism

To ensure atomicity, we will create a script or use Git hooks that can automatically apply changes to both configurations simultaneously. For example, using a Git pre-commit hook:

BASH
#!/bin/bash

# Define the path to the Kafka Connect config and Flink job config files
KAFKA_CONNECT_CONF="kafka-connect-connector/config.properties"
FLINK_JOB_CONF="flink-job/job-config.json"

# Check if changes are made in both configurations
if git diff --cached "$KAFKA_CONNECT_CONF" && git diff --cached "$FLINK_JOB_CONF"; then
    echo "Both configurations changed, proceed with commit."
else
    echo "Error: Both configurations must be modified together for consistency." >&2
    exit 1
fi

# Apply the changes (this is just an example)
git add $KAFKA_CONNECT_CONF
git add $FLINK_JOB_CONF
Step 3: Parity Check in CI

We will implement a parity check as part of our CI pipeline to ensure that the Kafka Connect connector and Flink job configurations remain consistent. This can be done using a custom script or a dedicated tool that reads both files and verifies their consistency.

Here's an example of how this might look in a Jenkinsfile:

GROOVY
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Build step here...
            }
        }

        stage('Check Config Consistency') {
            steps {
                sh 'python3 parity_check.py'
            }
        }

        stage('Test') {
            steps {
                // Test step here...
            }
        }
    }
}

#### Closing Summary
and Common Mistakes

In summary, the key is to ensure that any changes to the Kafka Connect connector configuration are reflected in the Flink job configuration within a single commit. This can be achieved through Git hooks or scripts that enforce atomicity. Additionally, implementing parity checks during CI helps maintain consistency across both configurations.

#### Common mistakes
include:

Not using Git hooks or scripts to enforce atomic commits.
Failing to implement or properly configure CI tools to perform the necessary checks.
Mismanaging configuration files such that changes are applied inconsistently between Kafka Connect and Flink.

**Code:**

```

#!/bin/bash

Define the path to the Kafka Connect config and Flink job config files

KAFKA_CONNECT_CONF="kafka-connect-connector/config.properties" FLINK_JOB_CONF="flink-job/job-config.json"

Check if changes are made in both configurations

if git diff --cached "$KAFKA_CONNECT_CONF" && git diff --cached "$FLINK_JOB_CONF"; then echo "Both configurations changed, proceed with commit." else echo "Error: Both configurations must be modified together for consistency." >&2 exit 1 fi

Apply the changes (this is just an example)

git add $KAFKA_CONNECT_CONF git add $FLINK_JOB_CONF

CODE
Copy

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Build step here...
            }
        }

        stage('Check Config Consistency') {
            steps {
                sh 'python3 parity_check.py'
            }
        }

        stage('Test') {
            steps {
                // Test step here...
            }
        }
    }
}
```

