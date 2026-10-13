---
type: "practice"
difficulty: "Intermediate"
---

# Design a class model for a shared component that must render differently by feature flag AND locale AND experiment variant; how do you resolve the combined context once and pass a stable resolved config down?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Design Tokens and Theming](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Design_Tokens_and_Theming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a class model for a shared component in a way that it renders differently based on feature flags, locales, and experiment variants, we need to ensure that the configuration is dynamically resolved once and then passed as a stable object down through the component hierarchy. This approach minimizes redundant computations and ensures thread safety.

#### Triaging the Question

The core of this question is about designing a class-based model for rendering components with dynamic configurations. The interviewer might be probing your understanding of:

Feature Flags: How to conditionally enable or disable features.
Locales: Handling different languages and regional settings.
Experiment Variants: Managing A/B tests or feature rollouts.
Performance Optimization: Ensuring the configuration is resolved only once and cached for subsequent uses.

#### Assumptions
The component can be reused across multiple parts of the application.
We need to support dynamic changes in configurations without re-rendering the entire component tree unnecessarily.
Locale settings are static but may change rarely, whereas feature flags and experiment variants might change frequently.

#### First-Principles Solution
Step 1: Define a Config Class

We start by creating a Config class that encapsulates all dynamic configuration data:

PYTHON
class Config:
    def __init__(self, feature_flags: dict, locale: str, variant: str):
        self.feature_flags = feature_flags
        self.locale = locale
        self.variant = variant

    def get_feature_value(self, feature_key: str) -> bool:
        return self.feature_flags.get(feature_key, False)
Step 2: Resolve Configuration Once

We create a ConfigResolver class that ensures the configuration is resolved only once and cached. This approach leverages memoization to avoid redundant computations.

PYTHON
class ConfigResolver:
    def __init__(self, feature_flags_dict, locale, variant):
        self.feature_flags = feature_flags_dict
        self.locale = locale
        self.variant = variant
        self.resolved_config = None

    def resolve(self) -> Config:
        if not self.resolved_config:
            self.resolved_config = Config(
                self.feature_flags,
                self.locale,
                self.variant
            )
        return self.resolved_config
Step 3: Component Class with Configuration

Next, we design the component class that uses the resolved configuration. The Component class will accept a Config object and use it to determine its rendering logic.

PYTHON
class Component:
    def __init__(self, config: Config):
        self.config = config

    def render(self) -> str:
        if self.config.get_feature_value("newFeature"):
            return f"Rendering new feature with {self.config.locale} locale and variant {self.config.variant}"
        else:
            return "Default rendering"
Step 4: Integration Example

Here’s how these components might be integrated in a sample application.

PYTHON
feature_flags = {"newFeature": True}
locale = "en-US"
variant = "control"

# Resolve the configuration once
config_resolver = ConfigResolver(feature_flags, locale, variant)
resolved_config = config_resolver.resolve()

# Create and render the component with the resolved configuration
component = Component(resolved_config)
print(component.render())

#### Closing Summary

By defining a Config class to encapsulate dynamic configurations and using a ConfigResolver to ensure this configuration is resolved only once, we can effectively manage different rendering scenarios based on feature flags, locales, and experiment variants. This approach optimizes performance while ensuring the component's behavior reflects current contextual settings.

#### Common Mistakes/Follow-Ups
Redundant Configuration Resolutions: Ensuring that the configuration is not re-resolved unnecessarily.
Thread Safety: Making sure that the Config object remains thread-safe if accessed concurrently from multiple threads.
Caching Strategies: Discussing strategies for invalidating cached configurations when relevant flags change.

