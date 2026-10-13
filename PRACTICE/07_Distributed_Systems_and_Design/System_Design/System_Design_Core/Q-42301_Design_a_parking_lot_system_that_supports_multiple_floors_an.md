---
type: "practice"
difficulty: "Basic"
---

# Design a parking lot system that supports multiple floors and vehicle sizes.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Order Tracking Design](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Order_Tracking_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The problem requires designing a parking lot system that can handle multiple floors with different vehicle sizes, such as compact cars, SUVs, and motorcycles. We need to ensure efficient space utilization and easy vehicle management.

#### Triage

The interviewer is probing your understanding of how to design a scalable and maintainable system for managing a multi-floor parking lot. The key aspects include:

Floor Management: How you will manage different levels within the parking lot.
Vehicle Size Consideration: Ensuring that vehicles are parked in appropriate spaces based on their size.
Efficient Space Utilization: Maximizing space usage while ensuring easy vehicle access and exit.
Fault Tolerance and Scalability: The system should be robust and scalable to handle potential growth.

#### First-Principles Solution

High-Level Architecture:

We will design the parking lot as a multi-tier structure with multiple floors, each having its own sections for different vehicle types.
A central control system will manage bookings, space allocation, and user interactions.

Components:

Parking Floors: Each floor will have distinct sections (e.g., compact, mid-size, SUV) to accommodate different vehicle sizes.
Vehicles: Different vehicles will occupy specific spaces based on their size.
User Interface: A mobile app or web interface for users to book parking spots.
Central Control System: Manages the entire lot, including real-time monitoring and booking.

Detailed Design:

Database Schema:
SQL
CREATE TABLE Floor (
  id INT PRIMARY KEY,
  level VARCHAR(50)
);

CREATE TABLE ParkingSpot (
  id INT PRIMARY KEY,
  floor_id INT,
  vehicle_size VARCHAR(50),
  status ENUM('available', 'occupied') DEFAULT 'available',
  FOREIGN KEY (floor_id) REFERENCES Floor(id)
);

CREATE TABLE Booking (
  id INT PRIMARY KEY,
  spot_id INT,
  user_id INT,
  start_time TIMESTAMP,
  end_time TIMESTAMP,
  FOREIGN KEY (spot_id) REFERENCES ParkingSpot(id),
  FOREIGN KEY (user_id) REFERENCES User(id)
);
User Interface:
HTML
<div>
  <h1>Parking Lot Booking</h1>
  <form id="bookingForm">
    <label>Vehicle Size:</label>
    <select id="vehicleSize">
      <option value="compact">Compact</option>
      <option value="mid-size">Mid-Size</option>
      <option value="SUV">SUV</option>
    </select><br>
    <!-- Other form fields -->
  </form>
</div>
Central Control System:
JAVASCRIPT
function parkVehicle(vehicleSize) {
  // Logic to find available spot and update database
}

function exitParking() {
  // Logic to remove booking and update spot status
}

Edge Cases:

What happens if all spots of a certain size are occupied?
How will the system handle temporary parking or emergencies?

#### Closing Summary

The proposed design ensures efficient management of multiple floors and different vehicle sizes in a parking lot system. By using a database to manage floor and spot statuses, we can efficiently allocate spaces and track bookings. The central control system provides robust functionality for both users and administrators.

#### Common Mistakes/Follow-Ups
Not Considering Edge Cases: Failing to handle scenarios where no suitable spots are available.
Ignoring User Experience: Not providing a seamless user interface that ensures ease of use.
Lack of Scalability: Designing the system without considering future growth, such as adding more floors or vehicle types.

**Comparison:**

```
Floor ID	Level
1	Ground Level
2	First Floor
3	Second Floor
Spot ID	Floor ID	Vehicle Size	Status
101	1	compact	available
102	1	mid-size	occupied
103	1	SUV	available
201	2	compact	available
202	2	mid-size	available
203	2	SUV	occupied
Booking ID	Spot ID	User ID	Start Time	End Time
1	101	1	2023-10-01 10:00:00	2023-10-01 16:00:00
```

**Code:**

```

CREATE TABLE Floor ( id INT PRIMARY KEY, level VARCHAR(50) );

CREATE TABLE ParkingSpot ( id INT PRIMARY KEY, floor_id INT, vehicle_size VARCHAR(50), status ENUM('available', 'occupied') DEFAULT 'available', FOREIGN KEY (floor_id) REFERENCES Floor(id) );

CREATE TABLE Booking ( id INT PRIMARY KEY, spot_id INT, user_id INT, start_time TIMESTAMP, end_time TIMESTAMP, FOREIGN KEY (spot_id) REFERENCES ParkingSpot(id), FOREIGN KEY (user_id) REFERENCES User(id) );

CODE
Copy

```javascript
function parkVehicle(vehicleSize) {
  const availableSpots = getAvailableSpots(vehicleSize);
  if (availableSpots.length > 0) {
    const spot = availableSpots[0];
    updateDatabase(spot, 'occupied');
    return spot;
  }
  return null; // No available spots
}

function exitParking() {
  const currentSpot = getCurrentSpot(); // Assume this function is defined elsewhere
  if (currentSpot) {
    updateDatabase(currentSpot, 'available');
    removeBooking(currentSpot);
  } else {
    console.log("No active booking found.");
  }
}
```

**Example input / output:**

```text
Input:
User requests a compact parking spot.

Output:
System checks the ground level and finds an available compact spot (101).
Spot is updated to 'occupied'.
Booking record is created for the user.
```

**Data:**

```sql
-- Sample Data
INSERT INTO Floor (id, level) VALUES (1, 'Ground Level'), (2, 'First Floor'), (3, 'Second Floor');
INSERT INTO ParkingSpot (id, floor_id, vehicle_size, status) VALUES
  (101, 1, 'compact', 'available'),
  (102, 1, 'mid-size', 'occupied'),
  (103, 1, 'SUV', 'available'),
  (201, 2, 'compact', 'available'),
  (202, 2, 'mid-size', 'available'),
  (203, 2, 'SUV', 'occupied');
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. User requests a compact spot
7. Is the ground level available?
8. Park vehicle at 101
9. Check first floor availability
10. Is the first floor available?
11. Park vehicle at a spot on thefirst floor
12. Check second floor availability
13. Is the second floor available?
14. Park vehicle at a spot on thesecond floor
15. Update database to 'occupied'
16. Update database to 'occupied'
17. Update database to 'occupied'

