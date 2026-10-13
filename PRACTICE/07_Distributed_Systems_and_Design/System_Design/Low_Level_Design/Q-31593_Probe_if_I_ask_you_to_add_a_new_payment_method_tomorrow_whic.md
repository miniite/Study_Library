---
type: "practice"
difficulty: "Intermediate"
---

# Probe: if I ask you to add a new payment method tomorrow, which files change in your design and why so few?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When tasked with adding a new payment method, it's crucial to understand the low-level design (LLD) of the system to minimize changes while ensuring functionality. The goal is to make the solution robust and modular, allowing easy integration without significant impact on other components.

#### Triaging the Question

The question asks you to identify which files would change when adding a new payment method. This probes your understanding of how systems are structured and whether the design allows for flexibility in changes. It also tests your ability to reason from first principles about how different parts of the system interact with each other.

#### Assumptions
The existing system supports multiple payment methods.
Payment logic is well-segregated from business logic.
Codebase follows best practices like modularization and separation of concerns.
Step-by-Step Solution

Identify the Components: In a typical e-commerce application, the core components related to payments are likely to include:

Models (e.g., PaymentMethod, Order)
Services (PaymentService for handling payment logic)
Repositories/Storage (PaymentRepository for storing methods and orders)
Controllers (for API endpoints)

Model Update: Adding a new payment method would require updating the PaymentMethod model to include additional fields like type, details, etc. For example:

PYTHON
class PaymentMethod(models.Model):
    type = models.CharField(max_length=50)  # e.g., "credit_card", "paypal"
    details = models.JSONField()            # JSON field for storing method-specific data

Service Layer: The PaymentService will need to be updated to handle the new payment method logic. This involves adding a case in the switch statement or if-else block that handles different payment types.

Repository/Storage: If you're using an ORM, updating the PaymentRepository might not change significantly unless there are additional fields needing storage support.

Controller Layer: The API endpoints for creating and managing orders would need to be updated to accept the new payment method. For example:

PYTHON
@api_view(['POST'])
def create_order(request):
    # ... existing code ...
    if request.data['payment_method'] == 'new_type':
        process_new_payment_method()
    else:
        process_standard_payment()
    # ... existing code ...

Configuration and Constants: If the new payment method requires any configuration or constants (e.g., API keys, fees), these would be added to a configuration file.

#### Closing Summary

In summary, adding a new payment method primarily affects the PaymentMethod model, the PaymentService, and possibly the controller layer. A modular design ensures that changes in one component don't significantly impact others, maintaining system stability and reducing integration issues.

#### Common Mistakes
and Follow-Ups
Ignoring Business Logic: Ensure business logic related to payments is well-segregated from other components.
Overcomplicating Changes: Try to minimize changes by adhering to modular design principles.
Lack of Test Coverage: Verify that the new payment method works correctly with existing tests.

In follow-up questions, the interviewer might ask about test cases for the new payment method or how you ensure backward compatibility during the update process.

