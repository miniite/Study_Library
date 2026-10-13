---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up CI/CD for a serverless (Lambda/Functions) backend?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to set up Continuous Integration and Continuous Deployment (CI/CD) for a serverless backend that primarily uses AWS Lambda functions. The goal is to ensure that every change in the codebase is automatically tested and deployed with minimal manual intervention.

#### Triaging the Question

The key aspects the interviewer might be probing include:

Understandability of CI/CD: How well you grasp the concept and its benefits.
Toolchain Familiarity: Knowledge of tools like AWS CodePipeline, Jenkins, GitHub Actions, etc.
Security Practices: Ensuring that sensitive data is handled correctly.
Cost Management: Understanding how serverless architectures behave with frequent deployments.

#### Assumptions
We are working on a project hosted in an AWS environment.
The codebase consists of multiple Lambda functions and possibly some other AWS services like DynamoDB, S3, etc.
The development team uses Git as their version control system.
There is a need for automated testing and deployment.
Step-by-Step Solution
Source Control Management: Ensure the code is stored in a centralized repository (e.g., GitHub or AWS CodeCommit).
Automated Builds: Use a CI tool to automatically build, test, and deploy the code.
Deployment Strategy: Implement a serverless deployment strategy using AWS Lambda, API Gateway, and other AWS services.
Setup

Initialize Git Repository:

BASH
git init

Install Necessary Tools:

For AWS CodePipeline: Ensure the necessary IAM roles are set up.
For CI tools like GitHub Actions or Jenkins: Set up pipelines and configurations.

Code Structure: Organize code into manageable directories and files, e.g., src, tests, config.

Automated Testing:

Write unit tests using a framework like Jest (for JavaScript).
Integrate these tests with the CI tool to run on each commit.

AWS Lambda Setup:

Package your code into zip files.
Define deployment packages and triggers in AWS CodePipeline.

API Gateway Configuration:

Define REST APIs or webhooks for invoking Lambda functions.
Ensure API Gateway is configured to route requests correctly.

CI Pipeline Example (GitHub Actions):

YAML
name: CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm ci
      - run: npm test
      - name: Deploy to AWS Lambda
        uses: jenkinsci/aws-lambda-deploy-action@v1.0.2
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-1
          LAMBDA_FUNCTION_NAME: myFunction
          ZIP_FILE_PATH: dist/myFunction.zip

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy API Gateway
        run: |
          aws apigatewayv2 update-api --api-id $API_ID --stage-name prod

#### Closing Summary
and Common Mistakes
Ensure all dependencies are correctly managed.
Test thoroughly to catch integration issues early.
Securely handle sensitive data like AWS credentials.

#### Common Mistakes
Ignoring Security: Forgetting to use environment variables or secret managers for AWS credentials.
Overlooked Testing: Failing to include a comprehensive test suite, leading to integration bugs post-deployment.
Incomplete Documentation: Not documenting the CI/CD process and setup details thoroughly.

**Code:**

```yaml
# GitHub Actions CI/CD example
name: CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm ci
      - run: npm test
      - name: Deploy to AWS Lambda
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-1
          LAMBDA_FUNCTION_NAME: myFunction
          ZIP_FILE_PATH: dist/myFunction.zip
        run: |
          aws lambda update-function-code --function-name $LAMBDA_FUNCTION_NAME --zip-file fileb://$ZIP_FILE_PATH

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy API Gateway
        run: |
          aws apigatewayv2 update-api --api-id $API_ID --stage-name prod
```

**Flow:**

1. Pass
2. Fail
3. Code Commit
4. GitHub Actions Trigger
5. Build & Test
6. Deploy to Lambda
7. Test Fixes
8. API Gateway Update

