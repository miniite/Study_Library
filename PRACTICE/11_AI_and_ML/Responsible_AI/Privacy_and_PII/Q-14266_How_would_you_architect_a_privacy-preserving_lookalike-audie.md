---
type: "practice"
difficulty: "Expert"
---

# How would you architect a privacy-preserving lookalike-audience system so advertisers get useful targeting without you exposing or exporting individual user attributes?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Controls for Vendor Models](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Controls_for_Vendor_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a privacy-preserving lookalike-audience system, we need to ensure that advertisers can target similar users effectively while protecting individual user data from exposure. The key is leveraging differential privacy techniques and machine learning models to create synthetic datasets or embeddings without directly accessing personal information (PII). Here’s how we can approach this:

Data Collection & Initial Preprocessing:

Collect large-scale anonymized data, ensuring no direct PII is included.
Perform initial preprocessing steps such as tokenization and feature extraction.

Model Training with Differential Privacy:

Use differential privacy (DP) to train machine learning models on the data.
Introduce noise during training to ensure that any individual record does not significantly affect model outputs.

Generating Lookalike Users:

Once trained, use these models to generate lookalike users based on certain characteristics.
Ensure this process also uses DP techniques to avoid exposing specific user data.

Anonymized Reporting & Aggregation:

Provide aggregated statistics or reports back to advertisers without revealing individual identities.
Use techniques like k-anonymity, l-diversity, and t-closeness to protect privacy further.

**Comparison:**

```
Feature	Description
ID	User identifier (omitted)
Features	Numerical representations of user characteristics
PII	Personal Information (not used in model training or lookalike generation)
```

**Code:**

```python
# Example code for training a model with differential privacy
import torch
from torch.nn import functional as F

def train_dp_model(model, data_loader, optimizer, epsilon=0.1):
    # Training loop with DP
    for batch in data_loader:
        inputs, targets = batch
        noise = torch.randn_like(inputs) * 0.1  # Adding differential privacy noise
        noisy_inputs = inputs + noise

        outputs = model(noisy_inputs)
        loss = F.cross_entropy(outputs, targets)

        optimizer.zero_grad()
        (loss / len(data_loader)).backward()  # Average loss across batches
        optimizer.step()

# Example function to generate lookalike users
def generate_lookalikes(model, user_features, num_lookalikes=10):
    with torch.no_grad():
        embeddings = model(user_features)
        noise = torch.randn_like(embeddings) * 0.1  # Adding DP noise for generation
        synthetic_embeddings = embeddings + noise

    return synthetic_embeddings
```

**Example input / output:**

```text
# Input: User data features and PII
user_features = [[0.2, 0.3, 0.4], [0.5, 0.6, 0.7]]
num_lookalikes = 10

# Output: Synthetic lookalike user embeddings without PII exposure
synthetic_embeddings = generate_lookalikes(model=model, user_features=user_features, num_lookalikes=num_lookalikes)
```

**Data:**

```json
{
  "user_data": [
    {"id": 1, "features": [0.2, 0.3, 0.4], "PII": "PiiData"},
    {"id": 2, "features": [0.5, 0.6, 0.7], "PII": "MorePii"}
  ]
}
```

**Flow:**

1. Data Collection
2. Initial Preprocessing
3. Differential Privacy Training
4. Model Generation
5. Lookalike User Generation withDP
6. Anonymized Reporting

