---
type: "practice"
difficulty: "Deep"
---

# Your observability-config monorepo reruns promtool and dashboard-lint across every service on each PR, and CI now takes 20 minutes. Design path-filtered change detection so only the changed service's rules validate, including when a shared recording-rule library changes that a service imports.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this AIOps scenario, we're dealing with optimizing CI processes in a monorepo structure where observability-config is stored. The current setup runs promtool and dashboard-lint across every service on each Pull Request (PR), which results in an unreasonably long 20-minute CI time. We need to design path-based change detection so that only the relevant services are validated, thereby significantly reducing the build time.

#### Triage

The core issue here is optimizing the CI process by ensuring that only changed or affected services are revalidated during a PR. This involves understanding how dependencies and changes propagate through the monorepo structure. We'll make assumptions about the repo's structure, dependency relationships, and the tools used for linting and validation.

#### First-Principles Solution
Dependency Analysis: Identify which services import or depend on shared components (like recording rules).
Change Detection: Implement a mechanism to detect changes in specific paths.
Selective Validation: Validate only the affected services when a change occurs.
Step-by-Step Solution

Repository Structure and Dependencies

Assume the observability-config monorepo has a structure where shared components are stored in one or more directories (e.g., shared/rules).
Services that import these rules might have configurations like:
YAML
services:
  - name: service-a
    path: /path/to/service-a
    dependencies:
      - recording-rules: ../shared/rules

Change Detection Mechanism

Implement a Git hook or pre-commit script to detect changes in the shared/rules directory.
Use file watchers (like inotifywait on Linux) to monitor specific paths.

Selective Validation Logic

For each service that imports these rules, store its path and dependency information.
When a change is detected, use this information to determine which services need revalidation:
PYTHON
def detect_changes(dirs_to_watch):
    changed_files = set()
    for dir in dirs_to_watch:
        # Watch and track changes using file system events or Git diff
        pass
    return changed_files

def validate_services(changed_files, service_dependencies):
    services_to_validate = set()
    for file_path in changed_files:
        for dependency_info in service_dependencies.values():
            if any(file_path.startswith(path) for path in dependency_info['paths']):
                services_to_validate.add(dependency_info['service_name'])
    return services_to_validate

# Example usage
changed_files = detect_changes(['shared/rules'])
validated_services = validate_services(changed_files, service_dependencies)

CI Integration

Integrate the selective validation logic into the CI pipeline.
Configure GitHub Actions or Jenkins to trigger only these services' validations.

#### Closing Summary
and Common Mistakes

By implementing path-based change detection, we can significantly reduce the build time for our observability-config monorepo. The key is to ensure that only the affected services are revalidated, which optimizes the CI process. Common mistakes include not properly identifying dependencies or failing to implement effective Git hooks/file watchers.

#### Follow-Ups
How would you handle changes in deeply nested directories?
What if a service's dependency rules change frequently?
How can we ensure that all relevant services are validated even when they indirectly depend on the changed files?

**Comparison:**

```
Service	Path	Dependencies
service-a	/path/to/service-a	../shared/rules
service-b	/path/to/service-b	../../another-shared/library
```

**Code:**

```python
def detect_changes(dirs_to_watch):
    # Placeholder for actual implementation using file system events or Git diff
    pass

def validate_services(changed_files, service_dependencies):
    services_to_validate = set()
    for file_path in changed_files:
        for dependency_info in service_dependencies.values():
            if any(file_path.startswith(path) for path in dependency_info['paths']):
                services_to_validate.add(dependency_info['service_name'])
    return services_to_validate
```

**Example input / output:**

```text
# Example input -> output flow
changed_files = detect_changes(['shared/rules'])
validated_services = validate_services(changed_files, {
    'service-a': {'paths': ['../shared/rules'], 'service_name': 'service-a'},
    'service-b': {'paths': ['../../another-shared/library'], 'service_name': 'service-b'}
})
print(validated_services)  # Output: set(['service-a'])
```

**Data:**

```yaml
services:
  - name: service-a
    path: /path/to/service-a
    dependencies:
      - recording-rules: ../shared/rules
  - name: service-b
    path: /path/to/service-b
    dependencies:
      - recording-rules: ../../another-shared/library
```

**Flow:**

1. Yes
2. No
3. Change Detection
4. Detect changes in shared rules
5. Are services affected?
6. Validate affected services
7. Skip validation

