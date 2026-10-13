---
type: "practice"
difficulty: "Intermediate"
---

# Copilot suggests embedding user queries and documents together in one API call. Why might you want them separate, and how would you verify the split?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reviewing AI Code for Correctness Bugs](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Correctness_Bugs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using AI-assisted coding tools like Copilot, it is often suggested to embed both user queries and documents in a single API call. However, there are scenarios where separating these elements might be beneficial. This separation can improve modularity, reduce complexity, and enhance security. In this explanation, I'll break down why we might want them separate, explore the trade-offs involved, and outline how to verify such a split.

Firstly, let's understand the scenario: typically, user queries are generated from the IDE or development environment, while documents (like source code files) reside on the local machine. Combining these in one API call can simplify implementation but might not always be the most efficient or secure approach.

Why Might You Want Them Separate?

Modularity and Reusability: By separating user queries and document uploads, you can make your system more modular. For instance, the query handling could be abstracted into a separate module that interacts with multiple APIs, while document management remains localized to the development environment.

Security and Privacy: Handling sensitive documents separately from user queries can enhance security by reducing the risk of exposing private information during transmission. This separation ensures that only necessary data is sent over the network, which is crucial in environments where data privacy is paramount.

Error Handling and Logging: Separating concerns allows for better error handling and logging mechanisms. If something goes wrong with document processing or query execution, you can diagnose issues more effectively by isolating them.

Scalability: In a distributed system, separating responsibilities means that different services can scale independently. For example, the backend service responsible for document storage might handle millions of files, while the frontend handles user queries.

How to Verify the Split

Integration Testing: Write comprehensive test cases that cover both scenarios—separated and combined. Ensure that when you split them, the functionality remains intact.

Performance Analysis: Measure performance metrics like latency and throughput for both approaches. This will help identify any bottlenecks or inefficiencies introduced by separation.

Security Audits: Conduct regular security audits to ensure that no sensitive information is accidentally exposed due to poor implementation practices.

Logging and Monitoring: Implement robust logging and monitoring mechanisms to track the flow of data and detect any anomalies early on.

User Feedback: Gather feedback from developers using the system to understand their preferences and pain points related to query-document separation.

#### Common Mistakes
and Follow-Ups

Ignoring Security Considerations: A common mistake is overlooking security when separating elements, leading to vulnerabilities.

Overcomplicating Simplicity: Sometimes, combining everything in one call can be simpler for implementation. It’s important not to over-engineer without clear benefits.

Summary

Separating user queries and documents in API calls offers several advantages, including better modularity, enhanced security, improved error handling, and scalability. To verify this separation effectively, you should conduct thorough integration testing, performance analysis, security audits, logging, and gather user feedback. By doing so, you can ensure that your system remains robust, secure, and efficient.

