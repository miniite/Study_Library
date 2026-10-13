---
type: "practice"
difficulty: "Deep"
---

# You have limited eval budget. How do you prioritize which models get deep responsible-AI scrutiny vs. lightweight checks, and how do you defend that prioritization?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fairness Uncertainty and Abstention](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Fairness_Uncertainty_and_Abstention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a limited evaluation (eval) budget in the context of AI governance and responsible AI, the key is to prioritize models based on their risk profile, potential impact, and alignment with organizational goals. This involves a strategic approach that balances thorough scrutiny with efficient resource allocation.

First, let's break down the question: The core challenge is to decide which machine learning (ML) models require deep, thorough evaluations versus those that can be assessed more lightly. The interviewer is probing your understanding of risk management, model impact assessment, and effective prioritization strategies. Assumptions include a finite budget for detailed reviews, and the need to make trade-offs between different models.

Step-by-Step Solution

Define Risk Categories:

High-Risk Models: These are models that pose significant risks due to their potential impact on sensitive areas like healthcare, finance, or personal data privacy.
Medium-Risk Models: Models used in less critical applications but still require some level of scrutiny.
Low-Risk Models: Models with minimal risk and no direct impact on high-stakes outcomes.

Assess Model Impact:

Evaluate the potential consequences if a model's outputs are incorrect or biased. High-impact models include those deployed in critical systems where errors could result in financial loss, harm to individuals, or regulatory issues.

#### Cons
ider the scope of deployment: models serving millions of users versus those with limited usage.

Prioritize Based on Risk and Impact:

High-Risk Models: Allocate more budget for these due to higher potential risks. This might include detailed bias audits, comprehensive performance testing, and thorough validation.
Medium-Risk Models: Use a lighter approach such as partial bias checks or quick accuracy assessments.
Low-Risk Models: Perform minimal checks but ensure that all models are at least reviewed for basic issues.

Risk Mitigation Strategies:

Implement automated monitoring systems to continuously track model performance and identify issues early.
Establish clear documentation and governance frameworks for each model, ensuring transparency and accountability.
Regularly update the prioritization list as new models come online or existing ones evolve in their use cases.

Defend Prioritization:

Communicate the rationale behind your prioritization strategy to stakeholders, emphasizing the importance of focusing on high-risk areas while being efficient with resources elsewhere.
Use metrics and examples to illustrate the potential impact of thorough vs. light scrutiny. For instance, a case where a poorly audited model led to significant financial losses or regulatory penalties can highlight the need for rigorous checks.

#### Closing Summary

In summary, prioritizing models based on risk and impact is crucial when resources are limited. By categorizing models into high-, medium-, and low-risk groups and allocating budgets accordingly, we ensure that high-priority areas receive thorough scrutiny while still maintaining operational efficiency. Defending this approach requires clear communication of the rationale and leveraging metrics to demonstrate its effectiveness.

#### Common Mistakes/Follow-Ups
Failing to Define Risk Clearly: Not establishing a robust framework for identifying high-risk models can lead to resource misallocation.
Ignoring Continuous Monitoring: Over-reliance on initial evaluations without ongoing oversight can miss emerging issues.
Lack of Communication: Failing to effectively communicate the prioritization strategy and its benefits to stakeholders may result in resistance or misunderstanding.

**Comparison:**

```
40%
35%
25%
Model Risk Categorization
High-Risk
Medium-Risk
Low-Risk
```

**Code:**

```python
# Example code for categorizing models based on risk

def categorize_models(models):
    high_risk = []
    medium_risk = []
    low_risk = []

    for model in models:
        if is_high_risk(model):
            high_risk.append(model)
        elif is_medium_risk(model):
            medium_risk.append(model)
        else:
            low_risk.append(model)

    return high_risk, medium_risk, low_risk

def is_high_risk(model):
    # Criteria for high-risk models
    return model.deployed_in_critical_system and model.handling_sensitive_data

def is_medium_risk(model):
    # Criteria for medium-risk models
    return not is_high_risk(model) and model.deployment_scope >= 100000
```

**Example input / output:**

| Input Models | High-Risk | Medium-Risk | Low-Risk |
|--------------|-----------|-------------|----------|
| Healthcare Predictor | Yes | No | No |
| Stock Market Analyzer | No | Yes | No |
| Weather Forecast Model | No | No | Yes |

**Data:**

```yaml
models:
- name: Healthcare Predictor
  deployed_in_critical_system: true
  handling_sensitive_data: true
- name: Stock Market Analyzer
  deployed_in_critical_system: false
  handling_sensitive_data: true
  deployment_scope: 120000
- name: Weather Forecast Model
  deployed_in_critical_system: false
  handling_sensitive_data: false
  deployment_scope: 50000
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is model deployed in a criticalsystem?
7. High-Risk
8. Does the model handlesensitive data or largedeployment scope?
9. High-Risk
10. Low-Risk

