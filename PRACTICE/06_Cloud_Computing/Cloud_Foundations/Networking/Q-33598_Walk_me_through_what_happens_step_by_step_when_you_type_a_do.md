---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through what happens step by step when you type a domain name in a browser and press Enter.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS and Private Endpoints](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_and_Private_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you type a domain name into a browser and press Enter, several steps happen to resolve the domain name to an IP address and establish a connection with the target server. This process involves DNS resolution and TCP/IP protocols. Let's walk through these steps in detail.

Firstly, we need to understand that when you enter a domain name like example.com, your browser doesn't know which IP address it corresponds to. Instead, it uses a Domain Name System (DNS) to resolve the domain name into an IP address. The DNS resolution process typically involves several layers: client, resolver, and recursive DNS server.

#### Triage

The question is asking about the entire journey from typing a URL in a browser to establishing a connection with the target server. This includes understanding how the browser interacts with its local network stack, DNS servers, and ultimately the server hosting the content.

#### First-Principles Solution

Client Request:

When you type http://example.com into your browser and press Enter, the browser sends an HTTP request to the web server.

DNS Resolution:

The browser queries its local DNS cache or the operating system's resolver (e.g., Windows' hosts file or Unix/Linux systems like /etc/hosts) for the IP address of example.com.
If not found, the browser sends a DNS query to a configured DNS server. Typically, this could be your ISP’s DNS server or a public one like Google's 8.8.8.8.

DNS Server Response:

The DNS server checks its own records (or queries higher-level DNS servers) and responds with the IP address of example.com.

TCP Connection Setup:

With the IP address, the browser now initiates a TCP connection to the target server.
The three-way handshake is performed:
SYN from your computer to the web server.
SYN-ACK from the web server back to your computer.
ACK from your computer to confirm the connection.

HTTP Request:

Once the TCP connection is established, the browser sends an HTTP request (e.g., GET / HTTP/1.1) to the server.

Response:

The web server processes the request and returns an HTTP response containing the content of the requested page.

Content Delivery:

The browser receives the response, parses it, and displays the webpage on your screen.

#### Closing Summary

In summary, when you type a domain name into a browser and press Enter, the process involves DNS resolution to convert the domain name into an IP address, followed by setting up a TCP connection with the server, and finally sending HTTP requests and receiving responses to display the content.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that every DNS query goes directly to a public DNS server without considering local caching.

#### Follow-Up
Discussing how clients can configure their systems to use multiple DNS servers for redundancy and faster resolution times (e.g., using both private and public DNS servers).

**Comparison:**

```
Step	Action	Entity
1	DNS Query initiated	Browser
2	IP Address returned by DNS	DNS Server
3	TCP Connection Established	Client & Web Server
4	HTTP Request Sent	Browser
5	HTTP Response Received	Browser
```

**Code:**

```python
# Example Python code to simulate DNS resolution
import socket

def resolve_domain_name(domain):
    try:
        ip_address = socket.gethostbyname(domain)
        print(f"Resolved {domain} -> {ip_address}")
        return ip_address
    except socket.gaierror as e:
        print(f"Failed to resolve {domain}: {e}")

resolve_domain_name("example.com")
```

**Example input / output:**

```text
Input:
- URL typed in browser: http://example.com

Output:
- DNS Resolution: example.com -> 93.184.216.34
- TCP Handshake established with web server at IP 93.184.216.34
- HTTP Request sent: GET /
- HTTP Response received and webpage displayed in browser.
```

**Data:**

```json
{
  "domain": "example.com",
  "resolved_ip": "93.184.216.34"
}
```

**Flow:**

1. Yes
2. No
3. User Types URL
4. Browser sends DNS Query toLocal Resolver
5. Local Resolver checks cache
6. Cached?
7. Local Resolver returns cachedIP
8. Local Resolver queries PublicDNS Server
9. Public DNS Server resolvesdomain and returns IP
10. Browser uses resolved IP toinitiate TCP connection
11. TCP Three-Way Handshakewith Web Server
12. Web Server Responds withHTTP Request
13. Browser displays receivedcontent

