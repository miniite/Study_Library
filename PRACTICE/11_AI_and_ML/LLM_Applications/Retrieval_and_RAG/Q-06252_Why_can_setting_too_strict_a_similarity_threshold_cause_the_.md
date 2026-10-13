---
type: "practice"
difficulty: "Intermediate"
---

# Why can setting too strict a similarity threshold cause the model to answer 'I don't know'?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Re-ranking](<../../../../11_AI_and_ML/LLM_Applications/Retrieval_and_RAG/Re-ranking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with similarity thresholds in retrieval strategies, setting them too strictly means that only highly relevant chunks of information are considered similar enough to be retrieved. If this threshold is too low, the model might retrieve irrelevant data and give incorrect answers. Conversely, if it's set too high, the model may struggle to find any matching chunk at all, leading to situations where it cannot provide a useful response.

In this context, setting an overly strict similarity threshold can cause the model to answer 'I don't know' for several reasons:

Insufficient Relevant Data: The search space might be too narrow, such that no chunks match the query closely enough.
Ambiguity in Queries: Sometimes, a query can have multiple valid interpretations or solutions, and if the threshold is too high, these may not all be considered relevant.

To illustrate this concept, imagine you are working on an AI system that answers questions about medical diagnoses based on symptoms. If your similarity threshold is set very high, and a user asks "What could cause sudden headaches?" with a strict threshold, the model might only consider chunks related to severe conditions like migraines or brain tumors. If there isn't a chunk explicitly mentioning these conditions in the available data, the system may fail to retrieve anything relevant.

Let's break down the first-principles reasoning step-by-step:

#### Triage

The question is probing our understanding of how similarity thresholds affect the retrieval process and the model’s ability to generate accurate responses. The key assumptions are:

The AI uses a vector space or some form of embedding for chunking.
There is a threshold set by which chunks are deemed similar enough.

#### First-Principles Solution
Understand the Similarity Metric: Typically, this involves comparing vectors in a high-dimensional space to see how close they are.
Threshold Setting: The threshold determines what level of closeness between vectors counts as "similar." If set too low, many chunks may be deemed similar; if too high, few or none might.
Impact on Retrieval: A higher threshold means more stringent matching criteria, potentially missing out on partially relevant data.
Error Handling in Response Generation: When no sufficiently similar chunks are found, the model defaults to a fallback response like "I don't know."
Detailed Example

Imagine an AI system that processes text queries about technical support issues. The similarity threshold is set very high, so it requires an exact match for any chunk to be considered relevant.

Query: "How do I connect my printer?"
Chunk Examples:
"Printing issues can sometimes be resolved by checking your printer settings."
"For setup instructions, refer to page 3 of the user manual."

If the threshold is too strict and no chunks match exactly, the model might return an "I don't know" response.

#### Closing Summary

Setting similarity thresholds that are too high can lead to the AI answering 'I don’t know' more frequently because it becomes overly selective about what counts as a valid retrieval. This problem highlights the importance of balancing precision and recall in information retrieval systems, ensuring they provide useful answers while avoiding over-strict conditions.

#### Common Mistakes/Follow-Ups
Misunderstanding Thresholds: Not understanding that setting thresholds too high can lead to under-retrieval.
Overfitting on Exact Matches: Assuming that only perfectly matching chunks are relevant without considering partial or contextually similar matches.
Lack of Fallback Mechanisms: Not having strategies in place for when no adequate chunks are found, such as providing more generic but still helpful information.

