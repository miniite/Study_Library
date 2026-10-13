---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a plugin/extension for a gateway (e.g., a Kong Lua or AWS Lambda authorizer) that adds custom header-based routing, and how do you test and version it safely?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Routing and BFF Patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a plugin for an API Gateway that adds custom header-based routing, we need to ensure that our solution is flexible, secure, and maintainable. The key steps are to understand the requirements, architect the solution from first principles, implement the logic, test it rigorously, and version it safely.

The core idea is to create a plugin that intercepts incoming requests, checks for specific custom headers, and routes them accordingly. This involves defining clear rules based on header values, possibly using regular expressions or string comparisons, and then configuring the API Gateway to use this plugin.

**Comparison:**

```
Header Value	Route
match1	service1
match2	service2
```

**Code:**

```
LUA
Copy
-- Example Lua script for Kong Gateway

local utils = require "kong.tools.utils"
local cjson = require "cjson"

-- Plugin configuration
function access(conf)
    -- Extract custom headers from request
    local header_value = ngx.req.get_headers()["X-Custom-Header"]

    if not header_value then
        -- If no custom header, return 403 Forbidden
        ngx.log(ngx.WARN, "No custom header found")
        return 403, cjson.encode({ message = "Missing required custom header" })
    end

    -- Routing logic based on header value
    if utils.regex.match(header_value, conf.regex) then
        -- Route to the appropriate service or endpoint
        local route_table = {
            ["match1"] = "service1",
            ["match2"] = "service2"
        }

        ngx.var.upstream_uri = "/" .. route_table[header_value]
    else
        ngx.log(ngx.WARN, "Custom header value does not match any configured regex")
        return 403, cjson.encode({ message = "Header value does not match" })
    end

    -- Continue processing the request with updated URI
end
```

**Example input / output:**

```

Request:

HTTP
Copy
GET /api/resource HTTP/1.1
Host: example.com
X-Custom-Header: match1

Response:

JSON
Copy
{
  "status": 200,
  "body": "Response from service1"
}

Request:

HTTP
Copy
GET /api/resource HTTP/1.1
Host: example.com
X-Custom-Header: invalid-value

Response:

JSON
Copy
{
  "status": 403,
  "body": "{'message': 'Header value does not match'}"
}
```

**Data:**

```yaml
# Example Kong plugin configuration in YAML

plugin: custom-header-router
regex: "^(match1|match2)$"
```

**Flow:**

1. No
2. Yes
3. Match
4. No
5. Receive Request
6. Extract X-Custom-Header
7. Check if header exists
8. Log warning, return 403
9. Match regex pattern
10. Determine route based onregex match
11. Log warning, return 403
12. Set `upstream_uri` to matchedservice endpoint
13. Continue processing withupdated URI

