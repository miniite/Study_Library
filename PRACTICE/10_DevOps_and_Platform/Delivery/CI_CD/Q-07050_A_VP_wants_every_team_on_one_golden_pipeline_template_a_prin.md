---
type: "practice"
difficulty: "Expert"
---

# A VP wants every team on one golden pipeline template; a principal on another org insists teams keep autonomy over their own pipelines. How do you adjudicate this standardization-vs-autonomy tension and what governance model do you propose?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Golden Pipelines and Templates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Golden_Pipelines_and_Templates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a large organization, achieving balance between standardization and autonomy is crucial for ensuring both efficiency and innovation. The VP wants every team to follow one golden pipeline template, while another principal insists on maintaining flexibility and control over their own pipelines. This tension can be resolved by implementing a governance model that leverages the strengths of both approaches.

Firstly, it's important to understand that standardization improves consistency, reduces errors, and enables easier sharing of knowledge across teams. On the other hand, autonomy allows for tailored solutions based on specific team needs and can foster innovation and creativity. To find a balance, we need a hybrid governance model that strikes an optimal point between these two.

#### Triaging the Question

The VP's stance emphasizes standardization to ensure uniformity in pipeline processes, which is beneficial for large-scale deployment, compliance, and ease of maintenance. The principal’s insistence on autonomy highlights the importance of flexibility, allowing teams to adapt pipelines to their specific workflows and innovation needs.

#### Assumptions
Team Diversity: Different teams have varying workflows and requirements.
Scalability: The organization needs a scalable governance model that can accommodate different team sizes and complexities.
Flexibility vs. Consistency: There is a need to balance flexibility with consistency to ensure both innovation and reliability.

#### First-Principles Solution

To adjudicate this tension, we propose a multi-tiered governance model:

Tier 1: Golden Pipeline Template

Create a core golden pipeline template that serves as the foundation for all teams. This template will cover common practices such as CI/CD lifecycle stages, security checks, and compliance standards.

Tier 2: Customization Layers

Allow teams to add or modify certain sections of the golden pipeline based on their specific needs. For instance:

Pre-Build Steps: Teams can include additional pre-build steps like static code analysis.
Post-Deploy Steps: Teams can customize post-deploy steps such as environment-specific configuration changes.
Tier 3: Approval and Change Management

Implement a robust approval and change management process for any deviations from the golden pipeline template. This ensures that critical changes are reviewed, approved by relevant stakeholders, and documented.

Detailed Governance Model

Golden Pipeline Template:

Core functionalities and best practices.
Version-controlled repository to maintain consistency across all teams.

Customization Layers:

Define a set of "slots" where customization is allowed.
Provide guidance on when and how customizations can be made.

Approval and Change Management:

Establish a formal approval process for changes to the golden template or customization layers.
Use tools like JIRA, GitHub, or Bitbucket for tracking requests and approvals.

Governance Committee:

Form a cross-functional governance committee with representatives from each team.
Meet regularly to review pipeline changes, address issues, and provide feedback.

#### Closing Summary

By adopting this multi-tiered governance model, we can ensure that the organization benefits from standardized best practices while still allowing teams the flexibility they need. This approach balances the VP's desire for uniformity with the principal's emphasis on autonomy.

#### Common Mistakes/Follow-Ups
Over-standardization: Failing to allow enough customization can stifle innovation.
Lack of Review Process: Without proper approval and change management, pipeline changes might introduce security or compliance risks.
Inadequate Governance Committee: A poorly formed committee could lead to inconsistent decision-making.

It's important to continuously evaluate the governance model based on feedback from teams and evolving organizational needs. Regular reviews can help refine the approach over time.

**Comparison:**

```
Tier	Description	Example Customization
1	Core functionalities and best practices	Common stages like Build, Test, Deploy
2	Allow teams to add or modify certain sections	Pre-build steps (linting), Post-deploy steps (configuration)
3	Robust approval and change management process	Formal review and approval by governance committee
```

**Code:**

```
Example YAML for Golden Pipeline Template

pipeline_template: stages: - name: Build steps: - script: mvn clean install - name: Test steps: - script: mvn test - name: Deploy steps: - script: docker build -t my-app .

CODE
Copy

```yaml
# Example YAML for Customization Layers
customization_slots:
  pre_build_steps:
    - script: npm run lint
  post_deploy_steps:
    - script: ./configure-env.sh
```

**Example input / output:**

| Input | Output |
|-------|--------|
| New team requests customization of the pipeline template for a specific environment. | Approval request is submitted to the governance committee, which reviews and approves the changes after a thorough discussion. |

**Data:**

```json
{
  "pipeline_template": {
    "stages": [
      {"name": "Build", "steps": [{"script": "mvn clean install"}]},
      {"name": "Test", "steps": [{"script": "mvn test"}]},
      {"name": "Deploy", "steps": [{"script": "docker build -t my-app ."}]}
    ]
  },
  "customization_slots": {
    "pre_build_steps": [{"script": "npm run lint"}],
    "post_deploy_steps": [{"script": "./configure-env.sh"}]
  }
}
```

**Flow:**

1. Yes
2. No
3. Golden Pipeline Template
4. Does Team NeedCustomization?
5. Customize within Slots
6. Use Golden Template
7. Submit Change Request
8. Approval Committee Review
9. Approved & Implemented

