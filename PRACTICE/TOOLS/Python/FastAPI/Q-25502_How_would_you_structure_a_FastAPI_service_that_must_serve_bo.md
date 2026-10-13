---
type: "practice"
difficulty: "Deep"
---

# How would you structure a FastAPI service that must serve both a real-time single-record inference endpoint and a large asynchronous batch scoring job that shares the same model and preprocessing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of serving both real-time single-record inference and large asynchronous batch scoring jobs using FastAPI, we need to design a robust and scalable service architecture. The key is to separate concerns between synchronous and asynchronous processing while ensuring that the shared model and preprocessing logic are efficiently managed.

The primary question being probed here is: How do you structure an API service in FastAPI to handle both types of workloads—real-time requests and batch jobs—and ensure they share resources effectively? We'll assume a couple of key assumptions:

The real-time endpoint will handle individual inference requests.
Batch scoring jobs will run asynchronously, processing large datasets.

First-principles reasoning dictates that we need to create two distinct FastAPI endpoints: one for the synchronous real-time inference and another for the asynchronous batch job. For efficient model management, we'll use a shared instance of the model and preprocessing pipeline. Let's walk through the solution step by step.

