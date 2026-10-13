---
type: "practice"
difficulty: "Deep"
---

# How would you evolve a public REST API contract (adding a required field, renaming, changing a type) without breaking existing clients, and how does System.Text.Json help you stay backward-compatible?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ASP.NET Core Web APIs and Auth](<../../../../TOOLS/DotNet/DotNet_Platform/ASP.NET_Core_Web_APIs_and_Auth.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evolving a public .NET REST API to add or modify fields in a way that doesn't break existing clients, we need to ensure our contract evolves gracefully. This means making sure the new changes don’t invalidate the serialized data expected by older clients. In this context, adding a required field, renaming an existing one, and changing a type are significant changes that require careful handling.

To achieve backward compatibility while evolving the API, we can leverage System.Text.Json in .NET. This library provides powerful deserialization and serialization capabilities with options to handle such transformations seamlessly.

#### Triaging the Question

The question is essentially asking how to evolve an existing JSON schema without breaking clients who are already consuming it. The core challenge here is ensuring that new versions of our API can coexist peacefully with old clients, which might be using the previous version's schemas.

#### Assumptions
We use System.Text.Json for serialization and deserialization.
Clients expect certain fields to exist or have specific types.
We need a strategy to evolve the schema without disrupting existing integrations.

#### First Principles Solution

To handle these changes, we can adopt a few strategies:

1. Adding a Required Field

When adding a new required field, we can initialize its value in our backend logic and ensure that it is always present when sending responses back to clients. For incoming requests, if the client sends data for this field (which they are not obligated to do), we will treat it as an optional update.

2. Renaming a Field

Renaming fields requires careful handling because changing keys in JSON objects can break existing client-side code that relies on specific key names. One approach is to use metadata or alternative deserialization strategies provided by System.Text.Json.

3. Changing the Type of a Field

Changing field types (e.g., from string to int) may require converting between these types when serializing and deserializing data.

#### Implementation
Using System.Text.Json

Let's walk through an example where we have an existing API with the following JSON schema:

JSON
{
  "id": 123,
  "name": "Example",
  "description": "A brief description"
}

We want to add a required age field and rename the description field to summary.

Step-by-Step Solution

Adding a Required Field:

In our backend, when sending a response, always include an age field even if we don’t have this data yet.
On the client side, treat the absence of the age field as a missing optional value.

Renaming a Field:

Use the JsonConverter attribute to handle renaming fields during deserialization and serialization. For example:

CSHARP
public class User
{
    [JsonPropertyName("summary")] // Renamed from "description"
    public string Description { get; set; }

    public int Age { get; set; }
}

Changing the Type of a Field:

For types like string to int, ensure you have conversion logic in place, either on your backend when serializing and validating data, or on the client side.
Sample Code

Here is how we might implement this using System.Text.Json:

CSHARP
using System;
using System.Text.Json;
using System.Text.Json.Serialization;

public class User
{
    [JsonPropertyName("summary")] // Renamed from "description"
    public string Description { get; set; }

    public int Age { get; set; }
}

// Example of deserialization with conversion logic
string jsonData = "{\"id\": 1, \"name\": \"User\", \"summary\": \"A brief description\"}";
var user = JsonSerializer.Deserialize<User>(jsonData);

if (user.Description == "A brief description") // Conversion check
{

#### Cons
ole.WriteLine("Description field was correctly converted.");
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, evolving a public REST API without breaking clients can be managed effectively by leveraging System.Text.Json's deserialization features. We can add fields with default values, rename fields using attributes, and manage type changes through custom logic.

#### Common Mistakes
Failing to handle optional fields properly.
Overlooking conversion issues between different data types during serialization/deserialization.
Not testing thoroughly across all client versions and new API versions.

#### Follow-ups
might include discussing additional strategies like versioning the API or using schema validation tools to catch issues early.

