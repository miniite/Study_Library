---
type: "practice"
difficulty: "Deep"
---

# How would you estimate the monthly cloud cost of a RAG chatbot before you build it, counting API calls and storage?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Cloud Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To estimate the monthly cloud cost of a Retrieval-Augmented Generation (RAG) chatbot in AWS, we need to consider the costs associated with API calls (both from users and from the RAG system itself), storage requirements, and potential additional services like database access or machine learning model training. This estimation involves breaking down each component’s cost and then summing them up.

#### Triaging the Question

The interviewer is probing your ability to:

Identify Key Components: Recognize the main contributors to cloud costs such as API calls, storage, and potential additional services.
Estimation Skills: Apply first-principles thinking to estimate each component accurately.

#### Assumptions
Management: Make reasonable assumptions to simplify complex real-world scenarios into manageable calculations.

#### Assumptions
API Call Costs: Estimate the number of API calls per user session, daily usage, and frequency.
Storage Requirements: Calculate storage needs for documents, chat history, and model data.
Additional Services: Consider any other services like database access or AI training that might be involved.
Step-by-Step Solution

API Call Costs

User Interaction API Calls: Assume an average of 50 user interactions per session, with each interaction making one API call to fetch documents.
RAG System API Calls: The RAG system itself makes additional API calls (e.g., for document retrieval and language processing) which are estimated as 10% of the total user interactions.
Total API Calls: Calculate daily, monthly, and yearly usage based on expected user sessions.

Storage Requirements

Document Storage: Each document is approximately 5MB, and assuming a chatbot handles 100 documents per session.
Chat History Storage: Store up to 30 days of chat history with each message being 1KB in size.
Model Data Storage: Model data can be around 5GB for training purposes.

Additional Services

Database Access: If there's a need for database access, estimate costs based on the number of queries per session.
AI Training Costs: If you're using an AWS SageMaker instance, calculate its cost based on usage and data transfer.
Detailed Calculation

API Call Costs:

User Interaction API Calls: 50 calls/session * 24 hours/day * 30 days/month = 36,000 API calls/month.
RAG System API Calls: 5% of user interactions = 1800 API calls/month.
Total API Calls: 37,800 API calls/month.

Storage Requirements:

Document Storage: 100 documents/session * 36,000 sessions/year * 5MB/document = 1.8GB/year.
Chat History Storage: 1KB/message * (50 messages/session * 24 hours/day * 30 days/month) * 12 months = 3.6GB/year.
Model Data Storage: 5GB/training run, assuming one training run every quarter.

Additional Services:

Database Access: Assuming 1,000 queries/session * 50 sessions/day * $0.004/query = $2/day or $60/month.
AI Training Costs: SageMaker instance usage of 10 hours/week at $0.3/hr = $30/month.

#### Closing Summary

The total estimated monthly cloud cost for the RAG chatbot can be calculated as follows:

API Calls: Estimated based on call frequency and service pricing.
Storage: Calculated based on document size, chat history, and model data storage needs.
Additional Services: Accounting for database access and AI training costs.

#### Common Mistakes/Follow-Ups
Overlooking Service Costs: Not considering all services used (e.g., API Gateway, S3, RDS).
Underestimating Storage Needs: Overly conservative estimates might lead to higher costs.
Ignoring Scalability and Growth: Assuming static numbers without factoring in potential growth.

Would you like to explore any of these aspects further?

**Comparison:**

| Component               | Estimated Cost |
|-------------------------|----------------|
| API Calls               | ~$150/month    |
| Storage                 | ~$90/month     |
| Database Access         | ~$60/month     |
| AI Training             | ~$30/month     |
| **Total Monthly Cost**  | **~$270/month**|

**Code:**

```python
# Example API call cost calculation
api_calls_per_session = 50
sessions_per_day = 24 * 30  # 30 days a month
total_api_calls_monthly = api_calls_per_session * sessions_per_day

# Additional RAG system calls (10% of user interactions)
rag_api_calls_monthly = total_api_calls_monthly * 0.1

# Total API calls monthly
total_api_calls_monthly += rag_api_calls_monthly
```

**Example input / output:**

```text
Input:
- API calls per session: 50
- Sessions per day: 24 * 30 (30 days a month)
- Additional RAG system calls: 10% of user interactions

Output:
- Total API calls monthly: 37,800
- Document storage needs: 1.8GB/year or ~0.15GB/month
- Chat history storage needs: 3.6GB/year or ~0.3GB/month
- Model data storage needs: 5GB
- Database access cost per month: $2/day * 30 days = $60/month
- AI training cost per month: $30/month
```

**Data:**

```json
{
  "api_call_cost": {
    "calls_per_session": 50,
    "sessions_per_day": 24 * 30,
    "total_calls_monthly": (50 + 1800) * (365 / 12)
  },
  "storage_requirements": {
    "document_storage_gb": (100 * 5MB) * (365 / 12),
    "chat_history_storage_gb": (50 * 1KB) * (365 / 12),
    "model_data_storage_gb": 5GB
  },
  "additional_services": {
    "db_access_cost_per_month": 1000 * $0.004,
    "sagemaker_training_cost_per_month": 10 hours/week * $0.3/hour
  }
}
```

**Flow:**

1. Define Key Components
2. API Call Costs
3. Storage Requirements
4. Additional Services
5. Document Storage
6. Chat History Storage
7. Database Access
8. AI Training
9. 1.8GB/year or 0.15GB/month
10. 3.6GB/year or 0.3GB/month
11. $2/day * 30 days = $60/month
12. $30/month training cost
13. Document Storage Cost
14. Chat History Storage Cost
15. Database Access Cost
16. AI Training Cost
17. Total Monthly Storage Cost
18. Total Monthly Cost~$270/month

