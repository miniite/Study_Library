---
type: "practice"
difficulty: "Deep"
---

# How would you design a poll/survey model so adding a new question type (ranking, matrix) later doesn't require touching the scoring, storage, and rendering code all at once—where's the polymorphism seam?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle this LLD challenge, we'll design a poll/survey model that decouples different aspects of handling questions (like scoring, storage, and rendering) from their implementations. The key idea is to use polymorphism and interfaces/seams to ensure that adding new question types in the future doesn't necessitate modifying existing code.

#### Triaging the Question

The core of this question is about designing a system where adding new question types—such as ranking or matrix questions—is decoupled from the scoring, storage, and rendering logic. This requires a well-defined interface for each operation (scoring, storage, rendering) that can be implemented by various question types.

#### Assumptions
We are dealing with multiple question types: text input, radio buttons, checkboxes, ranking, matrix, etc.
Each question type has its own specific scoring logic, storage requirements, and rendering needs.
The system must support different ways of interacting with these questions (e.g., frontend requests to backend for data).

#### First-Principles Solution
Define Interfaces: Create interfaces for each operation that different question types need to implement.
Implement Question Types: Each new question type will inherit from the appropriate base class or interface and implement the required logic.
Register Question Factories: Use a factory pattern to instantiate questions based on their type, ensuring the system can handle any new type without modification.
Step-by-Step Solution

Define Interfaces for Operations

ScoreQuestion: Interface defining methods like scoreAnswer().
StoreQuestion: Interface defining methods like storeAnswer().
RenderQuestion: Interface defining methods like render(), which can return HTML or other rendering formats.

Base Class for Questions

JAVA
public abstract class Question {
    private String id;
    private String type;

    // Constructor, getters and setters

    public void setScoringLogic(ScoreQuestion scoringLogic) { /* ... */ }
    public void setStorageLogic(StoreQuestion storageLogic) { /* ... */ }
    public void setRenderingLogic(RenderQuestion renderingLogic) { /* ... */ }

    protected abstract String getType();
}

Concrete Question Types

TextQuestion: Implements ScoreQuestion, StoreQuestion, and RenderQuestion.
RankingQuestion: Extends Question and implements the appropriate interfaces.
JAVA
public class RankingQuestion extends Question implements ScoreQuestion, StoreQuestion, RenderQuestion {
    // Implementation of scoreAnswer(), storeAnswer(), render()
}

Factory for Instantiating Questions

QuestionFactory to register different question types and their specific implementations.

Registering and Using Factories

JAVA
public class QuestionFactory {
    private static final Map<String, Class<? extends Question>> QUESTION_FACTORIES = new HashMap<>();

    static {
        QUESTION_FACTORIES.put("text", TextQuestion.class);
        QUESTION_FACTORIES.put("ranking", RankingQuestion.class);
        // Add more question types here.
    }

    public static Question getQuestion(String type) throws InstantiationException, IllegalAccessException {
        Class<? extends Question> questionClass = QUESTION_FACTORIES.get(type);
        return questionClass.newInstance();
    }
}

Rendering and Scoring Logic

Use the appropriate logic based on the question’s type when rendering or scoring.
JAVA
public class SurveyRenderer {
    private List<Question> questions;

    public SurveyRenderer(List<Question> questions) {
        this.questions = questions;
    }

    public String render() {
        StringBuilder sb = new StringBuilder();
        for (Question q : questions) {
            sb.append(q.render());
        }
        return sb.toString();
    }

    // Similar methods for scoring, storing answers
}

#### Closing Summary
and Common Mistakes/Follow-Ups

This design ensures that adding a new question type doesn’t require touching the existing scoring, storage, or rendering code. By using polymorphism and interfaces, we can maintain a clean separation of concerns.

#### Common Mistakes

Overlooking the need for an abstract base class or interface.
Failing to define clear interfaces for each operation (scoring, storing, rendering).
Not properly registering question types in the factory.

#### Follow-Ups

How would you handle validation and error checking?
What about internationalization support for different question types?
How do you ensure performance under high traffic with many concurrent users?

**Comparison:**

```
Operation	Interface	Description
Scoring	ScoreQuestion	Implement scoring logic for answers
Storing	StoreQuestion	Store user responses
Rendering	RenderQuestion	Render question to UI
```

**Code:**

```java
public abstract class Question {
    private String id;
    private String type;

    // Constructor, getters and setters

    public void setScoringLogic(ScoreQuestion scoringLogic) { /* ... */ }
    public void setStorageLogic(StoreQuestion storageLogic) { /* ... */ }
    public void setRenderingLogic(RenderQuestion renderingLogic) { /* ... */ }

    protected abstract String getType();
}

public class RankingQuestion extends Question implements ScoreQuestion, StoreQuestion, RenderQuestion {
    // Implementation of scoreAnswer(), storeAnswer(), render()
}
```

**Example input / output:**

```text
Input:
- Question type: "ranking"
- Options: ["A", "B", "C"]
- Selected option: "B"

Output:
- Rendered HTML for a ranking question with selected option B.
```

**Data:**

```json
{
  "id": "q1",
  "type": "ranking",
  "options": ["A", "B", "C"],
  "selectedOption": "B"
}
```

**Flow:**

1. Yes
2. No
3. User Input
4. Is it a new question type?
5. Register Question Type
6. Create Question Object UsingFactory
7. Set Scoring Logic, StorageLogic, Rendering Logic
8. Render/Score/Store Answersas Needed

