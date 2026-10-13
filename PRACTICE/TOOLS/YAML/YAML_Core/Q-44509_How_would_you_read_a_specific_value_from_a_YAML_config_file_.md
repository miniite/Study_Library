---
type: "practice"
difficulty: "Intermediate"
---

# How would you read a specific value from a YAML config file inside a shell script during a pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Linting YAML](<../../../../TOOLS/YAML/YAML_Core/Linting_YAML.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To read a specific value from a YAML configuration file within a shell script, we typically use tools like yq or yaml-cat. This is common in CI/CD pipelines where configurations need to be dynamically loaded during execution. Here's how you can approach this problem step-by-step.

Firstly, the question is essentially asking us to:

Identify which tool to use for parsing YAML.
Understand how to pass parameters to these tools.
Write a shell script that reads and processes the configuration file based on given keys or paths.

We will assume some common assumptions:

We have a config.yaml file.
The pipeline uses a shell script to read specific values from this file.
We are using yq, which is a lightweight, powerful, and flexible command-line YAML parser that supports jq-like syntax for querying YAML documents. Alternatively, we could use tools like yaml-cat.

Now let's break down the process:

Step 1: Install yq
BASH
# If not already installed, install yq via your package manager
sudo apt-get update && sudo apt-get install -y yq
Step 2: Write a Shell Script to Read Configuration Values

We will create a shell script read_config.sh that reads a specific value from the YAML file.

BASH
#!/bin/bash

# Define the path to your config.yaml file and the key you want to fetch
CONFIG_FILE=config.yaml
KEY_TO_FETCH="apiUrl"

# Use yq to extract the value based on the provided key
API_URL=$(yq eval ".${KEY_TO_FETCH}" ${CONFIG_FILE})

if [ -z "${API_URL}" ]; then
    echo "Key not found: ${KEY_TO_FETCH}"
else
    echo "API URL is set to: ${API_URL}"
fi

# Optionally, you can use the value in your script for further processing
echo "Using API URL: ${API_URL} for further operations..."
Step 3: Example YAML Configuration File

Here's a sample config.yaml file:

YAML
apiUrl: https://example.com/api
timeoutSeconds: 30
Step 4: Run the Shell Script in Your Pipeline

You can run this script as part of your CI/CD pipeline using tools like Jenkins, GitLab CI, or GitHub Actions.

#### Example
Workflow Configuration for GitLab CI:
YAML
stages:
  - setup

read_config_job:
  stage: setup
  script:
    - ./read_config.sh

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that your shell scripts can dynamically read and use configuration values from a YAML file. Key points to remember are:

Ensure the yq tool is available in the environment.
Use paths correctly with yq eval.
Handle cases where keys might not exist gracefully.

#### Common mistakes
include:

Forgetting to check if the key exists before using its value.
Not installing or sourcing necessary tools.
Misusing eval syntax, leading to incorrect parsing of values.

For a more advanced setup, consider integrating these scripts with other CI/CD tools and ensuring they are idempotent for better reliability.

