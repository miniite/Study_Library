---
type: "practice"
difficulty: "Deep"
---

# How would you use the assistant to add graceful degradation when a downstream returns partial data, so the endpoint returns a usable response with a clear partial-failure signal rather than a 500?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompting for Production Patterns](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with an AI-assisted coding scenario where downstream services may return partial data, it’s crucial to ensure that our endpoint gracefully handles this situation and returns a usable response with clear signaling of partial failure. This involves intercepting incomplete responses from the downstream service and presenting them as much as possible while also providing useful error information.

The core idea is to leverage the assistant to design an architecture where we can intercept, validate, and partially process data before returning it to the client. By doing so, we maintain a high level of service availability even when some services are experiencing issues.

#### Triaging the Question

This question primarily probes our ability to handle partial failures gracefully in a distributed system. It also assesses our understanding of error propagation, response composition, and the use of design patterns like circuit breakers or fallback mechanisms. We need to make assumptions about how data is structured and what constitutes "usable" versus "incomplete."

#### First-Principles Solution
Interception Layer: Implement an interception layer that wraps around the downstream service call.
Data Validation & Processing: Validate the incoming partial data and process as much of it as possible.
Error Handling & Fallbacks: Provide clear error signals to the client indicating which parts were successfully processed versus those where failure occurred.
Step-by-Step Solution
Define Data Schema:
Assume we are working with JSON responses from a downstream service, e.g., {"id": 123, "name": "Product A", "price": 49.99, "description": null}.
Interception Layer Code (in Python):
PYTHON
import json

class PartialDataHandler:
    def __init__(self, downstream_service):
        self.downstream_service = downstream_service

    def fetch_data(self, request_id):
        try:
            # Fetch data from the downstream service
            raw_data = self.downstream_service.get(request_id)
            if not raw_data or 'id' not in raw_data:
                return {'error': f'Missing required fields'}

            parsed_data = json.loads(raw_data)

            # Process and validate partial data
            processed_data = {
                'id': parsed_data['id'],
                'name': parsed_data.get('name', ''),
                'price': float(parsed_data.get('price', 0.0)),
                'description': parsed_data.get('description', 'Unknown')
            }

            return {'status': 'success', 'data': processed_data}
        except Exception as e:
            # Handle unexpected errors and provide fallback
            return {'error': str(e)}
Downstream Service Response Handling (Example):
If the downstream service returns {"id": 123, "name": null}, our handler processes it to ensure that the response is usable.
Client-Side Error Handling: On the client side, we can implement error handling logic to distinguish between missing fields and complete responses.

#### Closing Summary

By implementing this interception layer with clear error signaling, we ensure that downstream partial failures do not result in a full 500 Internal Server Error response. Instead, our endpoint returns a usable response while clearly indicating which parts were successful and which had issues. This approach enhances the reliability of our service without compromising on user experience.

#### Common Mistakes
& Follow-Ups
Overly Complex Handling: Some might overcomplicate the error handling logic, leading to overly verbose code.
Ignoring Partial Successes: Not providing useful information for partial successes can lead to confusion in debugging and troubleshooting.
Assuming Full Control Over Downstream Services: Assuming full control or predictability of downstream services without considering external factors.

