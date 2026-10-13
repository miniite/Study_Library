---
type: "practice"
difficulty: "Intermediate"
---

# Design a component library's theming system with light/dark and tokens. What is the object model?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Design Tokens and Theming](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Design_Tokens_and_Theming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing a component library’s theming system that supports both light and dark themes, along with theme token management, we need to ensure a flexible and efficient structure that can be easily extended and maintained. The object model will consist of several key components including ThemeConfig, ThemeTokens, and ThemeComponents.

Firstly, the ThemeConfig is responsible for defining whether the application should use light or dark mode. This config would also store default values if needed and serve as a central point to switch between themes dynamically. Secondly, ThemeTokens will encapsulate all theme-related variables such as colors, typography, spacing, etc., making it easy to apply consistent styling across different components. Lastly, ThemeComponents act as the individual building blocks that can reference tokens from ThemeTokens.

To get a clearer picture of how these pieces fit together, let’s break down each component in detail:

ThemeConfig:

Purpose: To manage and switch between light and dark themes.
Properties:
mode: A string indicating the current theme mode ("light" or "dark").
switchFunction: A function to toggle between modes.

ThemeTokens:

Purpose: To store all theme-related constants that can be used across components.
Properties:
colors: An object containing primary, secondary, text, background colors, etc.
typography: An object defining font family, sizes, line height, etc.
spacings: An array or object for different spacing values.

ThemeComponents:

Purpose: To define components that can be styled using tokens from ThemeTokens.
Properties:
Each component would have a theme prop that references the current theme.
The actual styling would be done via CSS-in-JS or styled-components, leveraging the tokens.

Let’s illustrate this with an example:

JAVASCRIPT
// ThemeConfig.js
export const ThemeConfig = {
  mode: 'light',
  switchFunction: () => (/* logic to toggle modes */),
};

// ThemeTokens.js
export const ThemeTokens = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    text: '#343a40',
    background: '#fff',
  },
  typography: {
    fontFamily: 'Arial, sans-serif',
    fontSize: '16px',
    lineHeight: '28px',
  },
  spacings: [0, 8, 16, 24],
};

// Button.js (ThemeComponent)
import React from 'react';
import styled from 'styled-components';

#### cons
t { colors } = ThemeTokens;

export const Button = ({ theme, children }) => {
  return (
    <StyledButton color={theme.colors.primary}>
      {children}
    </StyledButton>
  );
};

#### cons
t StyledButton = styled.button`
  background-color: ${props => props.color};
  padding: 10px;
  border-radius: 5px;
`;

// Usage
import React from 'react';
import ThemeConfig, { switchFunction } from './ThemeConfig';
import Button from './Button';

function App() {

#### cons
t handleToggle = () => switchFunction();
  return (
    <div>
      <Button theme={ThemeConfig.mode === 'light' ? {} : { color: '#fff' }}>Click Me</Button>
      <button onClick={handleToggle}>Toggle Theme</button>
    </div>
  );
}

**Comparison:**

```
Component	Props	Styles
ThemeConfig	mode, switchFunction	-
ThemeTokens	-	colors, typography, spacings
Button	theme	background-color, padding, border-radius
```

**Code:**

```javascript
// ThemeConfig.js
export const ThemeConfig = {
  mode: 'light',
  switchFunction: () => (/* logic to toggle modes */),
};

// ThemeTokens.js
export const ThemeTokens = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    text: '#343a40',
    background: '#fff',
  },
  typography: {
    fontFamily: 'Arial, sans-serif',
    fontSize: '16px',
    lineHeight: '28px',
  },
  spacings: [0, 8, 16, 24],
};

// Button.js (ThemeComponent)
import React from 'react';
import styled from 'styled-components';

const { colors } = ThemeTokens;

export const Button = ({ theme, children }) => {
  return (
    <StyledButton color={theme.colors.primary}>
      {children}
    </StyledButton>
  );
};

const StyledButton = styled.button`
  background-color: ${props => props.color};
  padding: 10px;
  border-radius: 5px;
`;

// Usage
import React from 'react';
import ThemeConfig, { switchFunction } from './ThemeConfig';
import Button from './Button';

function App() {
  const handleToggle = () => switchFunction();
  return (
    <div>
      <Button theme={ThemeConfig.mode === 'light' ? {} : { color: '#fff' }}>Click Me</Button>
      <button onClick={handleToggle}>Toggle Theme</button>
    </div>
  );
}
```

**Example input / output:**

```text
Input:
- ThemeConfig mode is set to 'light'.
- User clicks the toggle button.

Output:
- Button's background color changes from '#007bff' (primary) to white.
- The theme in App component updates to 'dark', causing other components to adapt their styling accordingly.
```

**Data:**

```json
{
  "themeConfig": {
    "mode": "light",
    "switchFunction": "function to toggle modes"
  },
  "themeTokens": {
    "colors": {
      "primary": "#007bff",
      "secondary": "#6c757d",
      "text": "#343a40",
      "background": "#fff"
    },
    "typography": {
      "fontFamily": "Arial, sans-serif",
      "fontSize": "16px",
      "lineHeight": "28px"
    },
    "spacings": [0, 8, 16, 24]
  }
}
```

**Flow:**

1. ThemeConfig
2. Toggle Mode
3. Update App State

