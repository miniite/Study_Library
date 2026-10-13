---
type: "practice"
difficulty: "Deep"
---

# You're deciding the org's approach to modeling 'commands' as first-class objects (validated, serializable, auditable) versus plain method calls with arguments; what does command-as-object buy you for retry, audit, and async, and when is it just ceremony over a method?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When deciding between modeling 'commands' as first-class objects versus plain method calls with arguments in your application architecture, command-as-object models offer significant benefits in terms of retry, auditability, and asynchronous processing. However, they can also introduce complexity and overhead. This decision hinges on the specific use case, project requirements, and architectural trade-offs.

#### Triage

The question is essentially probing whether using a "command" as an object versus plain method calls offers distinct advantages or if it's just additional ceremony. We need to consider the context of retry mechanisms, audit trails, and asynchronous processing while also acknowledging potential downsides such as increased complexity and performance overhead.

#### Assumptions
Retry Mechanism: The system needs robust error handling and retries.
Auditability: The application requires detailed logging or traceability for compliance or debugging purposes.
Asynchronous Processing: There is a need to handle operations in an asynchronous manner without blocking the main thread.

#### First-Principles Solution
Command-as-Object Model:

1. Retry Mechanism: Commands as objects can include retry logic baked into their execution flow. Each command can contain a retryStrategy that defines how and when to attempt re-execution. This encapsulates the retry behavior within the command itself, making it easier to manage retries across different components.

Yes
Failure
Success
Command Object
Retry?
Execute Command
Check for Error
Retry Logic
Execute Command
Complete

2. Auditability: Each command object can include fields like commandId, createdAt, and updatedAt to track its lifecycle. Additionally, logs or audits can be generated whenever a command is created, executed, or updated. This provides a clear audit trail that helps in debugging and compliance.

JAVASCRIPT
class Command {

#### cons
tructor(commandType, data) {
        this.commandId = generateUUID();
        this.commandType = commandType;
        this.data = data;
        this.createdAt = new Date().toISOString();
    }

    execute() {
        // Execute the command logic

#### cons
t success = performOperation(this.data);
        if (success) {
            this.updatedAt = new Date().toISOString();
            return true;
        } else {
            throw new Error('Command execution failed');
        }
    }
}

3. Asynchronous Processing: Commands can be designed to handle asynchronous operations more gracefully. They can use futures, promises, or other concurrency constructs to manage state and results asynchronously. This makes it easier to decouple the command execution from the calling context.

JAVASCRIPT
class Command {
    // ...

    async executeAsync() {
        return new Promise((resolve, reject) => {

#### cons
t success = performOperation(this.data);
            if (success) {
                this.updatedAt = new Date().toISOString();
                resolve(true);
            } else {
                reject(new Error('Command execution failed'));
            }
        });
    }
}
Plain Method Call Model:

1. Retry Mechanism: Plain method calls can still implement retry logic, but it needs to be handled by the caller or a higher-level mechanism. This can make the code less encapsulated and harder to maintain.

JAVASCRIPT
function performOperation(data) {
    try {
        // Perform operation
    } catch (error) {
        if (retryCounter < maxRetries) {
            retryCounter++;
            return performOperation(data); // Recursive call
        }
        throw error;
    }
}

2. Auditability: Audit logs can be generated manually by the caller or through a centralized logging mechanism, but this is less systematic and harder to implement consistently.

JAVASCRIPT
function performOperation(data) {

#### cons
ole.log(`Performing operation with data ${data}`);
    try {
        // Perform operation
    } catch (error) {

#### cons
ole.error(`Failed to perform operation: ${error.message}`);
        throw error;
    }
}

3. Asynchronous Processing: Plain method calls can be adapted for async processing, but it requires additional boilerplate code and context management.

JAVASCRIPT
async function performOperationAsync(data) {
    try {
        await new Promise((resolve) => setTimeout(resolve, 1000)); // Simulate async delay
        // Perform operation
    } catch (error) {
        throw error;
    }
}

#### Closing Summary

Command-as-object models provide a structured approach to handling operations that offer better support for retry mechanisms, audit trails, and asynchronous processing. However, they introduce additional complexity and may not always be necessary depending on the specific requirements of the application.

#### Common Mistakes/Follow-Ups
Overuse: Command objects can sometimes be overused in situations where plain method calls would suffice.
Performance Overhead: The overhead of creating and managing command objects can impact performance, especially if commands are frequently created and destroyed.
Complexity: Adding too many features or properties to the command object can make it overly complex and difficult to maintain.
Triaging Follow-Ups:
Would you consider a hybrid approach where some operations use plain method calls and others use command objects?
How do you balance between robustness (retry, audit) and simplicity in your application architecture?

**Comparison:**

```
Command Type	Data Fields
CREATE_USER	username, email
UPDATE_PROFILE	userId, bio
```

**Code:**

```javascript
class Command {
    constructor(commandType, data) {
        this.commandId = generateUUID();
        this.commandType = commandType;
        this.data = data;
        this.createdAt = new Date().toISOString();
    }

    async executeAsync() {
        return new Promise((resolve, reject) => {
            const success = performOperation(this.data);
            if (success) {
                this.updatedAt = new Date().toISOString();
                resolve(true);
            } else {
                reject(new Error('Command execution failed'));
            }
        });
    }
}
```

**Example input / output:**

```text
const command = new Command("CREATE_USER", { username: "john_doe", email: "johndoe@example.com" });
console.log("Executing command:", command);
try {
    await command.executeAsync();
    console.log("Command executed successfully.");
} catch (error) {
    console.error("Failed to execute command:", error.message);
}
```

**Data:**

```json
{
  "commandId": "123e4567-e89b-12d3-a456-426614174000",
  "commandType": "CREATE_USER",
  "data": {
    "username": "john_doe",
    "email": "johndoe@example.com"
  },
  "createdAt": "2023-10-01T15:00:00.000Z"
}
```

**Flow:**

1. Yes
2. Failure
3. Success
4. Command Object
5. Retry?
6. Execute Command
7. Check for Error
8. Retry Logic
9. Execute Command
10. Complete

