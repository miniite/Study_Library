---
type: "practice"
difficulty: "Basic"
---

# Design a bookmark/save-for-later feature across web and mobile.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a bookmark/save-for-later feature for both web and mobile, we need to ensure that it is user-friendly, efficient, and scalable. At a high level, this involves creating an interface that allows users to save content from any page they visit (web) or open in their app (mobile), storing the URL, title, and sometimes additional metadata like tags or descriptions. The system should be able to sync across devices and support retrieval of saved bookmarks.

To break down the problem step by step, we'll start with understanding the requirements, then move on to designing the data model, choosing a database, implementing caching strategies, securing user data, and finally, discussing scalability and cost implications.

**Comparison:**

```
User ID	URL	Title	Tags	Created At
123	https://example.com	Example Website	web, tutorial	2023-04-05T12:00:00Z
123	https://another-example.com	Another Example Website	web, tutorial	2023-04-06T12:00:00Z
```

**Code:**

```python
# Example code for saving a bookmark in Python (simplified)
def save_bookmark(user_id, url, title):
    # Assume `db` is our database connection object
    db.execute("INSERT INTO bookmarks (user_id, url, title) VALUES (?, ?, ?)", user_id, url, title)

# Example code for retrieving bookmarks in Python (simplified)
def get_bookmarks(user_id):
    # Again, assuming `db` is the database connection
    results = db.execute("SELECT * FROM bookmarks WHERE user_id=?", user_id).fetchall()
    return results
```

**Example input / output:**

```text
Input (Web Bookmark Save):
save_bookmark(123, "https://example.com", "Example Website")

Output:
Bookmark saved successfully with ID 1

Input (Mobile Bookmark Retrieve):
get_bookmarks(123)

Output:
[
  {
    "id": 1,
    "user_id": 123,
    "url": "https://example.com",
    "title": "Example Website",
    "tags": ["web", "tutorial"],
    "created_at": "2023-04-05T12:00:00Z"
  },
  {
    "id": 2,
    "user_id": 123,
    "url": "https://another-example.com",
    "title": "Another Example Website",
    "tags": ["web", "tutorial"],
    "created_at": "2023-04-06T12:00:00Z"
  }
]
```

**Data:**

```json
{
  "bookmarks": [
    {
      "id": 1,
      "user_id": 123,
      "url": "https://example.com",
      "title": "Example Website",
      "tags": ["web", "tutorial"],
      "created_at": "2023-04-05T12:00:00Z"
    },
    {
      "id": 2,
      "user_id": 123,
      "url": "https://another-example.com",
      "title": "Another Example Website",
      "tags": ["web", "tutorial"],
      "created_at": "2023-04-06T12:00:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. User Opens Web Page
10. Is URL Bookmarked?
11. Display Bookmark Icon
12. Show Save For Later Button
13. Update UI to Reflect SavedState
14. User Clicks Save For Later
15. Is User Logged In?
16. Save URL, Title, Tags inDatabase
17. Show Login Prompt
18. Notify User of Success
19. Take User to Login Page
20. Is URL Bookmarked?
21. Display Bookmark Icon
22. Show Save For Later Button
23. Update UI to Reflect SavedState
24. User Clicks Save For Later
25. Is User Logged In?
26. Save URL, Title, Tags inDatabase
27. Show Login Prompt
28. Notify User of Success
29. Take User to Login Page

