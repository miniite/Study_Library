---
type: "practice"
difficulty: "Intermediate"
---

# How would you prompt Cursor to implement error boundaries around risky UI regions and verify a thrown render error doesn't blank the whole app?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Agentic Coding with Cursor](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Agentic_Coding_with_Cursor.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this problem, we need to ensure that certain parts of our application's user interface (UI) are wrapped in an "error boundary" mechanism so that if an error occurs within these regions, it does not cause the entire application to crash or go blank. This is a crucial aspect of building robust and user-friendly applications.

#### Triaging the Question

The core question here is about implementing error handling for specific UI components using Cursor, a tool designed to assist with coding by suggesting code snippets. We need to understand how to properly structure our prompts and configurations in Cursor so that we can implement these error boundaries effectively.

#### Assumptions
Error Boundaries: These are React-specific mechanisms where you can catch JavaScript exceptions anywhere in the tree.
Cursor Integration: We assume that Cursor is integrated into a modern web application, likely using React or another framework that supports error boundary patterns.
Risky UI Regions: This could refer to components that handle user input, external APIs, or other parts of the UI where errors are more likely to occur.

#### First-Principles Solution

To solve this problem, we'll need to follow these steps:

Identify Risky Components: Determine which UI regions might throw a render error.
Wrap with Error Boundary: Use React's ErrorBoundary component to wrap the identified components.
Custom Error Handling: Define what happens when an error occurs within the boundary (e.g., logging, showing a custom error message).
Verify Functionality: Test that the application does not crash and that errors are handled gracefully.
Step-by-Step Implementation

Identify Risky Components: Suppose we have a form component where users can input data and submit it to an external API. This is a common source of errors (e.g., network issues, invalid data).

Wrap with Error Boundary:

JSX
import React, { Component } from 'react';
import axios from 'axios';

class RiskyForm extends Component {
  state = {
    error: null,
    isRenderingError: false,
  };

  componentDidCatch(error, info) {
    // Log the error and stack trace.

#### cons
ole.log('Logging Error', error);
    this.setState({
      error: error,
      isRenderingError: true,
    });
  }

  handleFormSubmit = async (event) => {
    try {
      event.preventDefault();

#### cons
t response = await axios.post('/api/data', { formInputs: event.target });

#### cons
ole.log('Success!', response.data);
    } catch (err) {
      // Handle the error here, but don't throw it.

#### cons
ole.error('Request failed with:', err.response.status);
    }
  };

  render() {
    if (this.state.isRenderingError) {
      return <div>Error: {this.state.error.message}</div>;
    }

    return (
      <form onSubmit={this.handleFormSubmit}>
        {/* Form fields go here */}
        <button type="submit">Submit</button>
      </form>
    );
  }
}

// Wrap the RiskyForm component with ErrorBoundary

#### cons
t App = () => {
  return (
    <ErrorBoundary fallback={<div>Oops, something went wrong!</div>}>
      <RiskyForm />
    </ErrorBoundary>
  );
};

Custom Error Handling: The componentDidCatch method logs the error and sets a state variable to indicate that an error has occurred, which is used in rendering to show a custom error message.

Verify Functionality: Test the application by simulating errors (e.g., network failure during API requests) to ensure that only the specific component fails gracefully, while the rest of the app remains functional.

#### Closing Summary

By using error boundaries and proper handling mechanisms, we can isolate potentially problematic areas in our UI and prevent them from causing a full application crash. This ensures a better user experience and easier debugging.

#### Common Mistakes/Follow-Ups
Not Wrapping All Risky Components: Make sure to identify all potential sources of errors.
Ignoring the  componentDidCatch Method: Properly logging and handling errors can help in diagnosing issues more effectively.
Failing to Test Thoroughly: It's important to test error scenarios rigorously to ensure comprehensive coverage.

**Code:**

```
JSX
Copy
import React, { Component } from 'react';
import axios from 'axios';

class RiskyForm extends Component {
  state = {
    error: null,
    isRenderingError: false,
  };

  componentDidCatch(error, info) {
    console.log('Logging Error', error);
    this.setState({
      error: error,
      isRenderingError: true,
    });
  }

  handleFormSubmit = async (event) => {
    try {
      event.preventDefault();
      const response = await axios.post('/api/data', { formInputs: event.target });
      console.log('Success!', response.data);
    } catch (err) {
      console.error('Request failed with:', err.response.status);
    }
  };

  render() {
    if (this.state.isRenderingError) {
      return <div>Error: {this.state.error.message}</div>;
    }

    return (
      <form onSubmit={this.handleFormSubmit}>
        {/* Form fields go here */}
        <button type="submit">Submit</button>
      </form>
    );
  }
}

const App = () => {
  return (
    <ErrorBoundary fallback={<div>Oops, something went wrong!</div>}>
      <RiskyForm />
    </ErrorBoundary>
  );
};
```

