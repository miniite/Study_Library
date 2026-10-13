---
type: "practice"
difficulty: "Deep"
---

# How do you separate quality-attribute requirements the GenAI platform must meet from 'nice-to-have' capabilities teams merely want during utility-tree construction? Why does ATAM force this separation before analysis, and how does it stop the AI roadmap from becoming a demo-driven wishlist?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Architectural Trade-off Analysis (ATAM), separating quality-attribute requirements from 'nice-to-have' capabilities is crucial to ensure that an AI platform, such as a GenAI system, is developed with clear priorities and realistic expectations. This separation helps in creating a robust roadmap that aligns with business goals and technical constraints. By forcing this distinction before analysis, ATAM prevents the development process from being swayed by short-term desires or demonstrations, ensuring that the final product meets critical quality attributes.

ATAM starts by identifying all relevant quality-attributes (QAs) for the platform, such as reliability, security, scalability, maintainability, usability, and performance. These are essential for the system to function correctly in its intended environment. On the other hand, 'nice-to-have' capabilities are secondary features that might enhance user experience but are not strictly necessary for basic functionality.

To triage these requirements effectively:

Quality-Attribute Requirements: These must be prioritized and clearly defined based on business needs and risk assessment.
Nice-to-Have Capabilities: These should be documented separately as potential enhancements or future work items.

For example, a GenAI platform might need to ensure data privacy (QA) but could benefit from more advanced natural language processing techniques (nice-to-have).

**Comparison:**

```
Quality Attribute	Importance
Reliability	High
Security	High
Usability	Medium
Performance	Low
Nice-to-Have Capabilities
Advanced NLP
Integration with third-party services
```

**Code:**

```python
# Example of defining quality-attribute requirements and nice-to-have capabilities in Python

class QualityAttribute:
    def __init__(self, name, importance):
        self.name = name
        self.importance = importance  # High, Medium, Low

quality_attributes = [
    QualityAttribute("Reliability", "High"),
    QualityAttribute("Security", "High"),
    QualityAttribute("Usability", "Medium"),
    QualityAttribute("Performance", "Low")
]

nice_to_haves = [
    "Advanced NLP",
    "Integration with third-party services"
]
```

**Example input / output:**

```text
Input:
Quality Attributes: Reliability (High), Security (High), Usability (Medium), Performance (Low)
Nice-to-Have Capabilities: Advanced NLP, Integration with third-party services

Output:
- Prioritize and focus on reliability and security for the initial development phase.
- Document advanced NLP and integration as potential future enhancements or optional features.
```

**Data:**

```json
{
  "qualityAttributes": [
    {"name": "Reliability", "importance": "High"},
    {"name": "Security", "importance": "High"},
    {"name": "Usability", "importance": "Medium"},
    {"name": "Performance", "importance": "Low"}
  ],
  "niceToHaves": [
    "Advanced NLP",
    "Integration with third-party services"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Are these requirementscritical?
5. Identify as Quality Attribute
6. Document as Nice-to-Have
7. Assign higher priority duringdevelopment
8. Mark for future enhancementsor optional features

