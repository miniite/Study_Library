---
type: "practice"
difficulty: "Deep"
---

# How do you make FastAPI startup fast and safe when it depends on a DB, Redis, and a config service that may be temporarily unavailable, without deploying a pod that can never become ready?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure FastAPI starts quickly and safely while handling dependencies like a database (DB), Redis, and a config service, we need to implement a robust startup routine that checks these services' availability before starting the application. This involves using startup and shutdown events in FastAPI, as well as setting up retry mechanisms and fallback strategies. The goal is to prevent the app from entering an "unhealthy" state if dependencies are not available but also ensure it doesn't remain indefinitely in a "not ready" status.

The key steps include:

Implementing health checks for each dependency.
Using FastAPI's on_startup and on_shutdown hooks.
Configuring retries and fallbacks to handle transient issues gracefully.
Ensuring proper logging and monitoring.

**Comparison:**

```
Service	Check Function	Delay (s)
DB	check_db_connection	1
Redis	check_redis_connection	1
Config	check_config_service	1
```

**Code:**

```python
from fastapi import FastAPI, HTTPException
import asyncio

app = FastAPI()

async def check_db_connection():
    try:
        # Simulate DB connection check
        await asyncio.sleep(1)  # Replace with actual DB health check logic
        print("Database is ready")
        return True
    except Exception as e:
        print(f"DB check failed: {e}")
        return False

async def check_redis_connection():
    try:
        # Simulate Redis connection check
        await asyncio.sleep(1)  # Replace with actual Redis health check logic
        print("Redis is ready")
        return True
    except Exception as e:
        print(f"Redis check failed: {e}")
        return False

async def check_config_service():
    try:
        # Simulate config service check
        await asyncio.sleep(1)  # Replace with actual config service health check logic
        print("Config service is ready")
        return True
    except Exception as e:
        print(f"Config service check failed: {e}")
        return False

@app.on_event("startup")
async def on_startup():
    db_ready = await check_db_connection()
    redis_ready = await check_redis_connection()
    config_ready = await check_config_service()

    if not (db_ready and redis_ready and config_ready):
        raise HTTPException(status_code=503, detail="Dependency services are not ready")

@app.on_event("shutdown")
async def on_shutdown():
    print("Shutting down gracefully...")
```

**Example input / output:**

```text
On startup:
1. `check_db_connection` is called.
2. Simulated delay, then "Database is ready" printed if successful, or exception logged otherwise.
3. `check_redis_connection` is called.
4. Simulated delay, then "Redis is ready" printed if successful, or exception logged otherwise.
5. `check_config_service` is called.
6. Simulated delay, then "Config service is ready" printed if successful, or exception logged otherwise.

If any of the checks fail:
- HTTP 503 error is raised with message "Dependency services are not ready".
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. No
5. No
6. No
7. Yes
8. Start
9. Check DB
10. DB Ready?
11. Check Redis
12. Redis Ready?
13. Check Config
14. Config Ready?
15. Start Application
16. Retry DB Check (timeout)
17. Retry Redis Check (timeout)
18. Retry Config Check (timeout)
19. Initial Checks Fail

