---
type: "practice"
difficulty: "Deep"
---

# Describe how you would model gaming telemetry so player, match, and session facts share conformed dimensions for both live-ops and monetization analytics.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively model gaming telemetry data, we need to create a robust data model that supports both live-operations (live-ops) and monetization analytics while ensuring conformed dimensions are shared across player, match, and session facts. The star schema is ideal for this scenario due to its simplicity and ability to facilitate fast query performance. Here’s how I would approach the modeling:

Firstly, we need to understand the key entities involved: players, matches, sessions, and telemetry events (which can include game events such as player actions, match outcomes, or session details). The goal is to ensure that these entities are related through a set of common dimensions such as date, time, location, device type, etc.

**Comparison:**

| Dimension Table Columns | Description                    |
|------------------------|-------------------------------|
| id                     | Unique identifier for each dimension row |
| date                   | Date of the event              |
| time                   | Time of the event               |
| location_id            | Location where the event occurred  |
| device_type            | Type of device used             |
| TelemetryEvent Table Columns | Description                      |
|-----------------------------|---------------------------------|
| id                          | Unique identifier for each telemetry event       |
| player_id                   | Player who performed the action     |
| match_id                    | Match in which the action occurred  |
| session_id                  | Session during which the action occurred    |
| event_type                  | Type of event (e.g., Login, Purchase)      |
| timestamp                   | Timestamp when the event occurred       |
| metadata                    | Additional information about the event        |

**Code:**

```
MERMAID
Copy
erDiagram
    Player ||--o{ Dimension : has many
    Match ||--o{ Dimension : has many
    Session ||--o{ Dimension : has many

    Player ||--| { TelemetryEvent
    Match ||--| { TelemetryEvent
    Session ||--| { TelemetryEvent

    Dimension {
        + id
        + date
        + time
        + location_id
        + device_type
    }

    TelemetryEvent {
        + id
        + player_id
        + match_id
        + session_id
        + event_type
        + timestamp
        + metadata
    }
```

**Example input / output:**

```

An example input for a player login event with a session and match could be:

JSON
Copy
{
  "player_id": 5,
  "match_id": 7,
  "session_id": 8,
  "event_type": "Login",
  "timestamp": "2023-10-01 14:30:00",
  "metadata": { "device_info": "Android" }
}

This event would be inserted into the TelemetryEvents table, and related dimensions (such as date, time, location) from the common dimension tables.
```

**Data:**

```json
{
  "Dimensions": [
    { "id": 1, "date": "2023-10-01", "time": "14:30:00", "location_id": "SF", "device_type": "Android" },
    { "id": 2, "date": "2023-10-02", "time": "09:05:30", "location_id": "NYC", "device_type": "iOS" }
  ],
  "TelemetryEvents": [
    {
      "id": 1,
      "player_id": 5,
      "match_id": 7,
      "session_id": 8,
      "event_type": "Login",
      "timestamp": "2023-10-01 14:30:00",
      "metadata": { "device_info": "Android" }
    },
    {
      "id": 2,
      "player_id": 5,
      "match_id": 7,
      "session_id": 8,
      "event_type": "ItemPurchase",
      "timestamp": "2023-10-01 14:31:00",
      "metadata": { "item_id": 101, "currency": "gold" }
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Player
4. Does a new session start?
5. Session
6. TelemetryEvent
7. Merge with commondimensions
8. TeelemetryEvent table insert

