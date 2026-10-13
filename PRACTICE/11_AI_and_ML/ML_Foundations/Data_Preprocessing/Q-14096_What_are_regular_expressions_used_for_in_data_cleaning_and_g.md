---
type: "practice"
difficulty: "Intermediate"
---

# What are regular expressions used for in data cleaning, and give an example of extracting a numeric ID from a messy text field in SQL?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Regular expressions (regex) are powerful tools for pattern matching and manipulating strings in data preprocessing and cleaning tasks. They allow us to search, match, and replace patterns within text, making them indispensable for handling messy datasets. In SQL, we can use regex to extract useful information from columns that contain mixed or inconsistent data types.

To illustrate this, let's consider an example where a column customer_notes contains various pieces of customer information, including numeric IDs. These IDs are embedded in the notes alongside other text, and our goal is to extract these IDs into a new column for further analysis. Here’s how we can approach it:

#### Triaging the Question

The question asks us to explain what regular expressions are used for in data cleaning and provide an example of extracting numeric IDs from a messy text field using SQL. The key parts of this task involve:

Understanding regex basics.
Demonstrating how to use regex in SQL.
Showing a practical example with input and output.

#### First-Principles Solution

Regular expressions are patterns used for matching strings or character sequences according to specific rules. In SQL, we can leverage the REGEXP function (or its equivalent depending on the database system) to apply these patterns. For our task, we need a regex that matches numeric IDs embedded in text.

Let's assume the customer notes might look like this:

"Customer ID: 123456 has placed an order."
"Order by Customer #789012."

We want to extract just the numeric parts (e.g., 123456, 789012) and store them in a new column.

SQL Example

Here’s how we can achieve this using SQL with regex:

SQL
-- Sample data
CREATE TABLE customer_notes (
    id INT PRIMARY KEY,
    customer_notes TEXT
);

INSERT INTO customer_notes (id, customer_notes)
VALUES
(1, 'Customer ID: 123456 has placed an order.'),
(2, 'Order by Customer #789012.');

-- Extracting numeric IDs using REGEXP in PostgreSQL
SELECT id,
       -- Use the REGEXP function to extract numeric values from customer_notes
       CASE
           WHEN customer_notes ~* '[0-9]+' THEN regexp_replace(customer_notes, '^.*(\d+).*$', '\1')
           ELSE NULL
       END AS extracted_id
FROM customer_notes;

In this example:

We use ~* to perform a case-insensitive search for digits ([0-9]+) in the customer_notes column.
The regexp_replace function is used to extract the first sequence of digits from each note. The pattern '^.*(\d+).*$' ensures that we only capture the numeric part, ignoring any leading or trailing text.

#### Closing Summary

In summary, regular expressions are essential for data cleaning tasks in SQL, allowing us to accurately and efficiently handle messy text fields by extracting specific patterns of interest. This example demonstrates how to use regex in a practical scenario where we need to extract numeric IDs from customer notes. Common mistakes include incorrect regex patterns leading to false positives or negatives.

#### Common Mistakes/Follow-Ups

Mistake: Using overly complex regex patterns that can be harder to maintain.

#### Follow-Up
Keep the pattern simple and test thoroughly.

Mistake: Not considering all possible variations in input text (e.g., different delimiters).

#### Follow-Up
Ensure robust testing with various edge cases.

**Comparison:**

```
table {
    header: "customer_notes"
    rows:
        "id, customer_notes"
        "1, Customer ID: 123456 has placed an order."
        "2, Order by Customer #789012."
}
```

**Code:**

```sql
-- Example SQL code
CREATE TABLE customer_notes (
    id INT PRIMARY KEY,
    customer_notes TEXT
);

INSERT INTO customer_notes (id, customer_notes)
VALUES
(1, 'Customer ID: 123456 has placed an order.'),
(2, 'Order by Customer #789012.');

-- Extract numeric IDs using REGEXP in PostgreSQL
SELECT id,
       CASE
           WHEN customer_notes ~* '[0-9]+' THEN regexp_replace(customer_notes, '^.*(\d+).*$', '\1')
           ELSE NULL
       END AS extracted_id
FROM customer_notes;
```

**Example input / output:**

| id | extracted_id |
|----|--------------|
| 1  | 123456       |
| 2  | 789012       |

**Data:**

```sql
-- Sample data insertion
INSERT INTO customer_notes (id, customer_notes)
VALUES
(1, 'Customer ID: 123456 has placed an order.'),
(2, 'Order by Customer #789012.');
```

**Flow:**

1. Yes
2. No
3. Input Data
4. Is regex pattern correct?
5. Extract IDs using REGEXP
6. Revise and test regex pattern
7. Output Cleaned Data

