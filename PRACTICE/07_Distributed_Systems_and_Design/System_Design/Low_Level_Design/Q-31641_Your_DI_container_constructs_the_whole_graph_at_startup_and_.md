---
type: "practice"
difficulty: "Deep"
---

# Your DI container constructs the whole graph at startup and boot time is now 40 seconds; how do you redesign for lazy resolution while still catching wiring errors early—what's the trade-off?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Design Pattern Governance](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Over-engineering_vs_Simplicity/Design_Pattern_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a DI (Dependency Injection) container that constructs the entire graph at startup, one of the main challenges is balancing performance and resource management against ensuring correct dependencies are wired up without runtime errors. In this scenario, we need to redesign our approach for lazy resolution while maintaining early detection of wiring errors. This is crucial because it allows us to delay costly construction until absolutely necessary, improving initial boot time and overall system responsiveness.

#### Triage

The question is essentially asking how to transition from eager initialization (where all dependencies are resolved upfront) to lazy initialization (where dependencies are only created when needed). The core challenge here is ensuring that the wiring of these dependencies does not introduce runtime issues. We need to balance this with the benefit of reduced initial boot times.

#### Assumptions
The current system uses a DI container that eagerly initializes all components at startup.
The boot time is now 40 seconds, which needs optimization.
Lazy initialization must be implemented without compromising on error detection during development and testing phases.
We need to consider both the technical feasibility and business implications of the solution.

#### Solution
Step-by-Step Approach

Identify Components for Lazy Initialization: Determine which components or services can be safely delayed in their creation until a specific action is needed. Typically, these would be services that are not critical at startup but might be used later on during application execution.

Introduce Factory Methods and Optional Dependencies: For each lazy component, introduce factory methods or optional dependency injection mechanisms. These allow the DI container to defer resolution of dependencies until they are actually required.

Modify Boot Process for Lazy Resolution:

Implement a boot process that starts with minimal initialization.
Gradually resolve components on-demand as they are needed by other parts of the application.
Use event-driven or reactive patterns where applicable, so we can trigger dependency resolution based on specific events or actions.

Wiring and Error Detection:

Ensure that even in a lazy setup, wiring checks are still performed during startup to catch any misconfigurations early.
Implement custom error handling for cases where dependencies fail to initialize lazily (e.g., due to missing configuration).

Performance Monitoring and Optimization: Continuously monitor the performance impact of your changes. Use profiling tools to identify bottlenecks and optimize further.

Testing Strategy:

Maintain rigorous unit and integration testing to ensure that lazy components do not introduce runtime issues.
Utilize mocking frameworks during development to simulate lazy dependencies, ensuring correctness without actual lazy creation.

#### Closing Summary

By transitioning from eager to lazy initialization through strategic design changes, we can significantly reduce the boot time of our application. However, this approach comes with its own set of challenges and trade-offs, particularly around early error detection and complexity in component management. The key is to carefully identify which components can be deferred and ensure that our testing strategy remains robust.

#### Common Mistakes/Follow-Ups
Ignoring Early Wiring Errors: Failing to maintain thorough checks during the initial boot process could lead to runtime errors that are harder to diagnose.
Over-Lazy Initialization: Making too many components lazy might complicate the codebase and introduce unnecessary latency in certain scenarios.
Performance Overhead from Lazy Resolution: Even with optimized lazy resolution, there can be performance overhead associated with on-demand creation of dependencies.

**Comparison:**

```
Scenario	Eager Initialization	Lazy Initialization
Initial Boot Time	High (40+ seconds)	Reduced (initially low, increased on-demand)
Complexity	Low, easy to manage	Medium to high, more complex logic required
```

**Code:**

```python
# Example of a factory method for lazy initialization

class ServiceFactory:
    @staticmethod
    def create_service(config):
        # Perform validation and configuration checks here
        if config.get('enabled'):
            return ConcreteService()
        else:
            raise ValueError("Service is not enabled")

class ApplicationBootstrapper:
    def __init__(self, dependency_container):
        self.container = dependency_container

    def run(self):
        # Minimal initialization on boot
        self.container.add_service('service1', ServiceFactory.create_service)

        # Lazy resolution triggered by events or actions
        event_dispatcher.on_event('some_event', lambda: self.container.get('service1'))
```

**Example input / output:**

```text
Input:
- Start the application with a configuration file enabling the service.

Output:
- Initial boot time reduced due to lazy initialization.
- Service creation deferred until needed.

Request->Response Example:
- Event dispatcher triggers an event, causing the `ServiceFactory.create_service` method to be called and the service created at that moment.
```

**Data:**

```json
{
    "config": {
        "enabled": true
    }
}
```

**Flow:**

1. Yes
2. No
3. Start Application
4. Is Service Enabled?
5. Initialize Service
6. Throw Error or DelayInitialization
7. Lazily Resolve DependencyWhen Needed

