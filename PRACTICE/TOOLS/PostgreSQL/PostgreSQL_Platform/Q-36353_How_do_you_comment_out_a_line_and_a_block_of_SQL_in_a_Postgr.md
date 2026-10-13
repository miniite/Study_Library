---
type: "practice"
difficulty: "Basic"
---

# How do you comment out a line and a block of SQL in a PostgreSQL script and why does it matter in migration files?

**Difficulty:** Basic · **Levels:** Fresher · [↑ DELETE TRUNCATE and DROP](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, commenting out lines or blocks of SQL is crucial when working with scripts, especially during migrations. By using specific comment styles, you can temporarily disable parts of your script without removing them entirely, making it easier to test and refine your database schema changes.

PostgreSQL supports two types of single-line comments: -- for a line-comment, and # for a hash-comment (though this is typically not used as PostgreSQL primarily recognizes --). For multi-line comments, you can use /* to start the comment block and */ to end it. Here’s how each looks in practice:

Single Line Comment:

SQL
-- This is a single line comment

Block Comment:

SQL
/*
This is a multi-line
comment.
*/

Using comments effectively ensures that your migration scripts remain readable and maintainable, allowing you to isolate problematic sections or experimental code.

