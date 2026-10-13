---
type: "practice"
difficulty: "Deep"
---

# You must migrate a large sync Python (Flask/WSGI) GenAI service to async without a big-bang rewrite. How would you sequence it so both worlds interoperate safely and you can measure the latency/throughput win at each step?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a large, synchronous Python (Flask/WSGI) GenAI service to an asynchronous model without a big-bang rewrite, we need to carefully plan and sequence our approach. We'll start by identifying the key components of the current architecture, isolating them into manageable pieces, and gradually introducing asynchronous capabilities while ensuring backward compatibility. This process will involve understanding the existing service's dependencies, performance metrics, and critical sections of code that can safely be moved to async.

**Comparison:**

```
Synchronous vs Asynchronous Functionality	Sync Flask Route	Async Flask Route
Return type	Response object	Awaited response
Blocking behavior	Yes	No
Performance metric	Time taken	Latency
```

**Code:**

```python
# Example of a synchronous Flask route
@app.route('/predict', methods=['POST'])
def predict():
    data = request.json  # Synchronous call
    result = genai_model.predict(data)  # Sync model inference
    return jsonify(result)
```

**Example input / output:**

```text
# Example input and output for the '/predict' route before migration
Request:
POST /predict
Content-Type: application/json

{
  "inputs": [
    "What is the capital of France?"
  ]
}

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{"output": "Paris"}
```

**Data:**

```json
{
  "data": [
    {
      "input": "[\"What is the capital of France?\"]",
      "expected_output": {"output": "Paris"}
    },
    {
      "input": "[\"Translate 'hello' into Spanish\"]",
      "expected_output": {"output": "hola"}
    }
  ]
}
```

**Flow:**

1. Current Synchronous Service
2. Isolate Critical Components
3. Create Async Version ofIsolated Components
4. Parallelize AsynchronousFunctions
5. Integrate Async and Sync Code
6. Test Interoperability
7. Migrate Full Service Gradually

