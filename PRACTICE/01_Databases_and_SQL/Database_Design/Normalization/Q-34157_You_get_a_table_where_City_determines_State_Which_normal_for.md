---
type: "practice"
difficulty: "Intermediate"
---

# You get a table where City determines State. Which normal form does that violate and how would you fix it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we first need to understand what normal forms are and their importance in database design. The issue described violates First Normal Form (1NF), which requires that a table should contain atomic values—meaning each cell must hold indivisible data. When City determines State, it implies that the same city can be associated with multiple states, leading to non-atomic values.

To fix this, we need to break down these dependencies and ensure that each column contains independent atomic data. Let's walk through the steps from first principles to arrive at a normalized solution.

**Comparison:**

| Original Data             | Normalized Data          |
|--------------------------|-------------------------|
| City         State        | City           State     |
| Chicago      [Illinois, Indiana] | Chicago   Illinois    |
| Los Angeles  California    | Los Angeles California |
| New York     New York      | New York   New York    |

**Code:**

```python
# Example code to illustrate the transformation
import pandas as pd

def normalize_state_city_mapping(df):
    # First, identify unique cities and states
    city_states = df.groupby('City')['State'].apply(list).reset_index()

    # Normalize by splitting each row for every state associated with a city
    normalized_data = []
    for index, row in city_states.iterrows():
        for state in row['State']:
            new_row = {'City': row['City'], 'State': state}
            normalized_data.append(new_row)

    # Convert to DataFrame and clean up duplicates
    normalized_df = pd.DataFrame(normalized_data).drop_duplicates()
    return normalized_df

# Sample data
data = {
    'City': ['Chicago', 'Los Angeles', 'New York'],
    'State': [['Illinois', 'Indiana'], ['California'], ['New York']]
}

df = pd.DataFrame(data)
normalized_df = normalize_state_city_mapping(df)
print(normalized_df)
```

**Example input / output:**

```text
# Sample input and output
Input:
| City      | State            |
|-----------|------------------|
| Chicago   | [Illinois, Indiana] |
| Los Angeles | California       |
| New York  | New York         |

Output after normalization:
| City      | State           |
|-----------|-----------------|
| Chicago   | Illinois        |
| Chicago   | Indiana         |
| Los Angeles | California     |
| New York  | New York        |
```

**Data:**

```json
{
  "data": {
    "City": ["Chicago", "Los Angeles", "New York"],
    "State": [["Illinois", "Indiana"], ["California"], ["New York"]]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is City mapped to multipleStates?
5. Normalize by splitting rows
6. Already in 1NF, no actionneeded
7. Create a new row for eachState- City combination
8. Remove duplicates if any
9. End

