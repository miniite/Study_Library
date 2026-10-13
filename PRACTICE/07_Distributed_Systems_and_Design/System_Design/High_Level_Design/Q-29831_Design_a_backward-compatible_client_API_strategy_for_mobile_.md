---
type: "practice"
difficulty: "Expert"
---

# Design a backward-compatible client/API strategy for mobile apps you can't force-update — millions of users run 2-year-old app versions; how do you evolve the backend without breaking them?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a backward-compatible API and client strategy for mobile apps with significant user base running outdated versions, we must adopt a layered architecture approach that decouples new features from existing ones. This involves creating a compatibility layer on the backend to handle different app versions gracefully.

#### Triaging the Question

The question is asking us to design an API strategy that ensures backward compatibility while allowing for the gradual introduction of new features and improvements in the backend without breaking older client apps. We need to consider various aspects such as versioning, feature toggles, and eventual migration plans.

#### Assumptions
User Base Size: Millions of users with varying app versions.
Update Constraints: Users cannot be forced to update their apps immediately.
Backend Complexity: The backend needs to support multiple client versions simultaneously.

#### Performance Considerations
Efficient use of resources is critical.

#### First-Principles Solution
1. Layered Architecture

A layered architecture approach ensures that the new features are introduced gradually, without impacting older clients:

Presentation Layer (Client): The mobile app interface and user interactions.
Business Logic Layer: Handles application-specific logic and rules.
Data Access Layer (Backend): Manages data storage and retrieval.
2. API Versioning

Implement a versioned API strategy to support multiple client versions:

Major Version Bump: Introduce breaking changes only when necessary, e.g., major updates in the app’s core functionality.
Minor Version Bump: Add new features or minor improvements without breaking existing clients.
3. Feature Toggles

Use feature toggles (also known as feature flags) to enable/disable specific backend features based on client version:

Client Detection: Detect which version of the app is making a request.
Conditional Logic: Serve different responses based on whether certain features are enabled or not.
4. Compatibility Layer

Create a compatibility layer that adapts requests from older clients to newer versions, ensuring seamless integration:

Mapping Old API Calls: Map legacy API calls to their corresponding new endpoints.
Fallback Mechanisms: Provide fallback logic for old clients when they request unsupported features.
5. Migration Strategy

Plan a gradual migration strategy over time:

Phased Rollouts: Gradually push out updates to different user groups, allowing for real-world testing and bug fixes.
Data Migration: If necessary, plan data migrations to support new feature requirements without disrupting existing users.
Detailed Implementation
Versioned API Design
Version 1.0 (Legacy): Basic functionality with limited features.
Version 2.0: Introduce new features like push notifications and user profiles.
Version 3.0: Add advanced analytics and social sharing capabilities.
Client
Detect Version
Send Request
API Gateway (v1)
Feature Toggles
F
Legacy Backend
G
Enhanced Backend

#### Example
Feature Toggle Logic
JAVASCRIPT
function handleRequest(request) {

#### cons
t clientVersion = detectClientVersion(request);

    if (clientVersion < 2.0) {
        // Serve legacy response
        return legacyResponse;
    }

    if (clientVersion >= 3.0 && enableFeature('socialSharing')) {
        // Serve enhanced response with social sharing features
        return enhancedResponseWithSocialSharing;
    } else {
        // Fallback to basic response
        return basicResponse;
    }
}
Compatibility Layer Example
JAVASCRIPT
function mapLegacyRequestToNew(request) {
    switch (request.method) {
        case 'GET /v1/users':
            if (clientVersion < 3.0) {
                return GET_V2_USERS(request);
            } else {
                return GET_USERS_V3(request);
            }
        // Handle other legacy calls similarly
    }
}

#### Closing Summary
and Common Mistakes

To summarize, we need to adopt a layered architecture, implement versioned APIs with feature toggles, and create compatibility layers to ensure backward compatibility. The key is to introduce new features gradually, test them in real-world scenarios, and plan for eventual migration.

#### Common Mistakes
Ignoring Legacy Clients: Failing to consider the needs of older clients can lead to a fragmented user experience.
Over-Engineering: Adding too many layers or unnecessary complexity can slow down development and maintenance.
Inadequate Testing: Not thoroughly testing the compatibility layer and feature toggles can result in unexpected bugs.
Final Thoughts

By carefully planning and implementing these strategies, we can ensure a smooth transition to new features while maintaining backward compatibility with millions of users running older app versions.

**Comparison:**

```
Client Version	API Version	Features Enabled
<= v2.0	1.0	Basic
>= v3.0	3.0	Advanced, Social Sharing
```

**Code:**

```javascript
function handleRequest(request) {
    const clientVersion = detectClientVersion(request);

    if (clientVersion < 2.0) {
        // Serve legacy response
        return legacyResponse;
    }

    if (clientVersion >= 3.0 && enableFeature('socialSharing')) {
        // Serve enhanced response with social sharing features
        return enhancedResponseWithSocialSharing;
    } else {
        // Fallback to basic response
        return basicResponse;
    }
}
```

**Example input / output:**

```text
Request (Client v2.0):
GET /v1/users

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
    "users": [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ],
    "featuresEnabled": ["pushNotifications"]
}
```

**Data:**

```json
{
    "request": {
        "method": "GET /v1/users",
        "headers": {
            "User-Agent": "App/2.0"
        }
    },
    "response": {
        "version": 3,
        "featuresEnabled": ["socialSharing"]
    }
}
```

**Flow:**

1. Client
2. Detect Version
3. Send Request
4. API Gateway
5. Feature Toggles
6. F
7. Legacy Backend
8. G
9. Enhanced Backend

