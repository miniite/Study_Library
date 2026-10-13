---
type: "practice"
difficulty: "Intermediate"
---

# Design the rider's live-map frontend showing the assigned driver moving toward them — how do you smooth GPS jitter, handle dropped location updates, and keep the ETA in sync?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a rider's live-map frontend that smoothly shows the driver’s progress to their location while handling GPS jitter and dropped location updates, we need to focus on three main aspects: smoothing the map updates, handling intermittent data, and maintaining accurate ETA synchronization.

#### Triaging the Question

The interviewer is probing your ability to handle real-world scenarios like GPS signal issues, which are common in mobile applications. You'll be expected to demonstrate an understanding of UI/UX principles, state management, and possibly server-side logic if relevant to the design.

#### First-Principles Solution
Smoothing GPS Jitter:
Implement a moving average filter or exponential smoothing.
Use inter-frame interpolation between consecutive location updates.
Handling Dropped Location Updates:
Maintain a history of locations and use this to predict the current position.
Utilize server-side heartbeat mechanisms to ensure the latest known position is displayed.
Syncing ETA:
Calculate ETA based on historical speeds or average speeds from the route.
Ensure ETA updates are consistent with any changes in the driver's route.
Detailed Solution
1. Smoothing GPS Jitter

To smooth out the GPS jitter, we can use a simple moving average filter. This involves averaging the latest N location points to get a more stable position:

JAVASCRIPT
function smoothLocation(locations) {

#### cons
t filteredLocations = [];
    for (let i = 0; i < locations.length - 1; i++) {
        let totalLat = 0, totalLng = 0;
        for (let j = 0; j <= i; j++) {
            totalLat += locations[j].lat;
            totalLng += locations[j].lng;
        }
        filteredLocations.push({
            lat: totalLat / (i + 1),
            lng: totalLng / (i + 1)
        });
    }
    return filteredLocations;
}
2. Handling Dropped Location Updates

To handle dropped location updates, we can maintain a queue of the last few known positions and use these to estimate the current position when new data is missing:

JAVASCRIPT
let locationQueue = [];

function updateLocation(newLocation) {
    if (locationQueue.length < 5) {
        locationQueue.push(newLocation);
    } else {
        locationQueue.shift();
        locationQueue.push(newLocation);
    }
}

function getLastKnownLocation() {
    return locationQueue[0];
}
3. Syncing ETA

We can calculate the ETA based on the driver’s current speed and distance to the destination. Historical data can be used to improve accuracy:

JAVASCRIPT
let lastUpdateTime = new Date();

#### cons
t speedHistory = [];
let totalDistanceToDestination = 10; // Example in km

function updateETA(currentLocation) {

#### cons
t timeNow = new Date();

#### cons
t deltaTime = (timeNow - lastUpdateTime) / 1000;
    lastUpdateTime = timeNow;

    let currentSpeed = calculateSpeed(currentLocation, getLastKnownLocation());
    speedHistory.push({ time: timeNow, speed: currentSpeed });

    if (speedHistory.length > 5) {
        speedHistory.shift();
    }

#### cons
t averageSpeed = speedHistory.reduce((acc, curr) => acc + curr.speed, 0) / speedHistory.length;

#### cons
t remainingTime = totalDistanceToDestination / averageSpeed;
    return new Date(timeNow.getTime() + (remainingTime * 1000));
}

#### Closing Summary

In summary, to handle GPS jitter and dropped location updates in a live-map frontend, we need to implement smoothing techniques like moving averages or interpolation. We also need to maintain a history of locations for better prediction when new data is missing. Finally, maintaining an accurate ETA requires historical speed data and real-time distance calculations.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the importance of server-side synchronization can lead to inconsistent displays.

#### Follow-Up
Discussing how to implement heartbeat mechanisms from the client to the server for reliable updates.

**Comparison:**

```
Time (ms)	Speed (km/h)
1634501000000	15
1634501010000	17
1634501020000	18
```

**Code:**

```javascript
function smoothLocation(locations) {
    const filteredLocations = [];
    for (let i = 0; i < locations.length - 1; i++) {
        let totalLat = 0, totalLng = 0;
        for (let j = 0; j <= i; j++) {
            totalLat += locations[j].lat;
            totalLng += locations[j].lng;
        }
        filteredLocations.push({
            lat: totalLat / (i + 1),
            lng: totalLng / (i + 1)
        });
    }
    return filteredLocations;
}

function updateLocation(newLocation) {
    if (locationQueue.length < 5) {
        locationQueue.push(newLocation);
    } else {
        locationQueue.shift();
        locationQueue.push(newLocation);
    }
}

function getLastKnownLocation() {
    return locationQueue[0];
}

let lastUpdateTime = new Date();
const speedHistory = [];
let totalDistanceToDestination = 10; // Example in km

function updateETA(currentLocation) {
    const timeNow = new Date();
    const deltaTime = (timeNow - lastUpdateTime) / 1000;
    lastUpdateTime = timeNow;

    let currentSpeed = calculateSpeed(currentLocation, getLastKnownLocation());
    speedHistory.push({ time: timeNow, speed: currentSpeed });

    if (speedHistory.length > 5) {
        speedHistory.shift();
    }

    const averageSpeed = speedHistory.reduce((acc, curr) => acc + curr.speed, 0) / speedHistory.length;

    const remainingTime = totalDistanceToDestination / averageSpeed;
    return new Date(timeNow.getTime() + (remainingTime * 1000));
}
```

**Example input / output:**

```text
Input:
- Location updates: [ {"lat": 37.422, "lng": -122.145}, {"lat": 37.421, "lng": -122.146} ]
- Current time: 1634501010000
Output:
- Smoothed location: [ {"lat": 37.4215, "lng": -122.1455} ]
- Last known location: { lat: 37.421, lng: -122.146 }
- ETA: 2021-10-18T09:43:30Z
```

**Data:**

```json
{
  "locationQueue": [
    {"lat": 37.422, "lng": -122.145},
    {"lat": 37.421, "lng": -122.146},
    {"lat": 37.420, "lng": -122.147},
    {"lat": 37.419, "lng": -122.148},
    {"lat": 37.418, "lng": -122.149}
  ],
  "speedHistory": [
    {"time": 1634501000000, "speed": 15},
    {"time": 1634501010000, "speed": 17},
    {"time": 1634501020000, "speed": 18}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Location Update
4. Is Queue Full?
5. Add to Queue, Shift Oldest
6. Add New Location to End ofQueue
7. Get Last Known Location
8. Calculate Speed History
9. Update ETA Based on AvgSpeed and Distance toDestination
10. Smooth Location usingMoving Average

