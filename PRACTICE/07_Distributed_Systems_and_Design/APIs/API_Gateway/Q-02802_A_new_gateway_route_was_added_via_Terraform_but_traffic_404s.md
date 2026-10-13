---
type: "practice"
difficulty: "Intermediate"
---

# A new gateway route was added via Terraform but traffic 404s even though the config looks right; how do you debug the gap between declared config, the running data-plane config dump, and DNS/target propagation?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When a new gateway route is added via Terraform but traffic 404s even though the configuration looks right, it's essential to debug the gap between the declared config in Terraform and the running data-plane configuration, as well as DNS/target propagation. This involves checking the Terraform state for correctness, comparing it with the actual running configuration, ensuring proper DNS resolution, and verifying that target services are up and reachable.

#### Triage

The question is asking how to identify and resolve issues between the declared configuration in Terraform and the deployed infrastructure. The key points being probed include:

Terraform Configuration: Ensuring the Terraform code accurately reflects intended changes.
Running Configurations: Verifying that the actual running environment matches the desired state.
DNS Resolution: Confirming that domain names resolve correctly to the expected IP addresses or hostnames.
Target Services: Checking whether the backend services are healthy and accessible from the gateway.
Step-by-Step Solution
1. Validate Terraform Configuration

First, ensure the new route is correctly defined in your Terraform configuration:

HCL
resource "aws_api_gateway_resource" "new_route" {
  rest_api_id = aws_api_gateway_rest_api.example.id
  parent_id   = aws_api_gateway_resource.parent_example.id
  path_part   = "newRoute"
}

resource "aws_api_gateway_method" "get_new_route" {
  rest_api_id            = aws_api_gateway_rest_api.example.id
  resource_id            = aws_api_gateway_resource.new_route.id
  http_method            = "GET"
  authorization = "NONE"
}
2. Compare Terraform State with Running Configuration

Use terraform state list and terraform state show <resource_name> to inspect the current state:

SH
terraform state list
terraform state show aws_api_gateway_resource.new_route

Compare these outputs with your actual running API Gateway configuration via the AWS Management Console or using tools like aws api-gateway get-resource.

3. Inspect Running Data-Plane Configuration

Use the AWS CLI to retrieve the current API Gateway configuration:

SH
aws apigateway get-resource --rest-api-id <api_id> --resource-id <resource_id>

Check if the route and method are present in the output.

4. Verify DNS Resolution

Ensure that the domain name resolves correctly using tools like nslookup or dig:

SH
nslookup example.com
5. Confirm Backend Service Health

Use network tools to verify that requests reach the backend service without being blocked by firewalls or load balancers. Tools such as curl, httpie, or even a simple HTTP request in your browser can help.

SH
curl http://example.com/newRoute
6. Check Target Group Health and Load Balancer Settings

For more complex setups, especially with ELBs or other load balancing solutions, ensure that the target group associated with the API Gateway is healthy:

SH
aws elb describe-instance-health --load-balancer-name <lb_name>

#### Closing Summary

By following these steps, you can systematically identify and resolve discrepancies between your Terraform configuration and the running environment. Key areas to focus on are validating Terraform configurations, comparing with actual configurations, ensuring proper DNS resolution, confirming backend services are healthy, and checking network and load balancing settings.

#### Common Mistakes/Follow-Ups
Misconfigurations in Terraform Code: Double-check for typos or logical errors.
Outdated State Files: Ensure you're using the latest state files from terraform apply.
DNS Propagation Delays: Allow sufficient time for DNS changes to propagate.
Network Issues: Check firewall rules, load balancer settings, and security groups.

