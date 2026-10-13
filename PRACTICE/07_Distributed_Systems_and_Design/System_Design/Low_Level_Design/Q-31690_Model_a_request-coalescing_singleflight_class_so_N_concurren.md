---
type: "practice"
difficulty: "Deep"
---

# Model a request-coalescing/singleflight class so N concurrent misses for the same key trigger one downstream fetch; how do you handle the case where the in-flight fetch fails and callers must not cache the error?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a request-coalescing or singleflight mechanism, we aim to ensure that only one concurrent fetch is made for a given key, and all other calls are blocked until the initial fetch completes. This approach helps in efficiently managing resources and avoiding redundant requests. In the event of an error during the fetch, it's crucial that no cached errors are returned to any callers.

TL;DR

We will implement a RequestCoalescer class using concurrency control mechanisms like channels or semaphores. When a request is made for a key, we check if another request is already in flight. If not, we start the fetch and block all other requests until the result is available. In case of an error during fetching, we ensure that this error is propagated to all waiting callers without caching it.

#### Triaging the Question

The question asks us to design a RequestCoalescer class such that multiple concurrent fetches for the same key are handled by ensuring only one request is processed at a time. If the initial fetch fails, no other caller should receive an error response; instead, all callers should be informed about the failure.

#### Assumptions
The key is immutable and unique.
Each call to the RequestCoalescer has a timeout for blocking.
The result or error of the fetch is eventually consistent.

#### Solution
from First Principles
Data Structures: We need to maintain state for each request, tracking its status (pending, fetched, failed) and associated key.
Concurrency Control: Use channels or semaphores to control access and ensure only one concurrent fetch per key.
Error Handling: Manage errors such that they are not cached but propagated correctly.
Step-by-Step Solution
Key Storage: We use a map to store the state of each request based on its key.
Channel for Fetches: Use a channel to coordinate between fetch requests and the actual fetching logic.
Semaphore/Channel Control: Ensure that only one fetch is in progress at any time.
Request Coalescer
Key -> State Map
Fetch Channel
Concurrent Fetch Control
Detailed Implementation
State Tracking: Use a concurrent-safe map to track the state of each key.
Channel for Coordination: Implement a channel to coordinate fetch requests and responses.
GO
package request_coalescer

import (
	"context"
	"errors"
	"sync"
)

// RequestKey is the type representing the unique identifier for a request.
type RequestKey string

// FetchResult is the structure containing the result or error of the fetch.
type FetchResult struct {
	Value  interface{} // Value returned by the fetch
	Error  error       // Error encountered during the fetch, if any
}

var (
	errNoValue     = errors.New("no value found")
	errFetching   = errors.New("error fetching data")
)

// RequestCoalescer is responsible for ensuring only one concurrent fetch per key.
type RequestCoalescer struct {
	stateMap  map[RequestKey]FetchResult // Tracks the state of each request
	fetchChan chan FetchResult           // Channel to coordinate fetch results
	mu        sync.Mutex                 // Mutex for thread safety
}

func NewRequestCoalescer() *RequestCoalescer {
	return &RequestCoalescer{
		stateMap:  make(map[RequestKey]FetchResult),
		fetchChan: make(chan FetchResult, 10), // Buffer size can be adjusted as needed
	}
}

// Fetch gets the value for a given key. If a fetch is already in progress,
// it blocks until the result is available.
func (rc *RequestCoalescer) Fetch(ctx context.Context, key RequestKey) (interface{}, error) {
	rc.mu.Lock()
	defer rc.mu.Unlock()

	if res, ok := rc.stateMap[key]; ok {
		return res.Value, nil
	}

	// Start a fetch if not already in progress.
	select {
	case rc.fetchChan <- FetchResult{}:
		go func(key RequestKey) {
			defer rc.fetchChan <- FetchResult{}

			value, err := doFetch(key)
			if err != nil {
				err = errors.Join(err, errFetching)
			}
			rc.stateMap[key] = FetchResult{Value: value, Error: err}
		}(key)
	default:
	}

	select {
	case res := <-rc.fetchChan:
		return res.Value, res.Error
	}
}

func doFetch(key RequestKey) (interface{}, error) {
	// Simulate a fetch operation.
	// In a real implementation, this would interact with an external service or database.
	value, err := simulateFetch(key)
	return value, err
}

func simulateFetch(key RequestKey) (interface{}, error) {
	// Simulate fetching data for the key. Return an error 20% of the time.
	if rand.Intn(5) == 0 {
		return nil, errors.New("simulated fetch failure")
	}
	return key + " value", nil
}

func main() {
	rc := NewRequestCoalescer()
	go func() { rc.Fetch(context.Background(), RequestKey("key1")) }()
	go func() { rc.Fetch(context.Background(), RequestKey("key2")) }()
	go func() { rc.Fetch(context.Background(), RequestKey("key3")) }()
	// Simulate some time passing for the fetch to complete.
	time.Sleep(2 * time.Second)
}

#### Closing Summary

The RequestCoalescer class ensures that only one concurrent request is made per key, leveraging channels and a mutex to coordinate between fetch requests. In case of an error during fetching, it's propagated to all waiting callers without caching the error. This design provides efficient resource management while ensuring consistency in response handling.

#### Common Mistakes
and Follow-Ups
Mistake: Not managing state correctly can lead to race conditions or incorrect results.

#### Follow-Up
Discussing how to handle retries, timeouts, or different fetch strategies based on application requirements could be useful.

**Code:**

```go
package request_coalescer

import (
	"context"
	"errors"
	"sync"
)

// RequestKey is the type representing the unique identifier for a request.
type RequestKey string

// FetchResult is the structure containing the result or error of the fetch.
type FetchResult struct {
	Value  interface{} // Value returned by the fetch
	Error  error       // Error encountered during the fetch, if any
}

var (
	errNoValue     = errors.New("no value found")
	errFetching   = errors.New("error fetching data")
)

// RequestCoalescer is responsible for ensuring only one concurrent fetch per key.
type RequestCoalescer struct {
	stateMap  map[RequestKey]FetchResult // Tracks the state of each request
	fetchChan chan FetchResult           // Channel to coordinate fetch results
	mu        sync.Mutex                 // Mutex for thread safety
}

func NewRequestCoalescer() *RequestCoalescer {
	return &RequestCoalescer{
		stateMap:  make(map[RequestKey]FetchResult),
		fetchChan: make(chan FetchResult, 10), // Buffer size can be adjusted as needed
	}
}

// Fetch gets the value for a given key. If a fetch is already in progress,
// it blocks until the result is available.
func (rc *RequestCoalescer) Fetch(ctx context.Context, key RequestKey) (interface{}, error) {
	rc.mu.Lock()
	defer rc.mu.Unlock()

	if res, ok := rc.stateMap[key]; ok {
		return res.Value, nil
	}

	// Start a fetch if not already in progress.
	select {
	case rc.fetchChan <- FetchResult{}:
		go func(key RequestKey) {
			defer rc.fetchChan <- FetchResult{}

			value, err := doFetch(key)
			if err != nil {
				err = errors.Join(err, errFetching)
			}
			rc.stateMap[key] = FetchResult{Value: value, Error: err}
		}(key)
	default:
	}

	select {
	case res := <-rc.fetchChan:
		return res.Value, res.Error
	}
}

func doFetch(key RequestKey) (interface{}, error) {
	// Simulate a fetch operation.
	// In a real implementation, this would interact with an external service or database.
	value, err := simulateFetch(key)
	return value, err
}

func simulateFetch(key RequestKey) (interface{}, error) {
	// Simulate fetching data for the key. Return an error 20% of the time.
	if rand.Intn(5) == 0 {
		return nil, errors.New("simulated fetch failure")
	}
	return key + " value", nil
}

func main() {
	rc := NewRequestCoalescer()
	go func() { rc.Fetch(context.Background(), RequestKey("key1")) }()
	go func() { rc.Fetch(context.Background(), RequestKey("key2")) }()
	go func() { rc.Fetch(context.Background(), RequestKey("key3")) }()
	// Simulate some time passing for the fetch to complete.
	time.Sleep(2 * time.Second)
}
```

