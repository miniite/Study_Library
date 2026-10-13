---
type: "practice"
difficulty: "Intermediate"
---

# How do you paginate a large result set efficiently with Skip and Take in EF Core?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Entity Framework Core Performance](<../../../../TOOLS/DotNet/DotNet_Platform/Entity_Framework_Core_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large datasets, efficient pagination is crucial to ensure performance and responsiveness. In .NET using Entity Framework Core (EF Core), Skip and Take are commonly used methods to implement this. The goal is to fetch only the desired subset of data from a database without loading the entire dataset into memory.

#### Triage

The question asks how to paginate large result sets efficiently using Skip and Take in EF Core. This involves understanding the limitations and optimizations needed when working with large datasets. We'll assume that the backend is built on .NET, specifically using Entity Framework Core as the ORM for interacting with a database.

#### First Principles Solution

Pagination using Skip and Take can be implemented by first skipping a specified number of records and then taking a certain number of records from the result set. This approach involves two main steps:

Skip: Skip over a specified number of elements.
Take: Take a specified number of elements.

However, simply using Skip and Take with large datasets can lead to inefficient queries because it often requires loading all preceding records into memory before returning the desired subset.

Optimized Approach

To handle large result sets efficiently, we need to consider the database's capabilities. One approach is to use Database Pagination or Row Numbering, which allows us to fetch only the relevant rows directly from the database without needing to load unnecessary data. Here’s how you can implement this:

1. Using Skip and Take
CSHARP
public async Task<List<T>> GetPagedResults(int page, int pageSize)
{
    return await _context.Set<T>()
                         .OrderBy(x => x.Id) // Assuming Id is a good ordering key
                         .Skip((page - 1) * pageSize)
                         .Take(pageSize)
                         .ToListAsync();
}
2. Using RowNumber

A more efficient approach leverages the database's row numbering capabilities, which are supported by SQL Server and other databases.

CSHARP
public async Task<List<T>> GetPagedResults(int page, int pageSize)
{
    return await _context.Set<T>()
                         .FromSqlRaw("SELECT * FROM ( SELECT ROW_NUMBER() OVER(ORDER BY Id) as RowNum, * FROM T ) AS Subquery WHERE RowNum BETWEEN @start AND @end",
                                     new { start = (page - 1) * pageSize + 1, end = page * pageSize })
                         .ToListAsync();
}

#### Closing Summary

In summary, while Skip and Take are straightforward for implementing pagination in EF Core, they can lead to inefficient performance with large datasets. Utilizing database-specific features like row numbering provides a more efficient way to handle large result sets by directly querying the relevant rows from the database.

#### Common Mistakes/Follow-Ups
Using Skip and Take: This approach can lead to loading unnecessary data, impacting memory usage.
Optimizing with Row Numbering: Ensure your ordering key is appropriate and does not cause performance issues (e.g., avoid heavy calculations in the order by clause).
Handling Edge Cases: What happens if there are fewer results than expected? How do you handle pagination at the last page?
Query Performance: Always profile and optimize queries to ensure they perform well under load.

**Code:**

```

public async Task<List<T>> GetPagedResults(int page, int pageSize) { return await _context.Set<T>() .OrderBy(x => x.Id) // Assuming Id is a good ordering key .Skip((page - 1) * pageSize) .Take(pageSize) .ToListAsync(); }

CODE
Copy
```csharp
public async Task<List<T>> GetPagedResults(int page, int pageSize)
{
    return await _context.Set<T>()
                         .FromSqlRaw("SELECT * FROM ( SELECT ROW_NUMBER() OVER(ORDER BY Id) as RowNum, * FROM T ) AS Subquery WHERE RowNum BETWEEN @start AND @end",
                                     new { start = (page - 1) * pageSize + 1, end = page * pageSize })
                         .ToListAsync();
}
```

