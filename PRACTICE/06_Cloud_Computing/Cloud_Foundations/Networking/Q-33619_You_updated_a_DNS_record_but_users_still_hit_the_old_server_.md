---
type: "practice"
difficulty: "Intermediate"
---

# You updated a DNS record but users still hit the old server. What could be going on?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS TTLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_TTLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you update a DNS record and users are still hitting the old server, it indicates there might be issues related to caching or propagation delays in the DNS system. Here’s a step-by-step analysis:

Understanding the Problem: The core issue is that the DNS query has not yet resolved to the updated IP address of the new server.

#### Assumptions
and Scope:
The DNS record update was done correctly on your authoritative DNS server.
Users are still being directed to the old server.

Let's break down potential causes:

1. DNS Caching

DNS servers maintain a cache of DNS records for performance reasons. This means that even after updating the DNS record, the DNS caches might not have been refreshed or purged yet.

Step-by-Step Solution:

Check Cache: Verify if your local DNS resolver is still using an old cached record.

On Linux/MacOS: Use dig +trace to trace DNS requests and responses.
On Windows: Use nslookup -type=A <hostname>.

Flush Cache: Clear the cache on your local machine or network devices.

For Linux/MacOS, use commands like dscacheutil -flushcache.
For Windows, go to Control Panel > Network and Internet Settings > Network Connections and then right-click on your connection and select "Properties". From there, you can clear DNS cache.
2. Propagation Delays

DNS changes do not propagate instantly across the globe due to the nature of distributed caching. The TTL (Time To Live) value in the DNS record determines how long a particular record is cached before it expires.

Step-by-Step Solution:

Check TTL: Look at the TTL value for the updated record.

Use dig +trace <hostname> to see the TTL values.

Wait for Propagation: Allow enough time based on the TTL. For example, if your TTL is 3600 seconds (1 hour), you should wait at least that long.

3. ISP/Caching DNS Servers

ISP and public caching DNS servers like Google DNS or Cloudflare might still be using old records.

Step-by-Step Solution:

Use Public DNS Servers: Temporarily change your network’s DNS resolver to a public one (e.g., Google's 8.8.8.8, Cloudflare's 1.1.1.1) and verify if the issue persists.
4. Misconfiguration

There might be misconfigurations in other parts of the system or firewalls that are still redirecting traffic to the old server.

Step-by-Step Solution:

Check Network Config: Ensure there are no local rules (firewall, load balancers) directing traffic to the old IP.
Use iptables/ufw commands on Linux for firewall checks.
Check load balancer configurations and settings.

#### Closing Summary

The issue is most likely related to DNS caching or propagation delays. By clearing your local cache and allowing sufficient time, you should see the updated server in action. If not, consider other misconfigurations like network devices or firewalls.

#### Common Mistakes

Ignoring TTL values leading to prolonged issues.
Not flushing caches on all relevant systems (local, public DNS servers).

**Code:**

```bash
# Example command to flush DNS cache on Linux/MacOS
dscacheutil -flushcache

# Example command to check DNS cache using dig
dig +trace example.com
```

**Example input / output:**

```text
$ dig +trace example.com
; <<>> DiG 9.10.6 <<>> +trace example.com
;; global options: +cmd
.   58742 IN NS d.root-servers.net.
.   58742 IN NS m.root-servers.net.
.   58742 IN NS a.root-servers.net.
.   58742 IN NS b.root-servers.net.
...
example.com. 103966 IN A <old IP address>
```

