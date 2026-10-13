---
type: "practice"
difficulty: "Intermediate"
---

# Design a frontend performance budget and the CI gates that enforce it. Which metrics (LCP, INP, bundle size) would you block a PR on, and how do you avoid death by false positives?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a frontend performance budget with effective CI gate enforcement, we need to focus on key metrics that directly impact user experience (UX). The primary metrics we'll consider are Largest Contentful Paint (LCP), Input Latency (INP), and bundle size. By setting thresholds for these metrics in our CI system, we can ensure that PRs do not introduce unacceptable performance regressions. However, avoiding false positives is crucial, as blocking valid changes due to minor deviations could hinder development.

Firstly, let's break down the question: The interviewer is probing how you would design a robust performance budget and its enforcement mechanism in CI. They are also interested in understanding your approach to managing the risk of false positives, which can lead to unnecessary rework or delays in the development process.

Metrics Selection
Largest Contentful Paint (LCP): This measures the time it takes for the largest visual content on the page to load and be available to the user.
Input Latency (INP): This measures how long a user has to wait before they can interact with elements on the page after initiating an action.
Bundle Size: This includes the size of JavaScript, CSS, and other assets that are loaded.
Setting Thresholds

For each metric, we need to set thresholds based on industry standards and our own performance goals. For example:

LCP: <2.5 seconds for a good user experience (90th percentile).
INP: <100 milliseconds.
Bundle Size: Aim for under 80KB.
CI Gate Enforcement

To enforce these metrics, we can use tools like Lighthouse or WebPageTest integrated into our CI pipeline. These tools provide detailed reports and allow us to set automatic gates that block PRs based on failing thresholds.

Step-by-Step Solution
Tool Selection: Choose a performance testing tool (e.g., Lighthouse).
Configuration: Set up the tool to run automated tests against deployed environments.
Test Cases: Define specific test cases for LCP, INP, and bundle size. For example:
LCP: Measure the time it takes for the main content of a page to load.
INP: Check if interactive elements respond within 100ms after user interaction.
Integration with CI: Integrate these tests into our CI pipeline so that PRs are automatically tested before merging.

#### Example
Workflow

When a developer pushes code, the following steps occur:

The CI system runs Lighthouse against the deployed app.
If any of the metrics exceed predefined thresholds, the build fails and the PR is blocked.
Developers receive detailed reports indicating which specific issues need to be addressed.
Avoiding False Positives

To avoid false positives, we must consider the context in which these metrics are measured. For example:

Environment Variability: Ensure that tests run under consistent conditions (e.g., same browser version).
Load Balancing and Caching: Consider the impact of caching strategies and load balancing configurations.
Edge Cases: Handle scenarios where small optimizations can cause thresholds to be exceeded, such as minor CSS or JS bundle changes.

#### Conclusion

In summary, designing a frontend performance budget involves selecting key metrics (LCP, INP, bundle size), setting appropriate thresholds, integrating these into our CI system, and managing false positives through careful testing and context understanding. By doing so, we ensure that the codebase remains performant while minimizing unnecessary disruptions in development.

#### Common mistakes
include overly strict thresholds leading to frequent false positives or ignoring edge cases, which can result in suboptimal performance without effective feedback.

**Comparison:**

```
Metric	Threshold (ms/KB)
LCP	<2500
INP	<100
Bundle	<80
```

**Code:**

```javascript
// Example configuration for Lighthouse
const lighthouseConfig = {
  extends: 'lighthouse:default',
  settings: {
   OnlyAudits: ['LCP', 'INP'],
    failOnWarnings: true,
    passThresholds: {
      'LCP': { value: 2500, scale: 'ms' },
      'INP': { value: 100, scale: 'ms' },
    }
  }
};
```

**Example input / output:**

```text
Input:
- Developer pushes a PR with changes to the frontend.

Output:
- CI system runs Lighthouse tests.
- If any metric exceeds threshold, build fails and PR is blocked.
- Detailed report sent back to developer.
```

**Data:**

```json
{
  "lighthouseReport": [
    {
      "LCP": 2498,
      "INP": 95,
      "bundleSize": 75KB
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. CI System Triggered
4. Run Lighthouse Tests
5. LCP, INP, Bundle SizeMeasured
6. LCP > 2500?
7. Build Fails; PR Blocked
8. PR Merged & Deployed

