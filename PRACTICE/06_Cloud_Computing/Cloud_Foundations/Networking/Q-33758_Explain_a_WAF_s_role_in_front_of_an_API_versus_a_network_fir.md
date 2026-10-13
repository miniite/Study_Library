---
type: "practice"
difficulty: "Deep"
---

# Explain a WAF's role in front of an API versus a network firewall. Give an attack each one catches that the other misses.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ mTLS and Zero Trust](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/mTLS_and_Zero_Trust.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the roles of a Web Application Firewall (WAF) and a network firewall, let’s first break down what they are designed to do and how they operate in different contexts. A WAF is primarily focused on protecting web applications from common web exploits by monitoring and controlling HTTP/HTTPS traffic using security rules. On the other hand, a network firewall operates at the TCP/IP layer, filtering packets based on IP addresses and port numbers.

A WAF's role in front of an API involves inspecting the HTTP/HTTPS traffic to ensure that requests are valid and do not contain malicious payloads or patterns. For example, it can block SQL injection attacks, cross-site scripting (XSS) attempts, and other application-layer threats by analyzing the content of HTTP headers and bodies.

A network firewall, in contrast, filters packets at a lower level, inspecting metadata such as IP addresses and ports to prevent unauthorized access. It can drop packets based on criteria like source/destination IP addresses or port numbers. However, it typically does not inspect the contents of the packets beyond this basic information.

Now let's look at specific attacks that each can catch but the other might miss:

WAF Missed by Network Firewall: SQL Injection Attack

#### Cons
ider an attacker trying to inject a malicious SQL query into a web application via a form field. A network firewall will only inspect the IP address and possibly the port number, so it cannot detect or block this attack unless specifically configured to do so at the application level.

However, a WAF can catch such an attack by analyzing the HTTP request payload for patterns indicative of SQL injection. For instance, if the attacker uses keywords like SELECT or constructs that could be harmful to the database, these would trigger rules in a properly configured WAF.

Network Firewall Missed by WAF: IP-Based Denial of Service (DoS) Attack

An attacker might launch a DoS attack by sending a large number of packets with specific source and destination addresses. The network firewall can effectively drop or limit traffic based on these IP addresses, thus mitigating the DoS attack.

However, if an attacker employs techniques like IP spoofing to hide their real IP address while still causing a flood of packets, a WAF might not detect this as a malicious request unless it has specific rules set up for such scenarios. A WAF typically relies on application-level analysis and cannot preemptively drop or limit packets based solely on the contents of HTTP/HTTPS requests.

#### Closing Summary

In summary, both network firewalls and WAFs play crucial roles in securing web applications and networks but operate at different levels. Network firewalls are essential for protecting against IP-based threats and can help prevent DoS attacks by blocking suspicious traffic at the packet level. On the other hand, WAFs provide deeper application-layer security by analyzing HTTP/HTTPS requests to detect and block specific types of attacks that a network firewall cannot see.

#### Common Mistakes
Misconfiguring either component: Both firewalls rely on proper configuration to be effective.
Overrelying on one over the other: A robust security strategy should include both components working in tandem, with WAFs handling application-layer threats and network firewalls filtering traffic based on IP/ports.

**Comparison:**

```
WAF Rule	Description
SQL Injection Detection	Matches keywords like SELECT, INSERT in request payload.
Network Firewall Rule	Description
IP-Based DoS Prevention	Drops or limits traffic based on source IP address and a predefined threshold of packets per IP.
```

**Code:**

```python
# Example of a simple WAF rule for detecting SQL injection patterns
def detect_sql_injection(request):
    sql_keywords = ['SELECT', 'INSERT', 'UPDATE', 'DELETE']
    payload = request['payload'].lower()
    for keyword in sql_keywords:
        if keyword in payload:
            return True
    return False

# Example of a network firewall rule to drop traffic based on IP address
def drop_traffic(ip_address, threshold):
    if ip_address in blocked_ips or len(blocked_traffic[ip_address]) > threshold:
        return True
    return False
```

**Example input / output:**

```text
# Example WAF detection output for a SQL injection attempt
request = {
    'payload': 'SELECT * FROM users WHERE id=1'
}

if detect_sql_injection(request):
    print("Potential SQL Injection detected")
else:
    print("Safe request")

# Example network firewall response to IP-based DoS attack
ip_address = "192.0.2.1"
blocked_ips = ["198.51.100.1", "203.0.113.1"]
threshold = 100

if drop_traffic(ip_address, threshold):
    print("Dropping traffic from this IP address")
else:
    print("Allowing traffic")
```

**Data:**

```

{ "request": { "payload": "SELECT * FROM users WHERE id=1" }, "response": false }

CODE
Copy

```json
{
  "ip_address": "192.0.2.1",
  "blocked_ips": ["198.51.100.1", "203.0.113.1"],
  "threshold": 100,
  "response": true
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. HTTP Request
6. Is SQL Keyword Present?
7. Log/Block Request
8. Normal Processing
9. Network Packet
10. Is IP Address in Blacklist?
11. Dropping Traffic
12. Allowing Traffic

