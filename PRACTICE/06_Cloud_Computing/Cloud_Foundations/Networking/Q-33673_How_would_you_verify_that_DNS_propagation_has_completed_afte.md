---
type: "practice"
difficulty: "Intermediate"
---

# How would you verify that DNS propagation has completed after a record change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ DNS TTLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_TTLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To verify if DNS propagation has completed after changing a DNS record, we need to ensure that all authoritative and caching DNS servers have updated their records. This process involves checking various aspects such as DNS resolution from different locations, verifying cache refresh times, and using tools for monitoring the propagation.

The core idea here is to systematically test the DNS infrastructure to confirm that updates are visible. We will probe multiple stages of DNS lookup: querying authoritative servers directly, testing recursive resolvers, and validating from edge locations where caching might occur.

**Code:**

```bash
# Example bash script for DNS propagation verification

#!/bin/bash

# Function to query DNS records using dig
query_dns() {
    local domain=$1
    local record_type=$2
    nslookup $domain | grep -i "Address" | tail -n 1
}

# Define the domain and record type (e.g., A, CNAME)
domain="example.com"
record_type="A"

echo "Verifying DNS propagation for ${domain} with record type: ${record_type}"

# Query from authoritative server
authoritative_server=$(query_dns $domain $record_type)

if [ -z "$authoritative_server" ]; then
    echo "Authoritative DNS query failed."
else
    echo "Authoritative server response: $authoritative_server"
fi

echo ""
echo "Verifying DNS propagation from public recursive resolver (Google DNS)"
public_resolver=$(dig +short ${domain} @8.8.8.8)

if [ -z "$public_resolver" ]; then
    echo "Public resolver query failed."
else
    echo "Public resolver response: $public_resolver"
fi

echo ""
echo "Verifying DNS propagation from edge location (using a different ISP)"
edge_location=$(dig +short ${domain} @208.67.222.222)

if [ -z "$edge_location" ]; then
    echo "Edge location query failed."
else
    echo "Edge location response: $edge_location"
fi

# Additional check for TTL and cache refresh (if applicable)
echo ""
echo "Verifying DNS record TTL (Time To Live) to ensure caching is not preventing updates"
ttl=$(dig +nocmd ${domain} SOA | sed -n 's/^.*TTL\s*\([0-9]*\).*/\1/p')
echo "DNS Record TTL: $ttl seconds"

# Wait for cache refresh based on TTL
sleep $((ttl+60))

echo ""
echo "Retrying DNS query after cache refresh"
refreshed_response=$(dig +short ${domain} @8.8.8.8)
echo "Refreshed response: $refreshed_response"
```

**Example input / output:**

```text
Verifying DNS propagation for example.com with record type: A

Authoritative server response: 192.0.2.1

Verifying DNS propagation from public recursive resolver (Google DNS)
Public resolver response: 192.0.2.1

Verifying DNS propagation from edge location (using a different ISP)
Edge location response: 192.0.2.1

Verifying DNS record TTL (Time To Live) to ensure caching is not preventing updates
DNS Record TTL: 3600 seconds

Retrying DNS query after cache refresh
Refreshed response: 192.0.2.1
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. Initiate DNS Query
5. Obtain Authoritative ServerResponse
6. Compare with Expected Value
7. Verify Public ResolverResponse
8. Compare with Expected Value
9. Check Edge Location Response
10. Compare with Expected Value
11. Wait for TTL Refresh
12. Retry DNS Query After CacheRefresh
13. Confirm Updated Record

