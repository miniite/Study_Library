---
type: "practice"
difficulty: "Deep"
---

# Your gateway sits behind a cloud load balancer and clients report TLS handshake failures only for a subset; how do you debug SNI, cert chain, and cipher mismatches across the LB-to-gateway path using openssl and access logs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Connection and Resource Limits](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Connection_and_Resource_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When debugging TLS handshake failures specifically related to Server Name Indication (SNI), certificate chains, and cipher suites in a setup with an API Gateway behind a cloud load balancer, it's crucial to methodically analyze each component of the path. By leveraging openssl for detailed SSL/TLS diagnostics and correlating logs from both the load balancer and gateway, you can isolate issues.

Triaging the Problem

The question is probing your ability to troubleshoot complex network interactions and understand the intricacies of TLS handshakes. You need to consider assumptions such as:

The cloud load balancer is correctly configured.
The API Gateway certificates are properly set up.
Network paths between the client, load balancer, and gateway are secure.

#### First Principles Solution

Analyze Access Logs: Start by examining the access logs from both the cloud load balancer and the API Gateway. Look for errors related to TLS handshakes and SNI mismatches.

Use openssl s_client: This command can help you diagnose issues at each stage of the connection. You can use it on your local machine or a server.

Test Connectivity from Client to LB: Use openssl s_client -connect <load_balancer_ip>:443 to test if the client can establish a secure connection directly with the load balancer.

Test Connectivity from LB to Gateway: Similarly, use openssl s_client -connect <gateway_ip>:443 on the load balancer to check if it can reach the gateway successfully.

Compare Certificates and Ciphers: Ensure that both the load balancer and the API Gateway have matching certificates and support the same cipher suites. Use openssl s_client -showcerts -connect <load_balancer_ip>:443 and compare the output with openssl s_client -connect <gateway_ip>:443.

Detailed Debugging Steps

Check Load Balancer Configuration: Ensure that SNI is properly configured on the load balancer to handle client requests correctly.

Test Client-to-LB Connection:

SH
openssl s_client -connect <load_balancer_ip>:443

Look for error messages related to SNI, certificate chain, or cipher mismatches.

Test LB-to-Gateway Connection from Load Balancer:

SH
openssl s_client -connect <gateway_ip>:443

This command will help you understand if the issue lies with the load balancer's configuration or the gateway itself.

Analyze Gateway Logs: Check the API Gateway logs for any TLS-related errors. These can provide insights into whether the problem is within the gateway application or infrastructure.

Cross-Reference Access Logs: Compare access logs from both the load balancer and the gateway to identify patterns in failed connections. Look for discrepancies in SNI, certificate expiration dates, or cipher suite support.

#### Closing Summary

By systematically testing each component of the path and comparing logs, you can effectively diagnose and resolve TLS handshake issues related to SNI, certificate chains, and cipher mismatches. This process involves a combination of network diagnostics using openssl, configuration checks, and log analysis to pinpoint the exact cause of the problem.

#### Common Mistakes
and Follow-Ups
Assuming LB Issues Without Verifying Gateway: Always verify that the gateway is also properly configured.
Ignoring Client-Side Configuration: Ensure client-side configurations (e.g., TLS settings in browsers) are not causing unexpected behavior.
Overlooking Network Path Interference: Check for any network elements (proxies, firewalls) between the client and load balancer.

By following these steps, you can confidently identify and resolve issues related to TLS handshakes in a cloud-based API gateway setup.

**Code:**

```
SH
Copy
# Test connectivity from client to LB
openssl s_client -connect <load_balancer_ip>:443

# Test connectivity from LB to Gateway
openssl s_client -connect <gateway_ip>:443
```

