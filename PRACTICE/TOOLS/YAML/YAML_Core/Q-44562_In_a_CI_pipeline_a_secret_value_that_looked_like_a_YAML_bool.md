---
type: "practice"
difficulty: "Deep"
---

# In a CI pipeline, a secret value that looked like a YAML boolean (Off) got coerced and broke a feature flag. Walk through YAML 1.1 versus 1.2 boolean rules and how parser version choice affects this.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indentation Pitfalls](<../../../../TOOLS/YAML/YAML_Core/Indentation_Pitfalls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In CI pipelines, we often use YAML to manage configurations, including sensitive data like feature flags. Recently, a situation arose where a secret value that appeared as a YAML boolean (Off) was being parsed differently between YAML 1.1 and 1.2, leading to unexpected behavior in the pipeline. To fully understand this issue, let’s walk through the rules of YAML booleans under these two versions and how parser version choices can impact our feature flag logic.

#### Triage

The question is probing into understanding the nuances of boolean handling in YAML between versions 1.1 and 1.2, specifically how a value like Off could be interpreted differently based on the YAML spec being used by your pipeline’s configuration parser. Key assumptions include:

You are working with a CI pipeline where feature flags are managed using YAML.
The issue arises due to a change in the boolean handling rules between YAML 1.1 and 1.2.

#### First-Principles Solution
YAML 1.1 Boolean Rules

In YAML 1.1, booleans are represented by the literal strings true and false. Any other string values are considered invalid for boolean context. Therefore, a value like Off would not be recognized as a valid boolean under YAML 1.1 rules.

YAML 1.2 Boolean Rules

YAML 1.2 relaxed some of these rules slightly to make certain values more flexible in contexts where booleans might be expected. According to the YAML 1.2 specification, true and false remain as valid boolean values, but other strings can also be parsed as true or false if they fit specific patterns:

A string that starts with t followed by an optional sequence of r, u, and e in any case is recognized as true.
Similarly, a string starting with f followed by an optional sequence of a, l, and s in any case is recognized as false.
Worked Example

#### Cons
ider the following feature flag configuration snippet:

YAML
feature-flag:
  value: Off

In YAML 1.1, this would be considered invalid for boolean context, leading to a parsing error or incorrect handling of the feature flag.

However, in YAML 1.2, if you re-parse Off as part of your pipeline configuration using a YAML parser that supports this rule, it might get interpreted differently:

If Off starts with an "F" (e.g., False), it could be recognized as false.
If it doesn’t fit the pattern, it would still be invalid and potentially lead to a failure or warning.

#### Closing Summary

This issue highlights the importance of understanding YAML versioning and how different versions can affect parsing behavior. Ensuring consistent use of YAML 1.2 in your CI pipeline configurations can prevent such issues, but it requires careful handling of configuration files and possibly updating parser settings if they are still based on older specifications.

#### Common Mistakes/Follow-Ups
Assuming all boolean values follow the same rules: Always check the version of the YAML specification used by your tooling.
Not considering edge cases in boolean parsing: Be aware that even valid strings like Off could be misinterpreted, leading to bugs.
Parser-specific behavior: Different parsers might implement these rules differently. Ensure you are using a consistent parser across your pipeline.

**Comparison:**

```
Version	Boolean Representation
YAML 1.1	true, false only
YAML 1.2	t[ru?e], f[a?l?s]
```

**Code:**

```yaml
# Example YAML 1.2 configuration snippet
feature-flag:
  value: Off
```

**Flow:**

1. yes
2. no
3. yes
4. no
5. yes
6. no
7. YAML 1.1 boolean parsing
8. Is value 'true' or 'false'?
9. Parse as boolean
10. Invalid boolean, parse error
11. YAML 1.2 boolean parsing
12. Does the value match t[ru?e]?
13. Parse as true
14. Does the value match f[a?l?s]?
15. Parse as false
16. Invalid boolean, parse warningor error

