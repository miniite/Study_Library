---
type: "practice"
difficulty: "Intermediate"
---

# What is the DbContext in EF Core and what is its role in change tracking?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Entity Framework Core Performance](<../../../../TOOLS/DotNet/DotNet_Platform/Entity_Framework_Core_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In Entity Framework Core (EF Core), DbContext plays a crucial role as the primary entry point to interact with the database. It acts as an in-memory representation of your data model and is responsible for managing object context and change tracking.

#### Triage
The question asks us to explain what DbContext is in EF Core, focusing on its role in change tracking. We need to ensure we cover how it manages entities and their state transitions (new, added, modified, deleted) within the database context.

#### First-Principles Solution

Introduction to DbContext: A DbContext instance is essentially an in-memory representation of your data model that maintains a connection to the database. It acts as a facade for accessing entities and performing CRUD operations.
Change Tracking Mechanism: The core functionality of DbContext revolves around its change tracking mechanism, which keeps track of changes made to entities within a single execution context. This is crucial for implementing Entity Framework's Object-Relational Mapping (ORM) capabilities.
Entity States:
Unchanged: Entities that have not been modified since their last load from the database.
Added: New entities that were created but never persisted to the database.
Modified: Existing entities that have been changed and need to be updated in the database.
Deleted: Entities marked for deletion, which will be removed from the database on save.

Detailed Explanation:

DbContext Initialization: When a DbContext is instantiated, it initializes an internal cache (also known as the change tracker) to keep track of entities and their state. This cache is used to determine which entities have been added, modified, or deleted since the last query.
Change Tracking Flow:
Adding Entities: When you call methods like Add on a DbContext, it marks the entity as "added" in its internal change tracker. Subsequent queries will return this entity with an "Added" state.
Modifying Entities: Any changes made to properties of entities within the context are tracked. For instance, if you update a property using Entity.Property = newValue, EF Core records this modification and marks the entity as "Modified".
Deleting Entities: Using methods like Remove or setting an entity's state to Deleted via Entry(entity).State = EntityState.Deleted tells EF Core that this entity should be removed from the database.
Saving Changes: When you call SaveChanges, EF Core uses the change tracker to determine which entities need to be persisted, deleted, or updated. It then generates SQL commands based on these changes and executes them against the database.

#### Closing Summary
Understanding how DbContext manages entity states is essential for working with EF Core effectively. The change tracking mechanism ensures that your application can efficiently handle data consistency and integrity during transactions.

#### Common Mistakes/Follow-ups

Assuming Change Tracking Works Across Multiple Contexts: Each DbContext instance operates independently, so changes tracked in one context do not affect others.
Not Saving Changes Explicitly: Remember to call SaveChanges when you want your modifications to be persisted to the database.

**Comparison:**

| EntityState | Description                         |
|-------------|-------------------------------------|
| Unchanged   | Entity not modified since last load. |
| Added       | New entity, not in the database yet.|
| Modified    | Entity properties have been changed. |
| Deleted     | Entity marked for removal from db.  |

**Code:**

```csharp
using Microsoft.EntityFrameworkCore;

public class MyDbContext : DbContext
{
    public DbSet<MyEntity> Entities { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer("YourConnectionStringHere");
    }
}

public class MyEntity
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

**Example input / output:**

```
Adding a New Entity:
CSHARP
Copy
using (var context = new MyDbContext())
{
    var entity = new MyEntity { Name = "NewEntity" };
    context.Add(entity); // Marks the entity as added
    context.SaveChanges(); // Persists changes to database
}
Modifying an Existing Entity:
CSHARP
Copy
using (var context = new MyDbContext())
{
    var entity = context.Entities.FirstOrDefault(e => e.Id == 1);
    if (entity != null)
    {
        entity.Name = "ModifiedName"; // Marks the entity as modified
        context.SaveChanges(); // Persists changes to database
    }
}
Deleting an Entity:
CSHARP
Copy
using (var context = new MyDbContext())
{
    var entity = context.Entities.FirstOrDefault(e => e.Id == 1);
    if (entity != null)
    {
        context.Remove(entity); // Marks the entity as deleted
        context.SaveChanges(); // Persists changes to database
    }
}
```

**Data:**

```json
{
  "Entities": [
    { "Id": 1, "Name": "EntityOne" },
    { "Id": 2, "Name": "EntityTwo" }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. DbContext Instantiation
5. Is Entity Added?
6. Is Entity Modified?
7. Change Tracking Marked asModified
8. Change Tracking Marked asAdded
9. Entity Deletion
10. Change Tracking Marked asDeleted
11. Save Changes
12. Generate SQL Commands andExecute

