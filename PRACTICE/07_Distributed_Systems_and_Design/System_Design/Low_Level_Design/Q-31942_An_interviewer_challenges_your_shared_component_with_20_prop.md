---
type: "practice"
difficulty: "Deep"
---

# An interviewer challenges your 'shared component with 20 props' data table used by 12 teams; how do you decide between compound components (Table.Row, Table.Cell), render props, and a config object so teams compose behavior without a prop explosion or a config-soup god component?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Design Tokens and Theming](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Design_Tokens_and_Theming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a complex data table shared among 12 teams, deciding between compound components (Table.Row, Table.Cell), render props, and a config object is crucial to maintain clean, scalable code without causing a "prop explosion" or creating an overly complex "config-soup" component. This decision should be guided by the principles of separation of concerns, reusability, and maintainability.

Firstly, let's consider the core problem: managing 20 props in a single data table used across multiple teams. This can quickly lead to prop explosion, where each team starts adding their own unique logic or styles, making the component harder to maintain and understand.

Compound Components (Table.Row, Table.Cell)

Key Idea: Compound components break down the larger component into smaller, reusable parts that encapsulate specific responsibilities.
Benefits:
Modularity: Each row and cell can be independently tested and updated without affecting others.
Maintainability: Smaller, more focused components are easier to understand and maintain.
Reusability: These components can be reused across different tables or even in other parts of the application.

#### Trade-offs
Complexity: Increased complexity due to additional component layers, which might affect performance if not optimized.

Render Props

Key Idea: Render props allow a component to pass down a render function as a prop, enabling it to control how child components are rendered.
Benefits:
Flexibility: Teams can define their own rendering logic within the provided props, leading to more tailored user interfaces.
Encapsulation: The core table component remains simpler and cleaner while allowing for complex customizations.

#### Trade-offs
Complexity: Render props can introduce additional complexity if not used correctly, requiring teams to understand and handle functions as a prop.

Config Object

Key Idea: A config object allows teams to pass in configuration data that controls how the table behaves without affecting its structure.
Benefits:
Simplicity: Teams can modify behavior by changing values within the config object, keeping the core component simple and clean.
Flexibility: The same base component can be reused with different configurations for various use cases.

#### Trade-offs
Clarity: Config objects can become large and unwieldy if not well-structured, leading to "config-soup" where it’s hard to understand what each setting does.
Triaging the Problem

In this scenario, we need to balance between modularity, reusability, and flexibility. The goal is to keep the core table component clean while allowing teams to add their own logic without causing a prop explosion or creating overly complex config objects.

Initial Design: Compound Components (Table.Row, Table.Cell)
Step-by-Step Solution:
Define Table as the top-level component.
Create Row and Cell components that encapsulate rendering logic for individual rows and cells.
Use context or props to pass necessary data (e.g., table headers, row data) down to these components.
Ensure each component has a single responsibility and is reusable.
JSX
// Example of Table.Row Component

#### cons
t Row = ({ rowData }) => {
  return (
    <tr>
      {rowData.map((cellData, index) => (
        <Cell key={index} cellData={cellData} />
      ))}
    </tr>
  );
};
Integrating Render Props
Step-by-Step Solution:
Modify the Table component to accept a renderRow prop that returns a React element.
Pass down necessary data and props to this render function.
JSX
// Example of Table with Render Props

#### cons
t Table = ({ rows, columns }) => {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={column.id}>{column.title}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {rows.map((row, rowIndex) => (
          <tr key={rowIndex}>
            {row.cells.map((cellData, cellIndex) => {

#### cons
t renderCell = (data) => columns[cellIndex].render(data);
              return <Cell key={cellIndex} cellData={cellData} renderCell={renderCell} />;
            })}
          </tr>
        ))}
      </tbody>
    </table>
  );
};
Using Config Objects
Step-by-Step Solution:
Define a TableConfig object that contains all the necessary settings for behavior and styling.
Pass this config down as props to the Table component.
JSX
// Example of Table with Config Object

#### cons
t Table = ({ rows, columns, config }) => {
  return (
    <table>
      {/* Render headers */}
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={column.id}>{config.headerConfig[column.id]}</th>
          ))}
        </tr>
      </thead>
      {/* Render rows and cells using config settings */}
      ...
    </table>
  );
};

#### Closing Summary

By choosing a combination of compound components, render props, and a well-structured config object, we can achieve a clean, maintainable, and flexible data table component that meets the needs of multiple teams. This approach ensures that each team can add their own logic without causing prop explosion or creating an overly complex configuration.

#### Common Mistakes/Follow-Ups
Prop Explosion: Ensuring that the number of props passed to components is kept minimal.
Config Soup: Avoiding overly large and unstructured config objects by organizing them in a logical manner.
Complexity Management: Striking a balance between simplicity and functionality, ensuring that each component has a single responsibility.

**Comparison:**

```
Component	Purpose
Table.Row	Encapsulates rendering logic for individual rows.
Table.Cell	Encapsulates rendering logic for individual cells.
Render Props	Allows passing a render function to control how child components are rendered.
Config Object	Contains configuration settings for behavior and styling.
```

**Code:**

```
JSX
Copy
// Example of Table.Row Component
const Row = ({ rowData }) => {
  return (
    <tr>
      {rowData.map((cellData, index) => (
        <Cell key={index} cellData={cellData} />
      ))}
    </tr>
  );
};

// Example of Table with Render Props
const Table = ({ rows, columns }) => {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={column.id}>{column.title}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {rows.map((row, rowIndex) => (
          <tr key={rowIndex}>
            {row.cells.map((cellData, cellIndex) => {
              const renderCell = (data) => columns[cellIndex].render(data);
              return <Cell key={cellIndex} cellData={cellData} renderCell={renderCell} />;
            })}
          </tr>
        ))}
      </tbody>
    </table>
  );
};

// Example of Table with Config Object
const Table = ({ rows, columns, config }) => {
  return (
    <table>
      {/* Render headers */}
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={column.id}>{config.headerConfig[column.id]}</th>
          ))}
        </tr>
      </thead>
      {/* Render rows and cells using config settings */}
      ...
    </table>
  );
};
```

**Example input / output:**

```text
// Example of rendering the Table component with props and config
<Table
  rows={rows}
  columns={columns}
  config={{
    headerConfig: { id: "ID", name: "Name", value: "Value" },
    renderCell: (data) => data.render(data)
  }}
/>
```

**Data:**

```json
{
  "columns": [
    { "id": "id", "title": "ID" },
    { "id": "name", "title": "Name" },
    { "id": "value", "title": "Value" }
  ],
  "rows": [
    { "cells": [{ "id": "1", "name": "John Doe", "value": 100 }]},
    { "cells": [{ "id": "2", "name": "Jane Smith", "value": 200 }]}
  ]
}
```

**Flow:**

1. Define Table Component
2. Compound Components
3. Table.Row, Table.Cell
4. Modularity
5. Maintainability
6. Render Props
7. ([Flexibility])
8. Encapsulation
9. Config Object
10. ([Simplicity])
11. Flexibility

