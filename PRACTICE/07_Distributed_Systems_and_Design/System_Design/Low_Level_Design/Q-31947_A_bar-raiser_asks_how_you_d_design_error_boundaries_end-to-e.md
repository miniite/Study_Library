---
type: "practice"
difficulty: "Deep"
---

# A bar-raiser asks how you'd design 'error boundaries end-to-end' so a typed backend error maps to a specific frontend UI treatment (retryable toast vs field error vs full-page) through a stable contract; how do you stop the mapping from becoming a giant switch that every screen re-writes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ DTOs and Domain Models](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/DTOs_and_Domain_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing 'error boundaries end-to-end' in a large-scale application, the goal is to ensure that error handling is consistent and maintainable across all layers of the system. This involves mapping typed backend errors to specific frontend UI treatments such as retryable toasts, field errors, or full-page errors. The challenge lies in preventing this mapping from becoming unwieldy—a giant switch statement scattered throughout various screens. Here's a detailed approach:

TL;DR

We'll design a robust error handling system using domain events and a central error boundary component that maps backend errors to frontend treatments without the need for screen-specific logic. This ensures consistency, maintainability, and scalability.

#### Triaging the Question

The interviewer is probing how you handle complex, distributed systems with strict requirements on error reporting and user experience. They want to see your ability to design a system that is both robust and scalable while maintaining clear separation of concerns.

#### Assumptions
The backend uses a typed API (e.g., GraphQL or REST).
Frontend components can be updated without changing the backend.
We need to handle errors consistently across multiple screens and user interactions.
Error severity and type are well-defined in both frontend and backend.
Step-by-Step Solution
1. Define Error Types

First, define a set of error types on the backend that correspond to different severity levels (e.g., Retryable, FieldError, FullPageError). This ensures consistency across the application.

JSON
{
  "errorTypes": {
    "RETRYABLE": {
      "name": "Retryable",
      "uiTreatment": "retryableToast"
    },
    "FIELD_ERROR": {
      "name": "Field Error",
      "uiTreatment": "fieldError"
    },
    "FULL_PAGE_ERROR": {
      "name": "Full Page Error",
      "uiTreatment": "fullPageError"
    }
  }
}
2. Use Domain Events

Domain events provide a clean way to handle errors without the need for conditional logic in every component. We can use an event-driven architecture where each backend service publishes specific error events.

ServiceA
Publishes error events
Centralizes error handling andmapping
3. Centralized Error Boundary Component

Implement a central CentralErrorBoundary component that subscribes to these domain events. This component will map backend errors to frontend treatments based on predefined rules.

JSX
function CentralErrorBoundary({ children }) {

#### cons
t [error, setError] = useState(null);

    useEffect(() => {
        // Subscribe to error events from the backend

#### cons
t eventSubscription = ErrorEventPublisher.onError(eventHandler);

        return () => {
            eventSubscription.unsubscribe();
        };
    }, []);

    function eventHandler(errorData) {
        switch (errorData.type) {
            case "RETRYABLE":
                setError({
                    type: "retryableToast",
                    message: errorData.message
                });
                break;
            case "FIELD_ERROR":
                setError({
                    type: "fieldError",
                    field: errorData.field,
                    message: errorData.message
                });
                break;
            case "FULL_PAGE_ERROR":
                setError({
                    type: "fullPageError",
                    message: errorData.message
                });
                break;
        }
    }

    if (error) {
        return <ErrorBoundaryComponent {...error} />;
    }

    return children;
}

function ErrorBoundaryComponent({ type, field = "", message }) {
    switch (type) {
        case "retryableToast":
            return <RetryToast message={message} />;
        case "fieldError":
            return <FieldError field={field} message={message} />;
        case "fullPageError":
            return <FullPageError message={message} />;
        default:
            return null;
    }
}
4. Use Error Types in Frontend Components

Ensure that frontend components can handle the error types without complex logic. For example, a form component should only need to check if there are FIELD_ERROR events.

JSX
function FormComponent({ onSubmit }) {

#### cons
t [errors, setErrors] = useState([]);

    useEffect(() => {
        // Listen for field errors and update state
        CentralErrorBoundary.onError((event) => {
            if (event.type === "FIELD_ERROR") {
                setErrors([{ field: event.field, message: event.message }]);
            }
        });
    }, []);

    function handleSubmit(event) {
        event.preventDefault();
        onSubmit();
    }

    return (
        <form onSubmit={handleSubmit}>
            {errors.map((error) => (
                <FieldError key={error.field} field={error.field} message={error.message} />
            ))}
            {/* Other form elements */}
        </form>
    );
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By using domain events and a centralized error boundary component, we ensure that the frontend remains decoupled from backend logic. This approach scales well as new screens are added or existing ones modified. The key takeaway is to avoid tightly coupling UI components with backend error handling.

#### Common Mistakes

Not centralizing error handling leads to duplicated code and inconsistent behavior.
Failing to define clear error types can result in ambiguous mapping rules.
Ignoring the need for event-driven architecture results in tightly coupled components that are harder to maintain.

#### Follow-Ups

How would you handle errors in microservices architectures?
What additional logging and monitoring mechanisms would you implement?

**Comparison:**

```
2024-01-01
2024-04-01
2024-07-01
2024-10-01
2025-01-01
2025-04-01
2025-07-01
2025-10-01
2026-01-01
2026-04-01
2026-07-01
2026-10-01
BackendErrorEvent
EventSubscription
EventHandling
ErrorMapping
Domain Events
CentralErrorBoundary
ErrorBoundaryComponent
Centralized Error Handling Workflow
```

**Code:**

```

function CentralErrorBoundary({ children }) { const [error, setError] = useState(null);

CODE
Copy
useEffect(() => {
    // Subscribe to error events from the backend
    const eventSubscription = ErrorEventPublisher.onError(eventHandler);

    return () => {
        eventSubscription.unsubscribe();
    };
}, []);

function eventHandler(errorData) {
    switch (errorData.type) {
        case "RETRYABLE":
            setError({
                type: "retryableToast",
                message: errorData.message
            });
            break;
        case "FIELD_ERROR":
            setError({
                type: "fieldError",
                field: errorData.field,
                message: errorData.message
            });
            break;
        case "FULL_PAGE_ERROR":
            setError({
                type: "fullPageError",
                message: errorData.message
            });
            break;
    }
}

if (error) {
    return <ErrorBoundaryComponent {...error} />;
}

return children;

}

function ErrorBoundaryComponent({ type, field = "", message }) { switch (type) { case "retryableToast": return <RetryToast message={message} />; case "fieldError": return <FieldError field={field} message={message} />; case "fullPageError": return <FullPageError message={message} />; default: return null; } }

CODE
Copy

```jsx
function FormComponent({ onSubmit }) {
    const [errors, setErrors] = useState([]);

    useEffect(() => {
        // Listen for field errors and update state
        CentralErrorBoundary.onError((event) => {
            if (event.type === "FIELD_ERROR") {
                setErrors([{ field: event.field, message: event.message }]);
            }
        });
    }, []);

    function handleSubmit(event) {
        event.preventDefault();
        onSubmit();
    }

    return (
        <form onSubmit={handleSubmit}>
            {errors.map((error) => (
                <FieldError key={error.field} field={error.field} message={error.message} />
            ))}
            {/* Other form elements */}
        </form>
    );
}
```

**Example input / output:**

```text
// Sample Input (Backend Event)
{
  "type": "RETRYABLE",
  "message": "Server is busy, please try again later."
}

// Output (Frontend Error Treatment)
<RetryToast message="Server is busy, please try again later." />
```

**Data:**

```json
{
  "errorTypes": {
    "RETRYABLE": {
      "name": "Retryable",
      "uiTreatment": "retryableToast"
    },
    "FIELD_ERROR": {
      "name": "Field Error",
      "uiTreatment": "fieldError"
    },
    "FULL_PAGE_ERROR": {
      "name": "Full Page Error",
      "uiTreatment": "fullPageError"
    }
  }
}
```

**Flow:**

1. Publishes error event
2. Maps to UI treatment
3. ServiceA
4. ErrorEventPublisher
5. CentralErrorBoundary listensand handles events
6. ErrorBoundaryComponentrenders appropriate UIcomponent

