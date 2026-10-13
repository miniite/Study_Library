---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd design a SQL schema for storing tool definitions (name, JSON schema, version) that an agent loads, and query the active version set for a given deployment.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a SQL schema for storing tool definitions and querying the active version set for a given deployment, we need to ensure our database schema is flexible enough to accommodate various versions of tools while providing efficient queries for retrieving the current active version.

#### Triaging the Question

The core of this problem involves designing a schema that can store metadata about each tool (name, JSON schema, and version) and then allow us to query which version of a tool is currently active within a specific deployment. The key aspects being probed include understanding normalization, flexibility in storing complex data types like JSON, handling many-to-one relationships, and efficient querying.

#### Assumptions
Tool Definitions: Each tool has a name, a JSON schema that describes its input/output structure, and one or more versions.
Deployments: Multiple deployments can use different versions of the same tool.
Active Version: For each tool in a deployment, there is exactly one version that is currently active.

#### First-Principles Solution

Table Structure:

tools: Stores information about each tool.
SQL
CREATE TABLE tools (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE,
    json_schema TEXT NOT NULL
);
tool_versions: Stores versioning information for each tool.
SQL
CREATE TABLE tool_versions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tool_id INT NOT NULL,
    version_number VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (tool_id) REFERENCES tools(id)
);
deployments: Stores deployment information.
SQL
CREATE TABLE deployments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE
);

Linking Deployments and Tool Versions:

A many-to-one relationship exists between deployments and tool_versions.
SQL
CREATE TABLE deployment_tool_versions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    deployment_id INT NOT NULL,
    tool_version_id INT NOT NULL,
    is_active BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (deployment_id) REFERENCES deployments(id),
    FOREIGN KEY (tool_version_id) REFERENCES tool_versions(id)
);

Inserting Data and Marking Active Versions:

Inserting new versions of tools and marking them as active.
SQL
INSERT INTO tools (name, json_schema) VALUES ('example_tool', '{
    "type": "object",
    "properties": {
        "input": {"type": "string"},
        "output": {"type": "array"}
    }
}');

INSERT INTO tool_versions (tool_id, version_number, is_active)
VALUES ((SELECT id FROM tools WHERE name = 'example_tool'), '1.0', TRUE);

-- Marking the latest version as active
UPDATE tool_versions SET is_active = FALSE WHERE tool_id = (SELECT id FROM tools WHERE name = 'example_tool');
INSERT INTO tool_versions (tool_id, version_number, is_active)
VALUES ((SELECT id FROM tools WHERE name = 'example_tool'), '2.0', TRUE);

-- Linking a deployment to an active tool version
INSERT INTO deployments (name) VALUES ('production_deployment');

UPDATE deployment_tool_versions SET is_active = FALSE WHERE deployment_id = (SELECT id FROM deployments WHERE name = 'production_deployment');
INSERT INTO deployment_tool_versions (deployment_id, tool_version_id, is_active)
VALUES ((SELECT id FROM deployments WHERE name = 'production_deployment'), (SELECT id FROM tool_versions WHERE version_number = '2.0' AND tool_id = (SELECT id FROM tools WHERE name = 'example_tool')), TRUE);

Querying Active Tool Versions for a Deployment:

Retrieving the active versions of all tools for a specific deployment.
SQL
SELECT
    t.name,
    tv.version_number,
    tv.json_schema
FROM
    tool_versions AS tv
    JOIN tools AS t ON tv.tool_id = t.id
    JOIN deployment_tool_versions AS dtv ON tv.id = dtv.tool_version_id
WHERE
    dtv.deployment_id = (SELECT id FROM deployments WHERE name = 'production_deployment') AND
    dtv.is_active = TRUE;

#### Closing Summary
and Common Mistakes:

To summarize, the schema design involves three main tables: tools, tool_versions, and deployments, with an additional linking table deployment_tool_versions to handle many-to-one relationships. This approach ensures flexibility in managing different versions of tools and their active status across various deployments.

#### Common mistakes
include not properly normalizing the data, failing to update the active version when inserting a new one, or missing critical foreign key constraints. Additionally, ensuring that the JSON schema is stored efficiently and queried effectively can be challenging without proper indexing and data type handling in SQL.

**Code:**

```sql
CREATE TABLE tools (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE,
    json_schema TEXT NOT NULL
);

CREATE TABLE tool_versions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    tool_id INT NOT NULL,
    version_number VARCHAR(100) NOT NULL,
    is_active BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (tool_id) REFERENCES tools(id)
);

CREATE TABLE deployments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL UNIQUE
);

CREATE TABLE deployment_tool_versions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    deployment_id INT NOT NULL,
    tool_version_id INT NOT NULL,
    is_active BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (deployment_id) REFERENCES deployments(id),
    FOREIGN KEY (tool_version_id) REFERENCES tool_versions(id)
);
```

**Example input / output:**

```text
-- Example insertions and updates
INSERT INTO tools (name, json_schema) VALUES ('example_tool', '{
         "type": "object",
         "properties": {
             "input": {"type": "string"},
             "output": {"type": "array"}
         }
     }');

INSERT INTO tool_versions (tool_id, version_number, is_active)
VALUES ((SELECT id FROM tools WHERE name = 'example_tool'), '1.0', TRUE);

UPDATE tool_versions SET is_active = FALSE WHERE tool_id = (SELECT id FROM tools WHERE name = 'example_tool');
INSERT INTO tool_versions (tool_id, version_number, is_active)
VALUES ((SELECT id FROM tools WHERE name = 'example_tool'), '2.0', TRUE);

INSERT INTO deployments (name) VALUES ('production_deployment');

UPDATE deployment_tool_versions SET is_active = FALSE WHERE deployment_id = (SELECT id FROM deployments WHERE name = 'production_deployment');
INSERT INTO deployment_tool_versions (deployment_id, tool_version_id, is_active)
VALUES ((SELECT id FROM deployments WHERE name = 'production_deployment'), (SELECT id FROM tool_versions WHERE version_number = '2.0' AND tool_id = (SELECT id FROM tools WHERE name = 'example_tool')), TRUE);
```

