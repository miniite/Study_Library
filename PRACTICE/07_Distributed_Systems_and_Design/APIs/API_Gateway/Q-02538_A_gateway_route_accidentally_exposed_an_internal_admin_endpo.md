---
type: "practice"
difficulty: "Deep"
---

# A gateway route accidentally exposed an internal admin endpoint to the public because a wildcard was too broad; how do you design default-deny routing and a review process that catches over-permissive route patterns?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Security and Data Masking](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Security_and_Data_Masking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of inadvertently exposing sensitive internal endpoints, we need to implement a default-deny routing strategy combined with a robust review process. The core idea is to ensure that all routes are explicitly allowed by default and only those deemed safe undergo scrutiny. Here’s how we can achieve this:

#### Triaging the Question

The question essentially asks us to design an API Gateway architecture that employs strict access controls, ensuring that any route not explicitly whitelisted remains inaccessible. We also need to establish a review process for new or modified routes to identify and mitigate over-permissive patterns.

#### Assumptions
The API Gateway is built using modern technologies like AWS API Gateway or Kubernetes Ingress.
There are predefined internal endpoints that should never be exposed publicly.
Regular code reviews and testing practices are in place.

#### First Principles Solution

Default-Deny Routing:

By default, all incoming requests to the API Gateway should fail unless explicitly allowed by a whitelisted route.

Whitelisting Routes:

Only routes that pass thorough scrutiny will be added to the whitelist and exposed publicly.

Review Process:

A manual or automated review process must evaluate new or modified routes for security and compliance.
This involves checking against a set of predefined criteria, such as path patterns, HTTP methods, and authentication mechanisms.

Configuration Management:

Use configuration management tools to ensure that only approved routes are active in the production environment.
Implement change control processes to track and audit route additions or modifications.
Detailed Solution
Step 1: Default-Deny Routing

To implement default-deny routing, we can use a combination of AWS API Gateway's Authorization feature and custom authorizers. Here’s an example using AWS CloudFormation:

YAML
Resources:
  ApiGatewayRestApi:
    Type: "AWS::ApiGateway::RestApi"
    Properties:
      Name: MySecureAPI

  DefaultRouteResource:
    Type: "AWS::ApiGateway::Resource"
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "{proxy+}"

  DefaultRouteMethod:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: ANY
      MethodResponses:
        - StatusCode: 403
      ResourceId: !Ref DefaultRouteResource
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: false

  CustomAuthorizer:
    Type: "AWS::ApiGateway::Authorizer"
    Properties:
      AuthorizerUri: "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/<LambdaFunctionArn>/invocations"
      IdentitySource: method.request.header.Authorization
      Name: MyCustomAuthorizer
      RestApiId: !Ref ApiGatewayRestApi
      Type: COGNITO_USER_POOLS  # or another auth type

  DefaultRoute:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: ANY
      MethodResponses:
        - StatusCode: 403
      ResourceId: !Ref DefaultRouteResource
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: false

This configuration sets up a default route that always returns a 403 Forbidden response unless the request is authorized by the custom authorizer.

Step 2: Whitelisting Routes

We need to create specific routes and ensure they are explicitly allowed. For example:

YAML
  AdminRoute:
    Type: "AWS::ApiGateway::Resource"
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "/admin"

  AdminMethod:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: GET
      MethodResponses:
        - StatusCode: 200
      ResourceId: !Ref AdminRoute
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: true

  AdminHandler:
    Type: "AWS::Lambda::Function"
    Properties:
      FunctionName: AdminHandler
      Runtime: python3.8
      Handler: index.handler
      Code:
        S3Bucket: <your-bucket>
        S3Key: handler.zip
Step 3: Review Process

Implement a continuous integration and deployment (CI/CD) pipeline that automatically deploys new routes after thorough testing and approval.

Automated Testing:

Use unit tests to verify route logic.
Integrate with security scanners to check for vulnerabilities.

Manual Approval:

Have a review board or automated tool that checks each proposed route against predefined patterns.

Audit Logs:

Maintain logs of all route changes and access attempts, which can be reviewed post-incident.

#### Closing Summary

In summary, implementing default-deny routing involves setting up strict access controls through custom authorizers and whitelisting approved routes. A robust review process ensures that only safe and necessary routes are exposed publicly, mitigating the risk of accidental exposure.

#### Common Mistakes/Follow-Ups
Overly Permissive Default Policies: Ensure that default policies always deny access unless explicitly allowed.
Insufficient Testing: Automated and manual testing should be a critical part of the deployment process.
Manual Review Shortcuts: Avoid shortcuts in the review process, as every route must be carefully evaluated for security.

By following these steps, we can significantly reduce the risk of exposing sensitive internal endpoints to unauthorized users.

**Comparison:**

| Route         | Method | Path          | Authentication Required?  | Default Action             |
|---------------|--------|---------------|----------------------------|---------------------------|
| Default Route | ANY    | {proxy+}      | Yes (Custom Authorizer)    | Forbidden                 |
| Admin         | GET    | /admin        | Yes (Custom Authorizer)    | Authorized                |

**Code:**

```yaml
# AWS CloudFormation template snippet for default-deny routing and whitelisting

Resources:
  ApiGatewayRestApi:
    Type: "AWS::ApiGateway::RestApi"
    Properties:
      Name: MySecureAPI

  DefaultRouteResource:
    Type: "AWS::ApiGateway::Resource"
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "{proxy+}"

  DefaultRouteMethod:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: ANY
      MethodResponses:
        - StatusCode: 403
      ResourceId: !Ref DefaultRouteResource
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: false

  CustomAuthorizer:
    Type: "AWS::ApiGateway::Authorizer"
    Properties:
      AuthorizerUri: "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/<LambdaFunctionArn>/invocations"
      IdentitySource: method.request.header.Authorization
      Name: MyCustomAuthorizer
      RestApiId: !Ref ApiGatewayRestApi
      Type: COGNITO_USER_POOLS

  DefaultRoute:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: ANY
      MethodResponses:
        - StatusCode: 403
      ResourceId: !Ref DefaultRouteResource
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: false

  AdminRoute:
    Type: "AWS::ApiGateway::Resource"
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "/admin"

  AdminMethod:
    Type: "AWS::ApiGateway::Method"
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: GET
      MethodResponses:
        - StatusCode: 200
      ResourceId: !Ref AdminRoute
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: true

  AdminHandler:
    Type: "AWS::Lambda::Function"
    Properties:
      FunctionName: AdminHandler
      Runtime: python3.8
      Handler: index.handler
      Code:
        S3Bucket: <your-bucket>
        S3Key: handler.zip
```

**Example input / output:**

| Input                             | Output                                      |
|----------------------------------|---------------------------------------------|
| `curl -X GET "https://api.example.com/admin"`  | Unauthorized (401) or Forbidden (403) if not authorized     |
| `curl -H "Authorization: Bearer <token>" -X GET "https://api.example.com/admin"` | Authorized access to admin endpoint              |

**Data:**

```yaml
# Sample CloudFormation parameters

Parameters:
  LambdaFunctionArn:
    Type: String
    Description: "ARN of the Lambda function for custom authorizer"
  BucketName:
    Type: String
    Description: "S3 bucket name for storing lambda code"

Resources:
  DefaultRouteResource:
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "{proxy+}"

  CustomAuthorizer:
    Properties:
      AuthorizerUri: "arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/<LambdaFunctionArn>/invocations"
      IdentitySource: method.request.header.Authorization
      Name: MyCustomAuthorizer
      RestApiId: !Ref ApiGatewayRestApi
      Type: COGNITO_USER_POOLS

  DefaultRoute:
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: ANY
      MethodResponses:
        - StatusCode: 403
      ResourceId: !Ref DefaultRouteResource
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: false

  AdminRoute:
    Properties:
      RestApiId: !Ref ApiGatewayRestApi
      ParentId: !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: "/admin"

  AdminMethod:
    Properties:
      AuthorizationScopes:
        - __default__
      AuthorizationType: CUSTOM
      HttpMethod: GET
      MethodResponses:
        - StatusCode: 200
      ResourceId: !Ref AdminRoute
      RestApiId: !Ref ApiGatewayRestApi
      RequestParameters:
        method.request.header.Authorization: true

  AdminHandler:
    Properties:
      FunctionName: AdminHandler
      Runtime: python3.8
      Handler: index.handler
      Code:
        S3Bucket: <your-bucket>
        S3Key: handler.zip
```

**Flow:**

1. No
2. Yes
3. Auth Failed
4. Auth Succeeded
5. API Gateway Request
6. Default Route Deny
7. Is the route in whitelist?
8. Return Forbidden 403
9. Custom Authorizer Check
10. Return Unauthorized 401
11. Forward Request to Handler

