---
type: "practice"
difficulty: "Intermediate"
---

# How would you check if a remote host is reachable on a specific TCP port from Linux?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NAT and Port Exhaustion](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/NAT_and_Port_Exhaustion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check if a remote host is reachable on a specific TCP port from a Linux machine, we can use basic networking tools like ping, telnet, and nc (Netcat). This task involves understanding how these tools work at the network level to ensure both the reachability of the host and the availability of the specific service running on that port. Let's break down the steps from first principles.

**Comparison:**

```
Tool	Usage	Notes
ping	To check network reachability and latency.	Uses ICMP protocol; can give a general idea of whether the host is up but does not check specific ports.
telnet	For simple TCP-based service testing.	Can be used to establish a connection, which implies that the port might be open, though it’s prone to false positives (e.g., if there's no service).
nc (Netcat)	Comprehensive tool for port scanning and network troubleshooting.	More flexible; can send arbitrary data over a network and receive responses.
```

**Code:**

```
SH
Copy
# Example commands to check TCP port connectivity
ping -c 4 example.com       # Ping the remote host four times
telnet example.com 80      # Try a basic telnet connection (HTTP default)
nc -zv example.com 22       # Scan for open ports using netcat
```

**Example input / output:**

```text
# Example output for ping
PING example.com (93.184.216.34) 56(84) bytes of data.
64 bytes from ec2-93-184-216-34.compute-1.amazonaws.com (93.184.216.34): icmp_seq=1 ttl=50 time=57.9 ms
64 bytes from ec2-93-184-216-34.compute-1.amazonaws.com (93.184.216.34): icmp_seq=2 ttl=50 time=56.8 ms

# Example output for telnet
Trying 93.184.216.34...
Connected to example.com.
Escape character is '^]'.

# Example output for netcat
nc: connect to example.com port 22 (tcp) failed: Connection refused
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the host reachable via ping?
7. Use telnet or nc to checkspecific port
8. Host is down; network issue
9. Check if service is listening onthe port
10. Service is running andaccessible
11. Port closed, firewalls, ormisconfiguration

