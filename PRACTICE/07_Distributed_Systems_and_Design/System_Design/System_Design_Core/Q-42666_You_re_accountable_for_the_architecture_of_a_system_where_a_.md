---
type: "practice"
difficulty: "Deep"
---

# You're accountable for the architecture of a system where a downstream partner deprecates an API you depend on with 90 days notice. Walk me through how you'd design the abstraction so such a forced migration is a config change, not a re-architecture.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a scenario where a downstream partner deprecates an API and gives 90 days notice, our goal as backend architects/principals is to design a system that can seamlessly migrate away from this dependency without requiring a complete re-architecture. This involves creating a robust abstraction layer that allows for easy configuration changes while maintaining high performance and reliability.

#### Triage

The key challenge here is designing an abstraction layer that decouples our application from the external API, enabling us to switch to alternative services or implementations with minimal code changes. The interviewer will be probing our ability to architect systems that are modular, flexible, and resilient. We need to consider factors like performance, failover strategies, and ease of maintenance.

Abstraction Design
Identify Dependencies: First, we need to identify all the components in our system that currently depend on this API.
Define Interfaces: Define a well-defined interface or set of interfaces for the functionality provided by the deprecated API. This interface should be abstract enough to accommodate different implementations.
Implement Adapter Pattern: Use the adapter pattern to wrap the existing integration code with an interface, allowing us to swap out different implementations without changing the consuming code.
Configuration Management: Introduce a configuration layer that allows runtime selection of which implementation to use. This could be done using environment variables or a centralized configuration service.
Step-by-Step Solution

Deprecation Notice Handling:

Upon receiving deprecation notice, we schedule a task for 90 days from the notice date.
The task will trigger a series of steps to identify and prepare for the migration.

Code Analysis & Refactoring:

Perform static code analysis to locate all dependencies on the deprecated API.
For each dependency, create an adapter that implements the required functionality in terms of our new interface.

Implementing Adapters:

Develop a set of adapters for different potential replacement services or internal implementations.
Each adapter should be tested independently to ensure it meets the performance and reliability requirements.

Configuration Management:

Create a configuration management system that allows us to specify which adapter to use at runtime.
This could be as simple as environment variables in development, but should be extensible for production scenarios where multiple environments might require different configurations.

Testing & Validation:

Thoroughly test the new setup using existing integration tests and additional unit tests for the adapters.
Validate performance and stability under realistic load conditions.

Rollout Strategy:

Gradually roll out the new configuration in a staged manner to minimize impact on production systems.
Monitor closely during the transition period to ensure no regressions occur.

#### Closing Summary

By implementing this abstraction layer, we can handle the deprecation of an external API with ease. The use of adapters and dynamic configuration ensures that our system remains flexible and resilient, allowing us to seamlessly switch between different implementations without major rework. This approach not only minimizes downtime but also enhances the maintainability and scalability of our architecture.

#### Common Mistakes
& Follow-Ups
Over-engineering: Avoid complex solutions that are harder to manage in the long run.
Inadequate Testing: Ensure thorough testing before and after the migration to catch any potential issues early.
Ignoring Performance Impacts: While flexibility is important, it should not come at the cost of performance. Monitor closely during the transition.

**Comparison:**

```
Migration Step	Description
1	Schedule task for 90 days from now.
2	Perform static code analysis.
3	Develop adapters for new implementations.
4	Implement configuration management system.
5	Test all components thoroughly.
6	Roll out new configuration in a staged manner.
```

**Code:**

```python
# Example of an adapter pattern implementation in Python

class ExternalAPIAdapter:
    def __init__(self, config):
        self.config = config

    def get_data(self):
        # Placeholder for actual API call
        pass

    def set_timeout(self, timeout):
        pass

class InternalImplementationAdapter(ExternalAPIAdapter):
    def get_data(self):
        return internal_function()

# Example configuration in environment variables
API_IMPLEMENTATION = "internal"  # or "external"
TIMEOUT = 5  # seconds

if API_IMPLEMENTATION == "internal":
    adapter = InternalImplementationAdapter(config={"timeout": TIMEOUT})
else:
    adapter = ExternalAPIAdapter(config={"timeout": TIMEOUT})

data = adapter.get_data()
```

**Example input / output:**

```text
Input:
- API deprecation notice received

Output:
1. Schedule migration task for 90 days from now.
2. Begin static code analysis and identify dependencies.
3. Develop adapters for new implementations.
4. Implement configuration management system.
5. Test all components thoroughly.
6. Roll out new configuration in a staged manner.

Request -> Response:
- Initial request: API deprecation notice
- Final response: System migration complete, with minimal downtime and no code changes required.
```

**Data:**

```yaml
api_adapter_config:
  implementation: internal
  timeout: 5
```

**Flow:**

1. Receive Deprecation Notice
2. Schedule Migration Task
3. Code Analysis & Refactoring
4. Implement Adapters
5. Integrate Config Management
6. Test Components Thoroughly
7. Rollout Configuration

