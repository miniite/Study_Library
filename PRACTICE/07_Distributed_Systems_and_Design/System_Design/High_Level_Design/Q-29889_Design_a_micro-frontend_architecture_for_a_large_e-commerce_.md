---
type: "practice"
difficulty: "Deep"
---

# Design a micro-frontend architecture for a large e-commerce site owned by 15 teams — how do you compose independently-deployed frontends share design-system and auth state and stop one team's bundle bloat from tanking everyone's Core Web Vitals?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Micro-frontends](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Micro-frontends.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In designing a micro-frontend architecture for a large e-commerce site, the key challenge is to ensure that independently-deployed frontends can share design systems and authentication states while preventing any single team's bundle bloat from impacting overall performance. The solution involves leveraging a shared library for the design system, a central authentication service, and a smart load balancing strategy.

#### Triaging the Question

The question asks us to design a micro-frontend architecture where multiple teams are independently developing their parts of an e-commerce site but need to share common resources such as design systems and authentication states. The primary goal is to ensure that the composition of these frontends does not lead to performance issues, specifically Core Web Vitals (CWVs) like render time and load time.

#### Assumptions
Teams: There are 15 teams working on different parts of the e-commerce site.
Design System: A common design system needs to be shared among all teams.
Authentication: A central authentication service is required for user sessions.
Performance Metrics: Core Web Vitals (CWVs) need to be maintained at a high level.

#### First-Principles Solution
Shared Design System:
Use a monorepo approach or a shared library to house the design system components.
Ensure that all teams can pull from this central source and update it as needed.
Central Authentication Service:
Implement an OAuth 2.0-based authentication service that all frontends communicate with.
This service should handle token issuance, validation, and revocation.
Micro-frontend Composition:
Use a micro-frontends framework like WebComponents or iframe-based techniques to compose different teams' frontends on the same page.
Dynamic Load Balancing:
Implement a dynamic load balancing strategy that ensures no single frontend bundle dominates the overall load time.
Performance Monitoring and Optimization:
Continuously monitor CWVs using tools like Lighthouse or WebPageTest.
Optimize assets, use lazy loading techniques, and implement caching strategies.
Step-by-Step Solution:

Shared Design System:

Design System Monorepo
Team A Component
Team B Component
Team C Component
Create a monorepo for the design system.
Teams can import and use components from this shared library.

Central Authentication Service:

User
OAuth 2.0 Provider
Authentication Token
Frontend A
API Requests
Frontend B
API Requests
Set up an OAuth 2.0 provider to issue and validate tokens.
All frontends request authentication tokens from this service before making API requests.

Micro-frontend Composition:

Frontend A
Iframe
Frontend B
Iframe
Shared Parent Page
Use iframes or WebComponents to embed different frontends on a shared parent page.
Ensure that each iframe's load does not impact the overall performance of the site.

Dynamic Load Balancing:

Load Balancer
Budget A
Budget B
Budget C
Use a smart load balancer that dynamically allocates budgets to each frontend based on performance metrics.
For instance, if one frontend's bundle size starts to grow, its budget is reduced to ensure overall CWVs remain stable.

Performance Monitoring and Optimization:

Continuous Integration
Lighthouse
CWV Data
Optimization Actions
Integrate performance monitoring tools into the CI/CD pipeline.
Use Lighthouse to gather CWV data and trigger optimization actions based on those metrics.

#### Closing Summary

The micro-frontend architecture for a large e-commerce site involves using a shared design system, a central authentication service, dynamic load balancing, and continuous performance monitoring. By ensuring that all teams can access common resources while maintaining individual bundle sizes, the overall CWVs of the site can be preserved.

#### Common Mistakes/Follow-Ups
Ignoring Performance Metrics: Monitoring tools should not be an afterthought but integrated from the beginning.
Overcomplicating Load Balancing: Implementing overly complex load balancing strategies might introduce unnecessary complexity without significant benefits.
Neglecting Design System Updates: Ensuring that all teams are using up-to-date design components is crucial to maintain a consistent user experience.

**Comparison:**

```
Team	Component	Description
A	Button	Custom styled button component
B	Card	Responsive card component
C	Footer	Dynamic footer with links
```

**Code:**

```javascript
// Example of importing a shared component from the monorepo in a frontend project
import MyComponent from 'design-system/components/my-component';
```

**Example input / output:**

```
Example of a Dynamic Load Balancing Strategy:
JSON
Copy
{
  "frontendA": { "budget": 20, "currentLoad": 18 },
  "frontendB": { "budget": 30, "currentLoad": 25 },
  "frontendC": { "budget": 10, "currentLoad": 9 }
}
```

**Data:**

```
Sample Data for Shared Design System:
JSON
Copy
{
  "components": [
    { "name": "Button", "styles": {...} },
    { "name": "Card", "styles": {...} }
  ]
}
Authentication Token Response Example:
JSON
Copy
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkaWQ6ZXRocDovL3NyYy9uMjI4Nzg2MDk0NmQ4OTZiNGEwOTFhYzkzNWFlNTA2YWZmNDE0MTgyODUyY2I1M2Y5OGE2MTVhZDliMWJlN2U4NDQiLCJpc3MiOiJST0xFX0ZJTEVTRVIiLCJleHAiOjE2NTkxOTIzMDgsImlhdCI6MTY1OTE4NzMwOCwianRpIjoiMmFhNWE0NGItMWU0Ni00MGMxLWIyODctZTgyOGQ5YjY3MThiIn0.9a7b6d2f3150854db5e3a8c4fba02b85bca450e4f875a416f78a0f06df62a45e36a7e95126a4d24b2b5e547",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

**Flow:**

1. User
2. OAuth2Provider
3. AuthToken
4. FrontendA
5. APIRequest
6. FrontendB

