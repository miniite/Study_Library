---
type: "practice"
difficulty: "Basic"
---

# Design a QR-code-based check-in system for an event.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Chat History Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Chat_History_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a QR-code-based check-in system for an event, we need to ensure that the solution is both robust and user-friendly while also considering factors like performance, security, and scalability. This design will focus on key components such as QR code generation, scanning, and backend processing.

TL;DR

The system involves generating unique QR codes for each attendee, which they scan upon arrival at the event. The backend processes these scans to update attendance records in real-time. We'll implement a scalable architecture with redundancy, security measures, and efficient data storage mechanisms.

#### Triage
Understanding the Question

The question is asking us to design a system that uses QR codes for checking attendees into an event. The key aspects to consider are:

Scalability: Handling a large number of attendees efficiently.
Security: Protecting sensitive attendee information.
Real-time Updates: Ensuring quick and accurate updates in the backend.

#### First Principles Solution

QR Code Generation:

Generate unique QR codes for each attendee with their registration details embedded within them.

Scanning Mechanism:

Attendees scan the QR code using a mobile app or web interface.

Backend Processing:

The backend processes the scanned data to update attendance records in real-time.
Detailed Steps
1. User Registration and QR Code Generation
Each attendee registers for the event with their personal information, such as name, email, and contact number.
A unique QR code is generated for each attendee during registration.
The QR code contains a serialized identifier corresponding to the attendee's record.
PYTHON
import qrcode

def generate_qr_code(attendee_id):
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    data = {'id': attendee_id}
    qr.add_data(data)
    qr.make(fit=True)
    img = qr.make_image(fill='black', back_color='white')
    return img
2. Scanning and Backend Processing
Attendees scan the QR code using a mobile app or web interface.
The scanner sends the serialized identifier to the backend server.
PYTHON
def process_scan(scan_data):
    attendee_id = scan_data['id']
    # Update attendance records in database
    update_attendance(attendee_id)
    return "Scan processed successfully"
3. Database Schema
A simple schema for storing attendee information and attendance status.
PLAINTEXT
Attendees:
| id | name | email       | phone          |
-----------------------------------------
| 1  | John | john@example.com | +1234567890   |

EventAttendance:
| id | attendee_id | event_id | checked_in_at     |
-------------------------------------------------------------------
| 1  | 1           | 1        | 2023-10-10 10:00:00 |
4. Backend Architecture
Frontend: Mobile app or web interface for scanning QR codes.
API Gateway: Handles incoming requests and routes them to the appropriate microservices.
Auth Server: Manages authentication and authorization.
Event Attendance Service: Processes scan data and updates attendance records.
Database: Stores attendee information and event attendance logs.
5. Security Measures
Use secure transmission protocols (HTTPS) for all interactions.
Implement rate limiting to prevent abuse.
Store sensitive data securely using encryption techniques.
Regularly audit the system for vulnerabilities.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, designing a QR-code-based check-in system requires careful consideration of scalability, security, and real-time updates. The backend processes must be efficient and reliable to handle high traffic during events.

#### Common mistakes
include:

Failing to implement proper rate limiting.
Not encrypting sensitive data.
Neglecting regular security audits.

#### Follow-ups
might involve discussing how to integrate with existing event management systems or exploring advanced features like real-time notifications for latecomers.

**Comparison:**

```
Component	Description
Auth Server	Manages authentication and authorization.
API Gateway	Routes requests to appropriate services.
Event Attendance	Processes scan data and updates records.
Database	Stores attendee info and attendance logs.
```

**Code:**

```python
import qrcode

def generate_qr_code(attendee_id):
    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )
    data = {'id': attendee_id}
    qr.add_data(data)
    qr.make(fit=True)
    img = qr.make_image(fill='black', back_color='white')
    return img

def process_scan(scan_data):
    attendee_id = scan_data['id']
    # Update attendance records in database
    update_attendance(attendee_id)
    return "Scan processed successfully"
```

**Example input / output:**

```text
Input: Scan data containing the QR code identifier (e.g., attendee_id = 1)
Output: "Scan processed successfully" and update attendance record in database.
```

**Data:**

```plaintext
Attendees:
| id | name | email       | phone          |
-----------------------------------------
| 1  | John | john@example.com | +1234567890   |

EventAttendance:
| id | attendee_id | event_id | checked_in_at     |
-------------------------------------------------------------------
| 1  | 1           | 1        | 2023-10-10 10:00:00 |
```

**Flow:**

1. Scan QR Code
2. API Gateway
3. Auth Server
4. Verify Token
5. Event Attendance Service
6. Update Attendance Record inDB
7. Success Response

