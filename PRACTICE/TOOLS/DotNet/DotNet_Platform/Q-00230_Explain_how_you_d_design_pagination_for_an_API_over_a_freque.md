---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design pagination for an API over a frequently-changing dataset so clients don't see duplicates or gaps — offset vs keyset/cursor pagination and their trade-offs in EF Core.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Entity Framework Core Performance](<../../../../TOOLS/DotNet/DotNet_Platform/Entity_Framework_Core_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design effective pagination for an API serving a frequently-changing dataset, we need to ensure that our pagination mechanism is robust against duplicates and gaps while efficiently handling changes over time. The primary techniques are offset-based and keyset (or cursor) pagination, each with distinct trade-offs.

Firstly, let's briefly understand the context: in a backend system using .NET, specifically Entity Framework Core (EF Core), we often need to handle large datasets that change frequently. Ensuring that clients see consistent results without duplicates or gaps is critical for maintaining user trust and satisfaction.

**Comparison:**

```
Pagination Type	Description	Pros	Cons
Offset	Starts from a given offset.	Simple to implement and understand.	Prone to duplicates and gaps if the dataset changes between requests.
Keyset	Uses a key (e.g., id) to start from.	Consistent results across multiple requests, handles frequent data changes better.	More complex implementation; requires additional checks for edge cases.
```

**Code:**

```csharp
using System;
using Microsoft.EntityFrameworkCore;

public class MyDbContext : DbContext
{
    public DbSet<MyEntity> Entities { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // Example configuration with in-memory database for simplicity.
        optionsBuilder.UseInMemoryDatabase("MyDb");
    }
}
```

**Example input / output:**

```text
// Example usage of offset pagination
using (var context = new MyDbContext())
{
    var results1 = context.Entities.OrderBy(e => e.id).Skip(2).Take(2);
    foreach (var entity in results1)
    {
        Console.WriteLine(entity.value); // Output: C, D
    }
}

// Example usage of keyset pagination
using (var context = new MyDbContext())
{
    var startKey = 3; // Starting from "C"
    var results2 = context.Entities.Where(e => e.id >= startKey).Take(2);
    foreach (var entity in results2)
    {
        Console.WriteLine(entity.value); // Output: C, D
    }
}
```

**Data:**

```json
{
  "entities": [
    { "id": 1, "value": "A" },
    { "id": 2, "value": "B" },
    { "id": 3, "value": "C" },
    { "id": 4, "value": "D" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Check if offset is valid
5. Fetch entities starting from thegiven offset
6. Handle invalid offset scenario,e.g., return an error or fallbackto keyset pagination
7. Return results and update lastseen entity id
8. Use keyset pagination insteadof offset
9. Fetch entities starting from thelast seen entity id
10. End
11. Return results with new lastseen entity id

