---
type: "practice"
difficulty: "Deep"
---

# Requirements arrive mid-project that your booking system must support both seats and time-slots (movies and salons); how do you refactor the model to share the core without a fragile if/else on 'type'?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to design a flexible booking system that can handle both seats (like in a movie theater) and time-slots (like in a salon). The key is to refactor our model so it's not tied to specific types like 'seat' or 'time-slot', but instead focuses on the common attributes they share. This approach will make the code more maintainable and less error-prone.

First, let's break down what we're dealing with. Both seats and time-slots have some core behaviors that are shared: booking, canceling, checking availability, etc. However, there are unique aspects to each: seats involve physical locations, while time-slots deal with temporal constraints. Our goal is to leverage commonality while accommodating these differences.

#### Triage

The interviewer is probing how we can design a flexible system that avoids the pitfalls of using a monolithic if/else structure. We need to identify the core attributes and behaviors that are shared and encapsulate them, then handle specific requirements through extensions or composition.

#### First-Principles Solution

Identify Core Attributes:

Both seats and time-slots have availability (whether they can be booked), id, and perhaps a name for identification.

Common Behaviors:

Booking, canceling, checking availability.

Shared vs. Specific Logic:

Use polymorphism or composition to handle common behaviors while allowing specific types to implement their unique requirements.

Model Refactoring:

Create a BookingEntity interface with methods like checkAvailability(), book(), and cancel().
Implement Seat and TimeSlot classes that both implement this interface but provide specific implementations for seat-related and time-slot-related behaviors.

Database Schema:

Ensure the database schema can support polymorphic relationships if using an ORM like Hibernate or a similar mechanism to map to different types of booking entities.
Detailed Steps
Define the Interface (if/abstract class):
JAVA
public interface BookingEntity {
    boolean checkAvailability();
    void book();
    void cancel();
}
Implement Specific Classes:
Seat Class:
JAVA
public class Seat implements BookingEntity {
    private int seatNumber;
    // Other seat-specific attributes

    @Override
    public boolean checkAvailability() {
        // Check if the seat is available in your database or system.
        return !isOccupied();
    }

    @Override
    public void book() {
        // Book the seat and mark it as occupied.
        markAsOccupied();
    }

    @Override
    public void cancel() {
        // Cancel the booking and mark it as free.
        unoccupySeat();
    }
}
TimeSlot Class:
JAVA
public class TimeSlot implements BookingEntity {
    private LocalDate date;
    private LocalTime startTime;

    @Override
    public boolean checkAvailability() {
        // Check if this time slot is available in your database or system.
        return !isBooked();
    }

    @Override
    public void book() {
        // Book the time slot and mark it as occupied.
        markAsOccupied();
    }

    @Override
    public void cancel() {
        // Cancel the booking and mark it as free.
        unoccupySlot();
    }
}
Database Schema:
SQL
CREATE TABLE BookingEntities (
    id INT PRIMARY KEY,
    entity_type VARCHAR(50) CHECK (entity_type IN ('seat', 'time_slot')),
    -- Other common fields like availability, name, etc.
);
Mapping with Polymorphism:

If using Hibernate or a similar ORM:

JAVA
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "entity_type", discriminatorType = DiscriminatorType.STRING)
public abstract class BookingEntity {
    @Id
    private Long id;

    // Common methods here

    public boolean checkAvailability() {
        return !isOccupied();
    }

    public void book() {
        markAsOccupied();
    }

    public void cancel() {
        unoccupy();
    }
}

@Entity
public class Seat extends BookingEntity {
    @Column(name = "seat_number")
    private int seatNumber;

    // Implement specific methods here
}

@Entity
public class TimeSlot extends BookingEntity {
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "start_time")
    private Date startTime;

    // Implement specific methods here
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using polymorphism or composition, we can share common logic while accommodating the unique requirements of seats and time-slots. This approach avoids fragile if/else structures and makes the system easier to maintain and extend.

#### Common Mistakes

Overly complex interfaces that lead to bloated code.
Not leveraging database features like polymorphic relationships effectively.

#### Follow-Ups

How would you handle additional booking entities in the future?
What are some potential performance optimizations for this system?

**Code:**

```java
public interface BookingEntity {
    boolean checkAvailability();
    void book();
    void cancel();
}

public class Seat implements BookingEntity {
    private int seatNumber;

    @Override
    public boolean checkAvailability() {
        return !isOccupied();
    }

    @Override
    public void book() {
        markAsOccupied();
    }

    @Override
    public void cancel() {
        unoccupySeat();
    }
}

public class TimeSlot implements BookingEntity {
    private LocalDate date;
    private LocalTime startTime;

    @Override
    public boolean checkAvailability() {
        return !isBooked();
    }

    @Override
    public void book() {
        markAsOccupied();
    }

    @Override
    public void cancel() {
        unoccupySlot();
    }
}
```

**Data:**

```sql
CREATE TABLE BookingEntities (
    id INT PRIMARY KEY,
    entity_type VARCHAR(50) CHECK (entity_type IN ('seat', 'time_slot')),
    -- Other common fields like availability, name, etc.
);
```

