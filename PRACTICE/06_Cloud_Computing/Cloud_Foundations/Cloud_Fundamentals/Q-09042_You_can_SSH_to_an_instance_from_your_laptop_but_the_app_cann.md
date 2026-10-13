---
type: "practice"
difficulty: "Intermediate"
---

# You can SSH to an instance from your laptop but the app cannot reach the database. Where do you look?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Private Endpoints and Network Paths](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Private_Endpoints_and_Network_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a situation where an application running on an instance is unable to access a database, even though you can SSH into the instance directly from your laptop, it's crucial to systematically troubleshoot the issue by considering various layers of the system stack. Let’s break down this scenario:

Triaging the Problem

The core question here is: Where do we start looking when an application running on a cloud instance cannot access a database? The interviewer might be probing your understanding of networking, security groups, routing tables, and possibly network policies or firewalls.

#### Assumptions
& Context
You can SSH into the instance from your laptop.
You cannot access the database from the application running on the instance.
The application is correctly configured to attempt the connection to the database.
Both the application server and the database are running in a cloud environment (e.g., AWS, GCP).
Step-by-Step Solution
Network Connectivity: Check if the application server can resolve the hostname of the database. Use tools like ping or nslookup within the instance to ensure that the DNS resolution is working correctly.
Port Availability: Verify that the port used by your application for connecting to the database (e.g., 3306 for MySQL) is open and accessible from the application server. Run commands like telnet [database IP] [port number] or use a network scanner tool such as nmap.
Security Groups & Firewalls: Inspect the security groups associated with both the instance running the application and the database. Ensure that the necessary inbound rules are in place to allow traffic on the required ports.
Routing & Network Segmentation: Check if there is any network segmentation or routing issue preventing the traffic from reaching the database. This includes verifying VPC peering connections, route tables, and any other network configurations.
Instance Health & Logs: Look at system logs (e.g., /var/log/syslog for Linux) to identify any errors that might indicate connectivity issues. Similarly, review the application server’s logs for any error messages related to database connection failures.

#### Closing Summary

By systematically checking each layer of the network stack, you can isolate where the problem lies and take appropriate corrective actions. Common mistakes include overlooking security group configurations or assuming DNS resolution without verifying it from within the instance.

#### Common Mistakes
& Follow-Ups
Mistake 1: Ignoring DNS resolution issues.
Mistake 2: Not checking the application’s configuration for correct database connection strings.
Mistake 3: Failing to consider network segmentation or routing rules.

Further questions the interviewer might ask could include:

How do you troubleshoot a specific security group rule?
What are the best practices for managing VPCs in a multi-tier architecture?

**Code:**

```bash
# Example commands to check connectivity and logs

ping [database IP]
nslookup [database hostname]
telnet [database IP] 3306
journalctl -xe
```

