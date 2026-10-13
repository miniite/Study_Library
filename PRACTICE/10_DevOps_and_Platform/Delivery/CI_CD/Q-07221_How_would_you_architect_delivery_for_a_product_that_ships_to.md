---
type: "practice"
difficulty: "Expert"
---

# How would you architect delivery for a product that ships to both your cloud and air-gapped customer environments you don't control, so a single pipeline can produce and promote artifacts destined for both?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Safe Delivery Platform Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Safe_Delivery_Platform_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're tasked with architecteding a CI/CD delivery system that can handle both cloud-based and air-gapped environments without control over the latter. The key is to ensure that the same pipeline can produce and promote artifacts tailored for each environment while maintaining security and flexibility.

#### Triage

The core of this question is about designing a CI/CD pipeline that can cater to different deployment requirements—cloud-native vs. air-gapped—without compromising on consistency, security, or maintainability. We'll need to consider the challenges of deploying software in an air-gapped environment where direct access to the internet and external systems may be restricted.

#### First-Principles Solution
Modular Pipeline Design: Create a modular CI/CD pipeline that can be configured for different deployment scenarios.
Artifact Customization: Customize artifacts based on target environments (cloud or air-gapped).
Secure Communication: Ensure secure communication and artifact transfer between stages of the pipeline.
Staging Environment: Implement a staging environment to test artifacts in a cloud-like setup before deploying them into an air-gapped environment.
Step-by-Step Solution
1. Modular Pipeline Design

We'll design our CI/CD pipeline as a set of modular components that can be configured independently:

Source Code
Build
Test
Package Artifact
2. Artifact Customization

During the Package Artifact stage, we can use environment-specific configuration files and scripts to customize the artifact for each target:

JAVASCRIPT
// Example of a function that customizes artifacts based on environment
function packageArtifactForEnvironment(env) {

#### cons
t baseConfig = require('./base.config');

    if (env === 'cloud') {
        return extend(baseConfig, './cloud-specific-config.json');
    } else if (env === 'air-gapped') {
        return extend(baseConfig, './airgapped-specific-config.json', { noInternet: true });
    }
}
3. Secure Communication

To ensure secure artifact transfer between stages of the pipeline:

Use encrypted channels for communication.
Implement authentication and authorization mechanisms.
Source Code
Build
Test
Package Artifact
Artifact Storage
Secure Transfer
4. Staging Environment

Implement a staging environment that mimics the cloud environment to test artifacts before deploying them into an air-gapped environment:

Source Code
Build
Test
Package Artifact

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we need a modular pipeline that can be configured for both cloud and air-gapped environments. This involves customizing artifacts based on the environment, ensuring secure communication, and using a staging environment to test before deployment.

#### Common Mistakes

Not adequately considering security in an air-gapped environment.
Failing to properly customize artifacts for each environment.
Overlooking the need for a staging environment to ensure compatibility with cloud-native expectations.

#### Follow-Ups
Security Considerations: How would you secure the pipeline and artifacts in both environments?
Artifact Transfer Mechanism: What mechanisms do you use to transfer artifacts between stages, especially in an air-gapped environment?
Automation vs. Manual Deployment: Should we fully automate deployments or allow for manual steps where necessary?

**Comparison:**

```
Artifact	Cloud Configuration	Air-Gapped Configuration
Feature 1	true	-
Feature 2	-	true
```

**Code:**

```javascript
function packageArtifactForEnvironment(env) {
    const baseConfig = require('./base.config');

    if (env === 'cloud') {
        return extend(baseConfig, './cloud-specific-config.json');
    } else if (env === 'air-gapped') {
        return extend(baseConfig, './airgapped-specific-config.json', { noInternet: true });
    }
}
```

**Example input / output:**

```

Example input and output for artifact customization:

PLAINTEXT
Copy
Input: packageArtifactForEnvironment('cloud')
Output: { cloudFeature1: true }

Input: packageArtifactForEnvironment('air-gapped')
Output: { airgappedFeature2: true, noInternet: true }
```

**Data:**

```json
{
  "base.config": {},
  "cloud-specific-config.json": {"cloudFeature1": true},
  "airgapped-specific-config.json": {"airgappedFeature2": true, "noInternet": true}
}
```

**Flow:**

1. Source Code
2. Build
3. Test
4. Package Artifact

