---
type: "practice"
difficulty: "Intermediate"
---

# What do the ADD_MONTHS, MONTHS_BETWEEN, and LAST_DAY date functions do in Oracle?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Aggregate Functions](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Aggregate_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this answer, we'll explore three essential date manipulation functions in Oracle: ADD_MONTHS, MONTHS_BETWEEN, and LAST_DAY. These functions are crucial for handling date arithmetic and extracting specific information from dates, making them indispensable tools for a database developer.

Firstly, let's understand what these functions do at a high level:

ADD_MONTHS adds or subtracts a specified number of months to/from a given date.
MONTHS_BETWEEN calculates the difference in months between two dates.
LAST_DAY returns the last day of the month for a given date.

Now, let's dive into each function with detailed explanations and examples:

ADD_MONTHS

The ADD_MONTHS function is used to add or subtract a specified number of months from a date. The syntax is as follows:

SQL
ADD_MONTHS(date_expression, number_of_months)

#### Example
Usage:

Let's say we have a start date of 2023-10-15 and want to calculate the end date after adding 6 months.

SQL
SELECT ADD_MONTHS('2023-10-15', 6) AS new_date FROM dual;

The result would be 2024-04-15.

MONTHS_BETWEEN

MONTHS_BETWEEN returns the difference in months between two dates. The syntax is:

SQL
MONTHS_BETWEEN(date1, date2)

#### Example
Usage:

To find out how many months are between 2023-01-01 and 2024-05-01, you would use:

SQL
SELECT MONTHS_BETWEEN('2024-05-01', '2023-01-01') AS months_between FROM dual;

The output is 17.8333, indicating 17 full months plus about a day.

LAST_DAY

LAST_DAY returns the last day of the month for a given date. The syntax is:

SQL
LAST_DAY(date_expression)

#### Example
Usage:

To find out the last day of February 2024, which is a leap year:

SQL
SELECT LAST_DAY('2024-02-15') AS last_day FROM dual;

The result would be 2024-02-29.

#### Closing Summary
and Common Mistakes
Correct Usage: Ensure you understand the expected date format (usually YYYY-MM-DD) for inputs.
Subtraction with ADD_MONTHS: You can subtract months by using a negative number. For example, to go back 3 months from '2024-01-15', use:
SQL
SELECT ADD_MONTHS('2024-01-15', -3) AS new_date FROM dual;
Handling Edge Cases: Be cautious with dates around the end of the month, especially when dealing with leap years.

By mastering these functions, you can effectively handle date calculations in Oracle databases.

**Comparison:**

| Function     | Description                                         |
|--------------|----------------------------------------------------|
| ADD_MONTHS   | Adds or subtracts months from a given date         |
| MONTHS_BETWEEN | Calculates the difference in months between two dates  |
| LAST_DAY    | Returns the last day of the month for a given date  |

**Code:**

```sql
-- Example usage of ADD_MONTHS
SELECT ADD_MONTHS('2023-10-15', 6) AS new_date FROM dual;

-- Example usage of MONTHS_BETWEEN
SELECT MONTHS_BETWEEN('2024-05-01', '2023-01-01') AS months_between FROM dual;

-- Example usage of LAST_DAY
SELECT LAST_DAY('2024-02-15') AS last_day FROM dual;
```

**Example input / output:**

```text
-- Input for ADD_MONTHS example
SELECT ADD_MONTHS('2023-10-15', 6) AS new_date FROM dual;

-- Output: 2024-04-15

-- Input for MONTHS_BETWEEN example
SELECT MONTHS_BETWEEN('2024-05-01', '2023-01-01') AS months_between FROM dual;

-- Output: 17.8333

-- Input for LAST_DAY example
SELECT LAST_DAY('2024-02-15') AS last_day FROM dual;

-- Output: 29-FEB-2024
```

**Data:**

```sql
-- Sample data for testing date functions
CREATE TABLE test_dates (
    start_date DATE,
    end_date DATE
);
INSERT INTO test_dates (start_date, end_date) VALUES ('2023-10-15', '2024-05-01');
```

**Flow:**

1. Yes
2. No
3. ADD_MONTHS
4. Months Positive?
5. Add Months
6. Subtract Months
7. Return New Date
8. Return New Date
9. last_day_function
10. last_day_function
11. Return Last Day of Month
12. Output: <Last Day>

