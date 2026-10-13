---
type: "practice"
difficulty: "Deep"
---

# How would you detect consecutive days of activity (a streak) for a user using LAG or ROW_NUMBER?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect consecutive days of activity (a streak) for a user, we can leverage window functions in SQL to compare each day's date with the previous one. This approach involves using LAG to fetch data from the previous row and determining if the current date is consecutive to the previous date. Alternatively, using ROW_NUMBER can help us group consecutive dates by assigning sequential numbers based on date order.

**Comparison:**

| user_id | activity_date |
|---------|---------------|
| 1       | 2023-10-05    |
| 1       | 2023-10-06    |
| 1       | 2023-10-07    |
| 2       | 2023-10-08    |
| user_id | activity_date | streak_status |
|---------|---------------|---------------|
| 1       | 2023-10-06    | Streak        |
| 1       | 2023-10-07    | Streak        |

**Flow:**

1. Yes
2. Consecutive
3. Not Consecutive
4. No
5. Start
6. Is user_id the same?
7. Use LAG to fetch previous date
8. Check if consecutive daysusing datediff
9. Mark as Streak
10. Mark as Not a Streak
11. Start new streak for differentuser_id
12. End

