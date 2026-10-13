---
type: "practice"
difficulty: "Deep"
---

# You're designing a component library's theming with design tokens; a team needs to override just one token per component instance; how do you design the token resolution so overrides cascade predictably?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Design Tokens and Theming](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Design_Tokens_and_Theming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a low-level (LLD) theme system that supports component-specific token overrides, the core challenge is to ensure that these overrides are applied in a predictable and cascading manner without causing conflicts or unintended side effects. This requires a deep understanding of how design tokens interact at different levels of the application hierarchy.

Firstly, let's clarify what we're dealing with: we have a set of default design tokens (colors, fonts, spacing, etc.) that are used globally in our component library. Additionally, users should be able to override any token on a per-component basis without affecting other components or causing global state issues. The key is to establish a consistent and scalable approach to token resolution.

To achieve this, we'll design a system where tokens are resolved based on the nearest specific override, ensuring that more local changes take precedence over global defaults.

Token Structure: Each component instance can have its own set of overridden tokens.
Token Resolution Logic: Tokens are looked up from the most specific to the most general level in the hierarchy.

#### Performance Considerations
We need an efficient way to resolve tokens without causing excessive overhead, especially during rendering.

Let's break down the solution step-by-step:

Step 1: Define Token Hierarchy

We'll organize tokens into a hierarchical structure where each component can have its own override layer on top of the default token values. This hierarchy will allow us to build up the final set of tokens from the most specific (component-level) to the most general (global).

Global Defaults
Component 1 Overrides
Component 2 Overrides
Sub-component 1 Overrides
Sub-component 2 Overrides
Step 2: Token Resolution Algorithm

The token resolution algorithm will traverse this hierarchy from the most specific to the most general, applying the nearest override for each token.

PYTHON
def resolve_tokens(defaults, component_overrides):
    resolved = {}

    # Start with global defaults
    resolved.update(defaults)

    # Traverse the component overrides in a depth-first manner
    def dfs(overrides):
        for key, value in overrides.items():
            if isinstance(value, dict):  # Recursive case: sub-component overrides
                resolved[key] = dfs(value)
            else:
                resolved[key] = value

    dfs(component_overrides)

    return resolved

# Example usage
default_tokens = {
    "color": "#0000FF",
    "font-size": "16px"
}

component_1_overrides = {
    "color": "#FF0000",  # Red
    "sub-component-1": {
        "color": "#00FF00"  # Green
    }
}

resolved_tokens = resolve_tokens(default_tokens, component_1_overrides)
print(resolved_tokens)  # {'color': '#00FF00', 'font-size': '16px'}
Step 3: Performance Optimization

To ensure efficient token resolution during rendering, we can precompute and cache the resolved tokens at build time. This way, runtime overhead is minimized.

PYTHON
from functools import lru_cache

@lru_cache(maxsize=2048)
def resolve_tokens_caching(defaults, component_overrides):
    return resolve_tokens(defaults, component_overrides)

# Example usage with caching
resolved_tokens_cached = resolve_tokens_caching(default_tokens, component_1_overrides)

#### Closing Summary
and Common Mistakes
Common Mistake 1: Failing to handle nested overrides properly can lead to unexpected behavior. Ensure that the token resolution logic correctly propagates changes through nested components.
Common Mistake 2: Ignoring performance implications during rendering. Use caching strategies to minimize runtime overhead, especially in large applications with many components.

By following this approach, we ensure that our theming system is both flexible and efficient, allowing for predictable and maintainable component overrides without introducing global state issues or cascading errors.

**Comparison:**

| Token     | Default Value   | Component Override       |
|-----------|-----------------|--------------------------|
| color     | #0000FF         | Red (#FF0000)            |
| sub-component-1.color | Inherit default  | Green (#00FF00)          |

**Code:**

```python
def resolve_tokens(defaults, component_overrides):
    resolved = {}

    # Start with global defaults
    resolved.update(defaults)

    # Traverse the component overrides in a depth-first manner
    def dfs(overrides):
        for key, value in overrides.items():
            if isinstance(value, dict):  # Recursive case: sub-component overrides
                resolved[key] = dfs(value)
            else:
                resolved[key] = value

    dfs(component_overrides)

    return resolved

def resolve_tokens_caching(defaults, component_overrides):
    from functools import lru_cache

    @lru_cache(maxsize=2048)
    def cached_resolve_tokens(defaults, component_overrides):
        return resolve_tokens(defaults, component_overrides)

    return cached_resolve_tokens(defaults, component_overrides)
```

**Example input / output:**

```text
# Example input and output
default_tokens = {
    "color": "#0000FF",
    "font-size": "16px"
}

component_1_overrides = {
    "color": "#FF0000",  # Red
    "sub-component-1": {
        "color": "#00FF00"  # Green
    }
}

resolved_tokens = resolve_tokens(default_tokens, component_1_overrides)
print(resolved_tokens)  # {'color': '#00FF00', 'font-size': '16px'}

# Example with caching
resolved_tokens_cached = resolve_tokens_caching(default_tokens, component_1_overrides)
print(resolved_tokens_cached)  # Same as resolved_tokens due to caching
```

**Data:**

```json
{
  "default_tokens": {
    "color": "#0000FF",
    "font-size": "16px"
  },
  "component_1_overrides": {
    "color": "#FF0000",  # Red
    "sub-component-1": {
      "color": "#00FF00"  # Green
    }
  }
}
```

**Flow:**

1. No Override
2. Has Override
3. Yes
4. No
5. Start
6. Get Default Tokens
7. Initialize Resolved Tokens withDefaults
8. Check for ComponentOverrides
9. Return Resolved Tokens
10. Traverse Overrides Depth-First
11. Resolve Sub-ComponentOverrides Recursively
12. Update Resolved Tokens withCurrent Override
13. Continue to Next Override
14. Check for More Overrides

