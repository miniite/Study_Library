---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a well-known port and a registered port?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NAT and Port Exhaustion](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/NAT_and_Port_Exhaustion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In networking fundamentals, particularly in TCP/IP, the concept of "well-known" vs "registered" ports plays a crucial role in understanding how different services communicate over the network. Well-known ports are those that are reserved by IANA (Internet Assigned Numbers Authority) for specific services and applications, while registered ports can be used freely by any application or service.

To answer your question directly: A well-known port is a specific port number between 0 and 1023 that has been reserved by the Internet Assigned Numbers Authority (IANA). These are typically used by system-level processes like HTTP servers, email services, and other standard network services. In contrast, registered ports range from 49152 to 65535 and can be assigned freely for custom or non-standard applications.

**Code:**

```python
# Example of how well-known and registered ports are used in a Python script
import socket

def start_http_server():
    # Well-known port: HTTP service uses port 80 by default
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind(('localhost', 80))

    try:
        server_socket.listen()
        print("HTTP Server started on port 80")
    except OSError as e:
        print(f"Failed to bind to well-known port: {e}")
    finally:
        server_socket.close()

def start_custom_app():
    # Registered port: can be any number in the range [49152, 65535]
    custom_port = 50000
    app_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    app_socket.bind(('localhost', custom_port))

    try:
        app_socket.listen()
        print(f"Custom App started on port {custom_port}")
    except OSError as e:
        print(f"Failed to bind to registered port: {e}")
    finally:
        app_socket.close()
```

**Example input / output:**

```

Input:

To start an HTTP server on port 80 (well-known)
To start a custom application on a registered port, for example, 50000

Output:

PLAINTEXT
Copy
HTTP Server started on port 80
Custom App started on port 50000
```

**Data:**

```json
{
  "wellKnownPorts": {
    "http": 80,
    "https": 443,
    "ftp": 21,
    "ssh": 22,
    "smtp": 25,
    "pop3": 110,
    "imap": 143
  },
  "registeredPortsRange": {
    "start": 49152,
    "end": 65535
  }
}
```

