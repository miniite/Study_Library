---
type: "practice"
difficulty: "Expert"
---

# How would you evolve delivery governance so that a new endpoint can't ship without required authz, rate limits, observability, and rollback wired in, enforced by the platform rather than reviewer vigilance, without turning it into red tape that teams route around?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Delivery Platform Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Safe_Delivery_Platform_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a new endpoint ships with built-in authz, rate limits, observability, and rollback capabilities, while avoiding turning this into red tape, we need to evolve our CI/CD delivery governance by leveraging platform automation and self-service mechanisms. This involves integrating security and compliance checks directly within the CI/CD pipeline and providing clear, user-friendly workflows for developers.

#### Triage

The core of the question is about enhancing CI/CD practices so that new endpoints are automatically validated against specific requirements (authz, rate limits, observability, rollback) without relying on manual reviewer vigilance. The challenge lies in ensuring these checks do not become cumbersome or disruptive to development teams. We need to balance thoroughness with developer convenience.

#### First-Principles Solution
Define Required Checks: Establish a clear list of mandatory checks that an endpoint must pass before deployment.
Automate the Pipeline: Integrate automated tests and compliance tools into the CI/CD pipeline.
Self-Service Workflows: Provide developers with self-service options to easily add or modify these configurations.
Continuous Monitoring and Feedback: Ensure continuous monitoring of deployed services and provide feedback loops for improvements.
Detailed Implementation
1. Define Required Checks

The required checks include:

Authorization (authz)
Rate Limiting
Observability (logging, metrics, tracing)
Rollback Mechanisms

These checks should be defined in a policy file or configuration management system that the CI/CD pipeline can reference.

2. Automate the Pipeline

We need to modify our CI/CD toolchain (e.g., Jenkins, GitLab CI) to include these checks as part of the build/deploy process. For example:

YAML
# Example GitLab CI configuration snippet for authz and rate limits check
stages:
  - lint
  - test
  - deploy

authz_lint:
  stage: lint
  script:
    - echo "Running authorization checks..."
    - ./check_authz.sh

rate_limit_check:
  stage: test
  script:
    - echo "Running rate limit checks..."
    - ./test_rate_limits.sh

deploy:
  stage: deploy
  when: manual
  dependencies:
    - authz_lint
    - rate_limit_check
3. Self-Service Workflows

Develop a user-friendly self-service mechanism for developers to configure these settings without needing IT intervention:

YAML
# Example self-service configuration snippet in a configuration management system (e.g., AWS CloudFormation)
Resource "aws_api_gateway_method" "example" {
  resource_id = var.resource_id
  http_method  = var.http_method
  authorization = {
    function_name = aws_lambda_function.example.arn
    function_version = "$LATEST"
  }
}

Resource "aws_apigatewayv2_authorization_setting" "example" {
  rest_api_id      = aws_apigatewayv2_api.example.id
  stage_name       = var.stage_name
  authorization_type = "CUSTOM"
  authorizer_id    = aws_apigatewayv2_authorizer.example.id
}

Resource "aws_lambda_permission" "example" {
  function_name        = aws_lambda_function.example.name
  action              = "lambda:InvokeFunction"
  principal           = "*"
  source_arn          = aws_apigatewayv2_authorizer.example.invoke_arn
}
4. Continuous Monitoring and Feedback

Implement monitoring tools like Prometheus, Grafana, and Jaeger for observability, and use rollbacks in case of issues:

YAML
# Example service mesh configuration with Istio for automatic rollbacks
apiVersion: install.istio.io/v1alpha1
kind: InstallationSpec
metadata:
  name: istio-install
spec:
  components:
    gateways:
      - kubernetes
    sidecarInjectorWebhook:
      enabled: true

#### Closing Summary
and Common Mistakes/Follow-Ups

By automating these checks, we ensure that new endpoints are consistently validated against required standards. However, common pitfalls include overly complex configurations leading to developer frustration or neglecting important compliance checks due to perceived red tape.

#### Common Mistakes

Overly complex check procedures
Insufficient self-service options for developers
Lack of continuous monitoring and feedback mechanisms

#### Follow-Ups

How would you handle edge cases where a new endpoint is required urgently?
What measures can be taken to ensure these checks don’t become barriers to innovation?
Can you discuss the trade-offs between manual review and automated processes?

**Code:**

```
GitLab CI example snippet

stages:

lint
test
deploy

authz_lint: stage: lint script: - echo "Running authorization checks..." - ./check_authz.sh

rate_limit_check: stage: test script: - echo "Running rate limit checks..." - ./test_rate_limits.sh

deploy: stage: deploy when: manual dependencies: - authz_lint - rate_limit_check

CODE
Copy

```yaml
# Example AWS CloudFormation snippet for API Gateway and Lambda function integration
Resource "aws_api_gateway_method" "example" {
  resource_id = var.resource_id
  http_method  = var.http_method
  authorization = {
    function_name = aws_lambda_function.example.arn
    function_version = "$LATEST"
  }
}

Resource "aws_apigatewayv2_authorization_setting" "example" {
  rest_api_id      = aws_apigatewayv2_api.example.id
  stage_name       = var.stage_name
  authorization_type = "CUSTOM"
  authorizer_id    = aws_apigatewayv2_authorizer.example.id
}

Resource "aws_lambda_permission" "example" {
  function_name        = aws_lambda_function.example.name
  action              = "lambda:InvokeFunction"
  principal           = "*"
  source_arn          = aws_apigatewayv2_authorizer.example.invoke_arn
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. CI/CD Pipeline
6. Is Endpoint Authz Done?
7. Run Rate Limit Checks
8. Run Observability Checks
9. Create Rollback Plan
10. Release Endpoint
11. Manually Intervene for Authz
12. Is Rate Limit OK?
13. Proceed to Observability
14. Fix and Re-Run Checks

