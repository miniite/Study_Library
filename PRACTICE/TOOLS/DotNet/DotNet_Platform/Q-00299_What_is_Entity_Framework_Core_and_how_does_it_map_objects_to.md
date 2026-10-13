---
type: "practice"
difficulty: "Basic"
---

# What is Entity Framework Core and how does it map objects to relational tables (ORM)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Entity Framework Core Performance](<../../../../TOOLS/DotNet/DotNet_Platform/Entity_Framework_Core_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Entity Framework Core is a popular object-relational mapper (ORM) for .NET applications, designed to simplify database access by automatically mapping your .NET classes to the underlying relational tables in a database. At its core, it abstracts away much of the boilerplate code needed to work with databases, allowing you to focus on writing business logic rather than handling low-level database operations.

To understand how Entity Framework Core maps objects to relational tables, we need to break down the key concepts and components involved.

**Comparison:**

```
Blog Table	Post Table
BlogId (Primary Key) <br> Url	PostId (Primary Key) <br> Title <br> Content <br> BlogId (Foreign Key to Blog)
```

**Code:**

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    // Navigation property
    public virtual ICollection<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    // Foreign key to Blog table (navigation property)
    public int BlogId { get; set; }
    public virtual Blog Blog { get; set; }
}

This is a simple example of defining two entities in C# using the Entity Framework Core code-first approach. Let’s dive into how these objects are mapped to tables.
```

**Flow:**

1. Blog
2. Posts

