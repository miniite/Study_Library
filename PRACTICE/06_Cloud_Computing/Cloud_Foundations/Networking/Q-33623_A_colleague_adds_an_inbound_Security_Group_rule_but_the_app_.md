---
type: "practice"
difficulty: "Intermediate"
---

# A colleague adds an inbound Security Group rule but the app is still unreachable. How do you debug it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS Failover](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When your application remains unreachable despite adding an inbound Security Group rule, debugging this issue involves a systematic approach to ensure that all relevant network configurations are correctly set up. This process will involve checking DNS resolution, TCP/IP connectivity, and firewall rules.

#### Triage

The question is asking how to identify the root cause of why an application is still inaccessible after configuring an inbound security group rule in AWS or another cloud environment. The interviewer is probing your understanding of networking fundamentals, specifically DNS, TCP/IP protocols, and security groups.

#### Assumptions
You are working with a cloud-based infrastructure like Amazon Web Services (AWS).
The Security Group rule has been correctly configured.
There might be issues at the network level that need to be checked.

#### First Principles Solution

Verify DNS Resolution:

Check if the domain name or IP address used in your application can be resolved successfully.
Use tools like nslookup or dig on the server to verify DNS resolution.

Check Network Connectivity:

Ensure that the network path from the source to the target is reachable using basic network commands such as ping.

#### Example
BASH
ping <IP_address_of_Target_Server>

TCP/UDP Port Verification:

Confirm that the necessary ports are open and accessible.
Use tools like nmap to scan for open ports on the target server.

#### Example
BASH
nmap -p <port_number> <IP_address_of_Target_Server>

Security Group Rules:

Double-check the inbound Security Group rules for any misconfigurations or missing rules.
Ensure that the source IP range and protocol match the expected values.

Firewall Settings:

Check if there are any local firewall rules (e.g., iptables on Linux) that might be blocking traffic.
Test disabling local firewalls temporarily to see if it resolves the issue.

Network Logs and Monitoring:

Review network logs for any errors or warnings.
Use cloud provider-specific tools like AWS CloudWatch to monitor network traffic.

Service Status Check:

Verify that the application service itself is running correctly.

#### Example
BASH
sudo systemctl status <application_service>

Traceroute and Path Analysis:

Use traceroute or tracert to analyze the path from the source to the target server.
This can help identify any hops that might be causing issues.

#### Closing Summary

By systematically verifying DNS resolution, network connectivity, security group rules, firewall settings, service status, and network logs, you can isolate and resolve the issue preventing your application from being accessible. Each step provides a clear path to troubleshooting and ensures no potential misconfigurations are overlooked.

#### Common Mistakes/Follow-ups
Focusing only on Security Groups without considering DNS or local firewalls.
Ignoring network logs and monitoring tools for insights into issues.
Not verifying the service status, leading to assumptions about application availability.

**Code:**

```bash
# Example of using nslookup to verify DNS resolution
nslookup example.com

# Example of using ping to check network connectivity
ping 192.0.2.1

# Example of using nmap to scan for open ports
nmap -p 80,443 192.0.2.1
```

**Flow:**

1. Start Debugging
2. Check DNS Resolution
3. Ping Target IP
4. Scan Open Ports with Nmap
5. Verify Security Group Rules
6. Check Local Firewall Settings
7. Review Network Logs
8. Monitor Service Status
9. Traceroute Path Analysis
10. End Debugging

