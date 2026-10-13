---
type: "practice"
difficulty: "Intermediate"
---

# Your team's DTOs and entities have drifted so much that mapping code is a bug source; how do you decide where mapping lives, and would you introduce a mapper library or hand-write it—defend the trade-off.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DTOs and Domain Models](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/DTOs_and_Domain_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with DTOs (Data Transfer Objects) and entities that have drifted significantly, the core issue is to maintain synchronization between these models while minimizing bugs in mapping code. The decision on where and how to handle this mapping involves evaluating trade-offs such as development speed, maintainability, and future extensibility.

To start, let's clarify what the question is really probing:

Where should mapping logic live? (Entity classes, DTOs, or a separate layer)
Should we use a library for handling these mappings?
What are the trade-offs involved in each approach?

#### Assumptions
The system is complex enough to require both DTOs and entities.
There have been significant changes in either model that necessitate re-mapping logic.
We need to ensure this mapping does not introduce bugs or performance issues.
Step-by-Step Solution
Option 1: Hand-Writing Mappers

Hand-writing mappers involves manually defining methods or functions to convert between entities and DTOs. This approach has the advantage of direct control over how data is transformed, which can be crucial in complex systems where subtle transformations are needed.

#### Pros

Fine-grained Control: Direct access allows for precise handling of edge cases.
Minimal Overhead: No additional libraries or dependencies to maintain.

#### Cons

Maintenance Burden: Every time the structure of entities or DTOs changes, you need to update the mappers, which can be error-prone and time-consuming.
Code Duplication: If multiple places require similar mappings, there may be duplication of code, leading to inconsistency.
Option 2: Introducing a Mapper Library

A mapper library automates much of the mapping process. Libraries like MapStruct in Java or AutoMapper in .NET provide annotations that map fields directly from source to target objects. This approach can significantly reduce development time and minimize errors due to manual coding.

#### Pros

Automated Mappings: Reduces manual effort by automatically generating mappings.

#### Cons
istency: Ensures consistent mapping logic across the system, reducing bugs.
Easier Maintenance: Changes in DTO or entity structures are reflected automatically without needing to update each mapper manually.

#### Cons

Overhead of Dependency Management: Adding a new library introduces additional dependencies that need to be managed.
Limited Customization: Some complex mappings may require custom logic, which can complicate the use of a generic mapper library.
Choosing Between Options

The decision between hand-writing mappers and using a library depends on several factors:

Complexity of Mappings: If mappings are simple and infrequent changes are expected, hand-coding might be sufficient.
Rate of Change: Frequent changes in the DTO or entity models would benefit from a more automated solution to reduce manual overhead.
Project Size and Team Experience: Larger projects with many teams collaborating on shared codebases often benefit from tools that enforce consistency.

#### Conclusion

For an intermediate experience level, it’s crucial to balance between control and maintainability. In most cases, introducing a mapper library like MapStruct or AutoMapper is recommended because it reduces the likelihood of bugs and ensures consistent mapping across the system. However, for highly complex projects with rapidly changing models, hand-coding mappers might be necessary to maintain precise control over transformations.

#### Common Mistakes
and Follow-Ups
Ignoring Library Overhead: Not considering the additional dependencies and potential complexity they bring.
Assuming Simple Mappings Are Enough: Assuming that simple mappings can handle all use cases without requiring any custom logic, leading to suboptimal solutions.
Failing to Consider Future Changes: Not accounting for the need to adapt mappings as the system evolves.

**Code:**

```

// Example of a hand-written mapper in Java public class UserMapper { public static UserDto toUserDto(User user) { return new UserDto( user.getId(), user.getName(), user.getEmail() ); } }

CODE
Copy

 ```mermaid
graph TD;
A[Entity] --> B[DTO]
B --> C[Database]
A --> D[Mapper Code]
D --> C
```

