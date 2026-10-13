---
type: "practice"
difficulty: "Deep"
---

# How would you design a CI gate for a RAG service that fails the build when retrieval quality (recall@k, MRR) drops below a threshold on a golden eval set, and how do you keep that eval fast enough for every PR?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a CI gate for a Retriever-Augmented Generation (RAG) service, we need to ensure that any pull request (PR) containing changes that degrade retrieval quality is automatically detected and blocked. The key metrics here are recall@k and Mean Reciprocal Rank (MRR). We will set up an automated pipeline where the service’s performance on a predefined golden evaluation set triggers a build gate if these metrics drop below a certain threshold.

The core idea is to maintain a consistent, fast, and reliable way to evaluate changes in every PR. This involves designing a robust CI/CD process that can quickly assess model quality without significantly increasing latency for developers.

**Code:**

```python
# Example of a basic evaluation script
import numpy as np

def evaluate_retrieval(model, eval_set, k=10):
    recall = 0.0
    mrr = 0.0
    total_docs = len(eval_set)

    for query in eval_set:
        relevant_docs = [doc.id for doc in query.relevant_docs]
        retrieved_docs = model.retrieve(query.text, k=k)

        # Calculate Recall@k
        recall += int(len(set(retrieved_docs) & set(relevant_docs)) > 0)

        # Calculate MRR
        ranks = np.array([relevant_docs.index(doc_id)+1 for doc_id in retrieved_docs if doc_id in relevant_docs])
        mrr += (1.0 / ranks[0] if len(ranks) else 0)

    recall /= total_docs
    mrr /= total_docs

    return {"recall@10": recall, "MRR": mrr}
```

**Example input / output:**

```text
# Example input and output for evaluation function
model = RetrieverAugmentedGenerationModel()
eval_set = get_eval_set()  # Function to fetch the golden evaluation set

result = evaluate_retrieval(model, eval_set)
print(result)  # Output should be {"recall@10": 0.85, "MRR": 0.72}
```

**Data:**

```json
{
  "eval_set_example": [
    {
      "text": "What is the significance of the Turing Test?",
      "relevant_docs": [
        { "id": 123456 },
        { "id": 789012 }
      ]
    },
    ...
  ]
}
```

