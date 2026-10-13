---
type: "practice"
difficulty: "Deep"
---

# How would you use an AI assistant to add graceful shutdown handling (drain connections, finish in-flight work) to a server?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Production Patterns](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Production_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system that incorporates an AI-assisted coding tool like Copilot or similar, one of the key challenges is ensuring robust and graceful shutdown handling. Graceful shutdown involves managing active connections, finishing in-flight work, and properly shutting down services to prevent data loss or application crashes. This task requires a careful balance between leveraging AI's capabilities while adhering to sound engineering practices.

In this scenario, we need to implement mechanisms for draining current requests, ensuring that ongoing tasks complete before stopping the service, and logging relevant information for troubleshooting. The core idea is to use the AI assistant to generate code snippets that handle these critical shutdown operations while maintaining a robust, maintainable solution.

**Code:**

```python
import asyncio

async def graceful_shutdown(app):
    print("Graceful shutdown initiated.")

    # Drain connections (close open connections and wait for completion)
    app['loop'].call_soon(app.shutdown)

    # Wait for all active requests to complete
    await app.shutdown()

    # Wait for all tasks to be done
    await asyncio.gather(*asyncio.all_tasks(app['loop']), return_exceptions=True)

    print("All connections drained and in-flight work completed.")

    # Clean up resources
    app.cleanup()

# Example usage within a FastAPI application
app = None  # Assume this is your FastAPI app object

@app.on_event('shutdown')
async def on_shutdown():
    await graceful_shutdown(app)
```

**Example input / output:**

```text
# Simulated input and output for demonstration purposes
import asyncio

async def request_handler(request):
    print(f"Request received at {request.path}")
    await asyncio.sleep(5)  # Simulate processing time
    return {"status": "ok"}

app = FastAPI()

@app.get('/test')
async def test():
    return await request_handler(request={})

# Assume the shutdown event is triggered by a signal or scheduled task

await graceful_shutdown(app)
```

**Flow:**

1. Yes
2. No
3. Graceful Shutdown Initiated
4. Are there active connections?
5. Drain Connections
6. Wait for Active Requests toComplete
7. Wait for All Tasks to be Done
8. Clean Up Resources and Log
9. Service Terminates Gracefully

