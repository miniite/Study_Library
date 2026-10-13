---
type: "practice"
difficulty: "Expert"
---

# How do you govern SDK/client-library strategy across 8 languages for a public API? Build-vs-generate-vs-community, versioning SDKs against API versions, and how you avoid an SDK bug becoming a de-facto breaking change for thousands of apps.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern SDK/client-library strategy across 8 languages for a public API, we need to balance build-vs-generate trade-offs, manage versioning effectively, and ensure that any bugs in an SDK don't become de facto breaking changes for thousands of apps. The key is to establish clear policies and processes while maintaining flexibility and responsiveness.

The core challenge here involves creating a robust and maintainable strategy that ensures the integrity and stability of our API across multiple language ecosystems. This requires us to carefully consider how we generate, build, or develop SDKs, handle versioning, and manage community contributions.

Firstly, let's break down the key components:

Build-vs-Generate: Deciding whether to manually maintain SDKs or automatically generate them from our API specification.
Versioning Strategy: Ensuring that changes in the API don't inadvertently cause breaking changes in existing SDK users.
Community Involvement: Facilitating contributions and maintaining a robust community of developers.

**Comparison:**

| Language | Action                |
|----------|-----------------------|
| Python   | Generate new version  |
| Java     | Update existing       |
| Swift    | Rebuild               |
| Ruby     | Generate new version  |
| Go       | Generate new version  |
| C++      | Build                 |
| JavaScript | Generate new version |
| PHP      | Update existing       |

**Code:**

```python
# Example of a versioning policy function to determine if a change is breaking
def is_breaking_change(old_api_spec, new_api_spec):
    # Implement logic to compare old vs new API specs
    pass
```

**Example input / output:**

```
Input: A request to add a new method PUT to /users.
Output: Version bump from 1.2 to 1.3, and an updated SDK for each of the 8 languages.
MARKDOWN
Copy
| Language | Action                |
|----------|-----------------------|
| Python   | Generate new version  |
| Java     | Update existing       |
| Swift    | Rebuild               |
| Ruby     | Generate new version  |
| Go       | Generate new version  |
| C++      | Build                 |
| JavaScript | Generate new version |
| PHP      | Update existing       |
```

**Data:**

```json
{
  "apiSpec": {
    "version": "1.2",
    "endpoints": [
      {"path": "/users", "methods": ["GET", "POST"]}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. API Version Bump
4. Is change breaking?
5. Generate SDKs for alllanguages
6. Update SDKs as needed
7. Govern and QA process foreach language
8. Govern and QA process foreach language

