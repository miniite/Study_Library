---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: your architecture strategy calls for one paved road, but the ML team's needs are totally different. How do you handle the exception without fragmenting?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Balancing Architecture Pillars](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Balancing_Architecture_Pillars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a situation where my cloud architecture strategy mandates a single "paved road" approach and the machine learning (ML) team has unique requirements, it's crucial to ensure that both the strategic vision of the architecture and the specific needs of the ML team are reconciled without leading to fragmentation. The key is to find a balance where commonalities can be standardized while still accommodating the unique needs of the ML team.

The first step in this process is understanding the core principles behind the "paved road" strategy: it aims to provide a consistent, scalable foundation that accelerates development and reduces operational complexity. However, every organization has specific use cases and teams with distinct requirements, such as the ML team needing custom training environments or specialized datasets.

To handle exceptions without fragmenting the architecture, we need to design a system that is flexible enough to accommodate these unique needs while still leveraging the benefits of the unified strategy. Here’s how I would approach this:

Identify Commonalities and Differences: Begin by mapping out both the "paved road" framework and the ML team's requirements. Highlight areas where they overlap (common services, shared infrastructure) and where they diverge.

Modular Design: Ensure that the architecture is built around modular components that can be extended or customized as needed. This modularity allows us to maintain a consistent core while still accommodating unique needs like ML-specific workloads.

Service Catalogs and APIs: Create a service catalog of common services provided by the "paved road" strategy, along with well-defined APIs that allow teams to interact with these services. For the ML team, this might include specialized data pipelines or custom training environments that are not part of the standard offerings but can be integrated through these APIs.

Customization Layers: Introduce customization layers where specific teams like the ML team can build on top of the base architecture. This could involve setting up dedicated environments with preconfigured tools and services tailored to their needs.

Governance and Policies: Establish governance policies that ensure any customizations do not compromise the integrity or security of the overall system. This includes regular audits and compliance checks to ensure all changes adhere to established standards.

Hybrid Architecture: Consider a hybrid approach where critical components are part of the unified architecture, but less-critical or highly-specialized components (like ML training environments) can exist in isolated subnets or dedicated clusters that still benefit from shared infrastructure services.

Iterative Refinement: Continuously refine and iterate on this design based on feedback from teams like the ML team to ensure it remains flexible yet robust.

By following these steps, we can maintain a unified architecture while still accommodating the unique needs of the ML team. This approach ensures that both the strategic vision and the practical realities are balanced, leading to an effective and cohesive cloud strategy.

**Comparison:**

```
Component	Type	Description
Data Storage	Common	Centralized storage accessible to all teams, ensuring consistency in data management.
ML Training	Custom	Specialized environments tailored for machine learning tasks, isolated from the main architecture but integrated through APIs.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Unified Architecture] --> B(Public APIs);
    C[ML Team Customization Layers] --> D(Dedicated Training Environments);
    E[Shared Infrastructure Services] --> B;
    F[Governance Policies] --> B;
    G[Hybrid Subnets/Clusters] --> H[Isolated but Integrated];
```

**Example input / output:**

```text
Input:
- Unified architecture plan with common services and shared infrastructure.
- ML team needs for specialized training environments.

Output:
- Integration of ML-specific customizations into the unified architecture via APIs and customization layers.
- Dedicated subnets or clusters for ML workloads that still benefit from shared services.
```

**Data:**

```json
{
  "architecture": {
    "paved_road": ["common_services", "shared_infrastructure"],
    "ml_team_customizations": ["specialized_training_environments", "custom_data_pipelines"]
  },
  "services": [
    { "name": "Data Storage", "type": "common" },
    { "name": "Machine Learning Training", "type": "custom" }
  ]
}
```

**Flow:**

1. Unified Architecture
2. Public APIs
3. ML Team Customization Layers
4. Dedicated TrainingEnvironments
5. Shared Infrastructure Services
6. Governance Policies
7. Hybrid Subnets/Clusters
8. Isolated but Integrated

