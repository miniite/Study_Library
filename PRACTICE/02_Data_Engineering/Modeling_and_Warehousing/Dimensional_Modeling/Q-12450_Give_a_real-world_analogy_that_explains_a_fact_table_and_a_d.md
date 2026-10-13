---
type: "practice"
difficulty: "Basic"
---

# Give a real-world analogy that explains a fact table and a dimension table to a beginner.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Imagine you're planning a big birthday party, and you want to keep track of all the details like guests, games, and food. In this scenario, we can think of the "fact table" as the main summary sheet that captures the total number of guests who played each game or ate each type of food. Each row in the fact table represents a specific event, such as "20 guests played hide-and-seek," or "50 people enjoyed pizza." This is like our core data point or measurement.

On the other hand, dimension tables are like separate sheets that store more detailed information about your party elements. For example:

A Guest table might include details like their names, ages, and what games they played.
A Game table could list all the games available, with a start and end time for each one.
An Item table (for food) would describe each type of food served.

Now, let's break this down step by step:

Fact Table: Think of it as a summary sheet that captures the key measurements or facts about your party events.
Dimension Tables: These are detailed sheets that provide context to understand what's captured in the fact table.

This analogy helps us understand why we use both fact and dimension tables together: The fact table gives us the "what" (who played which games), while the dimension tables give us the "why" and "how" (details about each guest, game, and food item).

**Comparison:**

| Fact_Party | Guest       | Game        | Food         |
|------------|-------------|-------------|--------------|
| PartyID    | GuestID     | GameID      | FoodID       | NumberOfParticipants |
| 1          | 1           | 1           | 1            | 20                   |
| Dim_Guests | Name   | Age | Gender |
|------------|--------|-----|--------|
| 1          | Alice  | 12  | Female |
| Dim_Games | GameName         | StartTime | EndTime      |
|-----------|------------------|-----------|--------------|
| 1         | Hide and Seek    | 2023-10-05| 14:00:00     |
| Dim_FoodItems | ItemName   | Type |
|---------------|------------|------|
| 1             | Pizza      | Main Course |

**Code:**

```sql
-- Example SQL for a simplified data model

CREATE TABLE Fact_Party (
    PartyID INT,
    GuestID INT,
    GameID INT,
    FoodID INT,
    NumberOfParticipants INT,
    Time戳 DATETIME,
    PRIMARY KEY (PartyID, GuestID, GameID, FoodID)
);

CREATE TABLE Dim_Guests (
    GuestID INT PRIMARY KEY,
    Name VARCHAR(100),
    Age INT,
    Gender VARCHAR(10)
);

CREATE TABLE Dim_Games (
    GameID INT PRIMARY KEY,
    GameName VARCHAR(50),
    StartTime DATETIME,
    EndTime DATETIME
);

CREATE TABLE Dim_FoodItems (
    FoodID INT PRIMARY KEY,
    ItemName VARCHAR(50),
    Type VARCHAR(20)
);
```

**Example input / output:**

```text
Input:
- Fact_Party data: PartyID=1, GuestID=1, GameID=1, FoodID=1, NumberOfParticipants=20, Time戳="2023-10-05 14:00:00"

Output:
- Dim_Guests: {"GuestID": 1, "Name": "Alice", "Age": 12, "Gender": "Female"}
- Dim_Games: {"GameID": 1, "GameName": "Hide and Seek", "StartTime": "2023-10-05 14:00:00", "EndTime": "2023-10-05 15:00:00"}
- Dim_FoodItems: {"FoodID": 1, "ItemName": "Pizza", "Type": "Main Course"}
```

**Data:**

```json
{
  "Party": {
    "Guests": [{"Name": "Alice", "Age": 12, "Gender": "Female"}, {"Name": "Bob", "Age": 13, "Gender": "Male"}],
    "Games": [{"GameName": "Hide and Seek", "StartTime": "2023-10-05 14:00:00", "EndTime": "2023-10-05 15:00:00"}, {"GameName": "Treasure Hunt", "StartTime": "2023-10-05 16:00:00", "EndTime": "2023-10-05 17:00:00"}],
    "FoodItems": [{"ItemName": "Pizza", "Type": "Main Course"}, {"ItemName": "Cakes", "Type": "Dessert"}]
  }
}
```

**Flow:**

1. Fact Table
2. Guests
3. Games
4. Food Items
5. Dim_Guests
6. Dim_Games
7. Dim_FoodItems

