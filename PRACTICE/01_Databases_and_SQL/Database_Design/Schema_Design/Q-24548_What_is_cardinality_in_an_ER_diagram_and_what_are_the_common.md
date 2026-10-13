---
type: "practice"
difficulty: "Basic"
---

# What is cardinality in an ER diagram and what are the common cardinality types?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ER Modeling Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/ER_Modeling_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cardinality in Entity-Relationship (ER) diagrams represents the relationship between entities—specifically, how many instances of one entity correspond to a single instance of another entity. Common cardinality types include 1-to-1, 1-to-many, and many-to-many relationships. Understanding these is crucial for designing efficient and normalized database schemas.

To break down what the interviewer might be probing:

You need to define cardinality.
Explain the different cardinality types.
Provide examples or real-world scenarios for each type.
Discuss any common mistakes or nuances in understanding cardinality.

Now, let’s explore this step by step:

Definition of Cardinality: Cardinality defines the number of relationships between entities. It's a measure of how one entity is related to another and is fundamental when designing database schemas.

Cardinality Types:

1-to-1 (One-to-One): This type of relationship exists when each instance of Entity A can relate to exactly one instance of Entity B, and vice versa. For example, a user has only one profile on a social media platform.

SQL
-- Example in SQL
CREATE TABLE User (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE Profile (
    id INT PRIMARY KEY,
    bio TEXT,
    FOREIGN KEY (id) REFERENCES User(id)
);

1-to-Many (One-to-Many): This type of relationship exists when one instance of Entity A can relate to multiple instances of Entity B, but each instance of Entity B is related to only one instance of Entity A. For example, a user can have many posts on a blog.

SQL
-- Example in SQL
CREATE TABLE User (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE Post (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

Many-to-Many (Many-to-Many): This type of relationship exists when multiple instances of Entity A can relate to many instances of Entity B, and vice versa. For example, a student can be enrolled in multiple courses, and a course can have multiple students.

SQL
-- Example in SQL
CREATE TABLE Student (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Course (
    id INT PRIMARY KEY,
    title VARCHAR(255)
);

CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Student(id),
    FOREIGN KEY (course_id) REFERENCES Course(id)
);

Real-World Examples:

A 1-to-1 relationship can be seen in a library management system where each book has one unique barcode and vice versa.
A 1-to-many relationship can be seen in a customer order system, where multiple orders can belong to the same customer, but each order belongs only to one customer.
A many-to-many relationship is common in social networks where multiple users can follow each other.

#### Common Mistakes
and Nuances:

Misunderstanding 1-to-0..1 relationships as 1-to-1, which can lead to redundant data storage.
Not considering normalization when designing schemas might result in performance issues or redundancy.
Overusing many-to-many relationships unnecessarily complicates the schema without adding significant value.

**Comparison:**

| User         | Profile         | Post            |
|--------------|----------------|----------------|
| id | username | bio          | user_id       | title      | content               | user_id   |
| 1 | alice    | About Alice  | 1             | First Post | Hello World!          | 1         |
|                 |               |               |              | Second Post | Another Post          | 1         |
| Student | Course | Enrollment       |
|---------|--------|-----------------|
| id      | name   | title           | student_id | course_id |
| 1       | Alice  | Mathematics     | 1         | 1         |

**Code:**

```sql
-- Example SQL for 1-to-1 relationship
CREATE TABLE User (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE Profile (
    id INT PRIMARY KEY,
    bio TEXT,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES User(id)
);

-- Example SQL for 1-to-many relationship
CREATE TABLE User (
    id INT PRIMARY KEY,
    username VARCHAR(255)
);

CREATE TABLE Post (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES User(id)
);
```

**Example input / output:**

```text
Input:
- User(id=1, username='alice')
- Post(id=1, title='First Post', content='Hello World!', user_id=1)
- Post(id=2, title='Second Post', content='Another Post', user_id=1)

Output:
- User 1 has two posts: 'First Post' and 'Second Post'.
```

**Data:**

```sql
-- Sample Data for 1-to-Many Relationship
INSERT INTO User (id, username) VALUES (1, 'alice');
INSERT INTO Post (id, title, content, user_id) VALUES (1, 'First Post', 'Hello World!', 1);
INSERT INTO Post (id, title, content, user_id) VALUES (2, 'Second Post', 'Another Post', 1);

-- Sample Data for Many-to-Many Relationship
CREATE TABLE Student (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Course (
    id INT PRIMARY KEY,
    title VARCHAR(255)
);

CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    FOREIGN KEY (student_id) REFERENCES Student(id),
    FOREIGN KEY (course_id) REFERENCES Course(id)
);

INSERT INTO Student (id, name) VALUES (1, 'Alice');
INSERT INTO Course (id, title) VALUES (1, 'Mathematics');

-- Example enrollment
INSERT INTO Enrollment (student_id, course_id) VALUES (1, 1);
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD
3. U(User) --> P[Profile] (One-to-One)
4. U --> M[Post] (One-to-Many)
5. S(Student) --> C(Course) (Many-to-Many)

