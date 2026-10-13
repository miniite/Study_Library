---
type: "practice"
difficulty: "Deep"
---

# Design an anti-corruption layer between your clean domain model and a messy third-party CRM API; where exactly do you translate, and how do you stop the CRM's naming and nullability from leaking inward?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective anti-corruption layer (ACL) between a clean domain model and a messy third-party CRM API, we need to ensure that the internal logic of our application remains decoupled from external inconsistencies like naming conventions and nullability issues. This involves translating data structures and ensuring the integrity and consistency of our domain model.

#### Triaging the Question

The core challenge is to isolate the clean domain model from the quirks of the third-party CRM API, preventing it from being polluted by messy external data representations. The interviewer is probing your ability to architect a system that maintains separation of concerns and handles complex transformations gracefully. We need to make several key assumptions:

The CRM API has known issues with its data structures.
Our domain model follows best practices for clean code.
We are designing this layer in a way that supports future changes without breaking existing functionality.
Step-by-Step Solution
Define the Boundary: Identify the boundary between our application's internal logic and the external CRM API. This boundary will be where we perform all translations, ensuring no direct interaction occurs.
Translation Layer Design: Create a translation layer that converts data from the CRM API format to the clean domain model format and vice versa.
Nullability Handling: Ensure that any null values or unexpected data types from the CRM are handled gracefully within the translation logic.
Naming Conventions: Rename fields as necessary to align with our internal standards, ensuring no naming conflicts.
Detailed Solution

Boundary Definition:

The translation layer will act as a service that interacts directly with the CRM API and converts its responses into data structures compatible with our domain model.
Similarly, when sending requests to the CRM, this layer will transform our clean domain model into the required format.

Translation Layer Design:

Implement a TranslationService class responsible for converting between the two formats.
Use DTOs (Data Transfer Objects) to represent data structures from both the CRM and the domain model.

Nullability Handling:

Use default values or sensible fallback strategies to handle null values in the CRM API response.
For example, if a field is null but should never be null, provide a default value based on business logic.

Naming Conventions:

Define a set of rules for renaming fields from the CRM API to our domain model.
Use mapping tables or decorators in the DTOs to automate these renamings.
Code Example
PYTHON
class CustomerDTO:
    def __init__(self, crm_customer_id: str, name: str = None, email: str = None):
        self.crm_customer_id = crm_customer_id
        self.name = name if name else "Unknown"
        self.email = email

class TranslationService:
    @staticmethod
    def map_crm_to_domain(crm_data: dict) -> CustomerDTO:
        mapped_data = {
            "crm_customer_id": crm_data["id"],
            "name": crm_data.get("name", "Unknown"),
            "email": crm_data.get("email")
        }
        return CustomerDTO(**mapped_data)

#### Closing Summary
and Common Mistakes
Common Mistake: Directly mapping CRM data to domain models without a translation layer can lead to brittle code that breaks when the CRM schema changes.
Key Insight: The anti-corruption layer ensures that the internal model remains clean, isolated from external inconsistencies.

By following this approach, we ensure that our application is robust and maintainable, even as third-party APIs evolve or change. This design also supports future enhancements and scalability.

