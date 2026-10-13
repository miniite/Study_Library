---
type: "practice"
difficulty: "Deep"
---

# When a workflow needs a value from Parameter Store or an SSM secret at runtime, how do you fetch it without persisting it in the environment?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Secrets in Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Secrets_in_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address fetching values from AWS Parameter Store or SSM secrets within GitHub Actions without persisting them in the environment, we need to ensure that the secret is only used at runtime and not exposed. This involves leveraging GitHub's built-in features and best practices for handling secrets securely. Here’s a step-by-step guide on how to do this.

Firstly, the question is asking us to understand how to fetch sensitive information (like secrets) from AWS Parameter Store or SSM in a GitHub Actions workflow without storing these values in the environment variables of the pipeline. The interviewer is probing our knowledge of secure practices, best coding standards, and the use of built-in tools for handling secrets.

We assume that you have an AWS account with necessary permissions to access SSM and Parameter Store, and that your GitHub repository has the required secrets stored as Environment Variables or Secrets in GitHub’s own infrastructure. Here are the steps we can take:

Use actions/secrets from GitHub Actions: GitHub provides native support for fetching secrets using actions/secrets.
Access AWS Secrets via AWS CLI: If you need to fetch an SSM secret, use the AWS CLI or SDKs within your action.

Let's break down the solution step-by-step:

Store Secrets in GitHub: Use GitHub’s repository secrets feature to store sensitive information such as AWS credentials, access keys, and other secrets.
Use actions/secrets: In your workflow file, use ${{ secrets.YOUR_SECRET_NAME }} to reference a secret stored in the GitHub Actions environment.
Fetch SSM Secrets via AWS CLI: If you need to fetch data from an SSM parameter or secret, you can include the necessary credentials and run aws ssm get-parameter --name <PARAMETER_NAME>.

Here's a practical example:

YAML
# Example workflow file (.github/workflows/example.yml)
name: Example Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up AWS CLI
      run: |
        echo "AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}" >> $GITHUB_ENV
        echo "AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}" >> $GITHUB_ENV

    - name: Fetch SSM Parameter
      id: fetch_ssm_param
      run: |
        aws ssm get-parameter --name /my/parameter-name --query 'Parameter.Value' --output text > parameter_value.txt
      env:
        AWS_DEFAULT_REGION: "us-west-2"  # Set the appropriate region

    - name: Use Parameter Value
      run: echo "The fetched value is $(cat parameter_value.txt)"

In this example, we first set up the necessary AWS credentials in GitHub Actions using ${{ secrets }}. Then, we use the aws ssm get-parameter command to fetch a specific SSM parameter and store its value in a file. Finally, we read and use that value within the workflow.

**Code:**

```yaml
# Example workflow file (.github/workflows/example.yml)
name: Example Workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up AWS CLI
      run: |
        echo "AWS_ACCESS_KEY_ID=${{ secrets.AWS_ACCESS_KEY_ID }}" >> $GITHUB_ENV
        echo "AWS_SECRET_ACCESS_KEY=${{ secrets.AWS_SECRET_ACCESS_KEY }}" >> $GITHUB_ENV

    - name: Fetch SSM Parameter
      id: fetch_ssm_param
      run: |
        aws ssm get-parameter --name /my/parameter-name --query 'Parameter.Value' --output text > parameter_value.txt
      env:
        AWS_DEFAULT_REGION: "us-west-2"  # Set the appropriate region

    - name: Use Parameter Value
      run: echo "The fetched value is $(cat parameter_value.txt)"
```

