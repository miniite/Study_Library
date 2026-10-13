---
type: "practice"
difficulty: "Intermediate"
---

# An interviewer asks why you would not store the database password in the container image even though the image is 'private'. What layers of exposure does a baked-in secret create?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Secrets and Key Management](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/Secrets_and_Key_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Storing database passwords directly within container images, even if they are private, introduces significant security risks and can compromise your application's integrity and availability. In this context, "private" means that only authorized individuals have access to the image, but it does not mitigate exposure in other layers of your infrastructure.

Firstly, let's consider why storing secrets directly within container images is problematic:

Version Control and Public Disclosure: Container images are often version-controlled using Git or similar tools. If a developer accidentally pushes an image with sensitive data like database passwords to a public repository (e.g., GitHub, Docker Hub), the secret can be exposed publicly.

Image Registry Security: Even if the registry is private, it's not immune to security breaches. Malicious actors might gain access through vulnerabilities in the registry or through social engineering attacks targeting administrators.

Logging and Auditing: Container orchestration tools like Kubernetes log container activities, including image pulls and runs. If a sensitive secret is stored within an image, it can leave traces in logs that may be audited or exposed over time.

Secret Leakage Through Volumes and ConfigMaps: When containers are deployed with volumes or secrets mounted from a ConfigMap, any secret stored directly in the image can still be accessed by other components within the system, such as logging agents, monitoring tools, or even misconfigured applications.

To understand why this is risky, let's break it down step by step:

Version Control and Public Disclosure: In a typical development workflow, container images are often pushed to private registries for deployment. However, developers might sometimes push these images to public repositories by mistake. Even if you have strict policies in place, human error can still lead to accidental exposure.

Image Registry Security: Private registries are not completely secure. While they restrict access through authentication and authorization mechanisms, no system is entirely immune to breaches. If a breach occurs, an attacker could gain full control over the registry and any stored images.

Logging and Auditing: Container orchestration tools log detailed information about container activity, including image pulls and runs. These logs can be audited by security teams or even accessed by attackers. Storing sensitive data like database passwords in plain text within images can leave traces that might be used to infer the secrets.

Secret Leakage Through Volumes and ConfigMaps: When containers are deployed with volumes or ConfigMaps, any secret stored directly in the image can still be accessed by other components within the system. For example, a logging agent running on the same host as your application could potentially read the password from an environment variable.

To mitigate these risks, it's best to store sensitive data like database passwords externally and mount them into containers at runtime using secrets management tools such as Kubernetes Secrets or AWS Secrets Manager. This approach ensures that:

Seperation of Concerns: Sensitive data is not embedded within the application code or container images.
Fine-Grained Access Control: Secrets can be managed with fine-grained access controls, ensuring only authorized users and processes have access to them.
Auditability: Changes to secrets can be audited without exposing sensitive information in logs or other traceable artifacts.

In summary, storing database passwords directly within container images introduces unnecessary risks and exposure points. Using external secret management tools provides a more secure and controlled approach for managing sensitive data in production environments.

#### Common mistakes
and follow-ups:

Mistake: Storing secrets like database credentials directly in the container image.

#### Follow-Up
Discussing how to securely manage secrets, such as using Kubernetes Secrets or AWS Secrets Manager, and ensuring proper access controls are in place. Additionally, exploring best practices for rotating and updating secrets over time.

**Comparison:**

```
Layer of Exposure	Description
Version Control	Secrets can be pushed to public repositories.
Image Registry	Private registries are not completely secure.
Logging	Logs may contain traces of secrets, even when stored externally.
Volumes/ConfigMaps	External secrets can still be accessed by other components.
```

**Code:**

```yaml
# Example of a Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
data:
  username: c3VwZXJzZWNyZXQ=
  password: cGFzc3dvcmQ= # Base64 encoded "password"
```

**Example input / output:**

```

A sample input to a Kubernetes Secret would be:

Input:

Username: admin
Password: securepassword123

Output:

YAML
Copy
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c2VjcmV0MjMy
```

**Data:**

```json
{
  "username": "admin",
  "password": "securepassword123"
}
```

**Flow:**

1. Yes
2. No
3. Store Database PasswordDirectly in Image
4. Accidental Exposure?
5. Log, Audit, and Breach Risks
6. Still High Risk if Exposed viaOther Layers
7. Sensitive Information Leaked
8. Manage Secrets Properly withExternal Tools
9. No Accidental Exposure;Fine-Grained Access Control

