---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to compute the average rating per movie and only show movies with at least 50 ratings.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to calculate the average rating of each movie and ensure that we only display movies that have received at least 50 ratings. Let's break down the solution step by step.

#### Triaging the Question

The core task is to use aggregate functions in SQL (specifically AVG() for calculating the average rating) and filter the results based on a minimum threshold of ratings per movie (HAVING clause).

#### First-Principles Solution
Identify Tables: Assume we have two tables: movies with columns like movie_id, title, etc., and ratings with columns like rating_id, movie_id, rating_value.
Join the Tables: We need to join the movies table with the ratings table on movie_id.
Aggregate Function: Use AVG() to calculate the average rating.
Filter by Count of Ratings: Use a GROUP BY clause and HAVING clause to filter movies that have at least 50 ratings.
Full Solution:

Here is the SQL query:

SQL
SELECT m.movie_id, m.title, AVG(r.rating_value) AS avg_rating
FROM movies m
JOIN ratings r ON m.movie_id = r.movie_id
GROUP BY m.movie_id, m.title
HAVING COUNT(r.rating_id) >= 50;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

This query correctly computes the average rating for each movie and filters out those with fewer than 50 ratings. A common mistake is forgetting to use GROUP BY along with HAVING, which could lead to incorrect results or errors.

Edge Cases:
If a movie has no ratings, it will not be included in the result.
Ensure that rating_value is of a numeric type.

#### Common Mistakes
Forgetting the GROUP BY clause.
Not using the HAVING clause to filter by count.
Incorrect data types for AVG() and COUNT() functions.

**Comparison:**

| movie_id | title        | avg_rating |
|----------|--------------|------------|
| 1        | Movie A      | 4.3        |

**Code:**

```sql
SELECT m.movie_id, m.title, AVG(r.rating_value) AS avg_rating
FROM movies m
JOIN ratings r ON m.movie_id = r.movie_id
GROUP BY m.movie_id, m.title
HAVING COUNT(r.rating_id) >= 50;
```

**Example input / output:**

```text
Input:
movies = [
  { movie_id: 1, title: 'Movie A' },
  { movie_id: 2, title: 'Movie B' }
],
ratings = [
  { rating_id: 1, movie_id: 1, rating_value: 4 },
  { rating_id: 2, movie_id: 1, rating_value: 5 },
  // Assume we have more ratings for both movies
]

Output:
[
  { movie_id: 1, title: 'Movie A', avg_rating: 4.3 }, // Example average value
]
```

**Data:**

```json
{
  "movies": [
    {"movie_id": 1, "title": "Movie A"},
    {"movie_id": 2, "title": "Movie B"}
  ],
  "ratings": [
    {"rating_id": 1, "movie_id": 1, "rating_value": 4},
    {"rating_id": 2, "movie_id": 1, "rating_value": 5},
    // ... more ratings
  ]
}
```

**Flow:**

1. Yes
2. START
3. JOIN tables movies and ratings
4. Filter by count of ratings >=50
5. GROUP BY movie_id, title
6. Aggregate AVG(rating_value)
7. SELECT avg_rating for eachmovie
8. END

