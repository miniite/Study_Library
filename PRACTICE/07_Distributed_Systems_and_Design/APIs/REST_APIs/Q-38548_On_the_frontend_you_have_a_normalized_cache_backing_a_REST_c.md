---
type: "practice"
difficulty: "Deep"
---

# On the frontend you have a normalized cache backing a REST client. When a PATCH returns the updated resource, how do you reconcile it with cached list views and derived selectors without a full refetch?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with PATCH operations in a frontend application, the challenge is to efficiently update the cached state and ensure that derived selectors and list views reflect these changes accurately. The goal here is to reconcile the updated resource with the cached list without causing unnecessary full refetches or unnecessary re-computations of derived data.

#### Triage

The question is asking for a strategy to handle partial updates (PATCH requests) on cached resources in such a way that it minimizes the number of operations and ensures consistency across multiple views and selectors. The key points being probed are:

How to efficiently update the cache when only specific parts of the resource have been updated.
Ensuring derived data and list views remain consistent with the updated resource.

#### Assumptions
The application uses a REST client for making API calls.
There is a normalized cache that stores resources in a structured format (e.g., JSON).
Derived selectors are functions that compute additional data based on cached resources (e.g., filtering, aggregating).
List views display subsets of the cached data.

#### First Principles Solution

The solution involves two main steps: updating the cache and re-computing derived selectors. We need to ensure these steps are as efficient as possible:

Patch Cache: Apply the patch to the relevant parts of the resource in the cache.
Recompute Selectors: Update any affected derived data.

Here’s a detailed breakdown:

Step 1: Patch Cache

When a PATCH request returns an updated resource, we need to apply this update directly to our cached copy. This involves identifying which fields have changed and updating those fields accordingly in the cache.

JAVASCRIPT
// Pseudo-code for patching the cache

#### cons
t originalResource = cache.get(resourceId);

#### cons
t patchData = response.data; // Assume the API returns only the updated fields

for (const key of Object.keys(patchData)) {
  if (patchData[key] !== undefined) {
    originalResource[key] = patchData[key];
  }
}

cache.set(resourceId, originalResource);
Step 2: Recompute Selectors

After updating the cache, we need to recompute any derived data that may have been affected by the changes. For example, if a list view filters resources based on certain fields, those filters should be updated.

JAVASCRIPT
// Pseudo-code for recomputing selectors

#### cons
t filteredResources = cachedList.filter(resource => resource[field] === value);

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key is to handle patches efficiently by only updating what needs to be changed in the cache. After updating the cache, recompute any derived data that depends on those changes.

#### Common Mistakes

Not properly identifying which parts of the resource have been updated.
Failing to recompute selectors and list views that depend on the updated fields.
Full refetching the entire resource instead of just applying the patch.

#### Follow-Ups
How would you handle conflicts or inconsistencies between cached data and server responses?
What if the PATCH updates a nested object within a larger resource? Would your solution change?

**Comparison:**

```
Field	Original Value	Updated Value
id	123	123
name	Original Name	Updated Name
status	active	active
```

**Code:**

```javascript
// Example code for updating cache with a patch response
const originalResource = cache.get(resourceId);
const patchData = response.data;

for (const key of Object.keys(patchData)) {
  if (patchData[key] !== undefined) {
    originalResource[key] = patchData[key];
  }
}

cache.set(resourceId, originalResource);

// Example code for recomputing selectors
const cachedList = cache.getAll();
const filteredResources = cachedList.filter(resource => resource[field] === value);
```

**Example input / output:**

```text
Input:
- Cache state before patch
  - Resource with id '123' has name 'Original Name'
- Server response with PATCH data
  - Updated name to 'Updated Name'

Output:
- Updated cache state after patch
  - Resource with id '123' now has name 'Updated Name'
- Recomputed derived selector based on the updated field
  - Filtered list shows only resources where name is 'Updated Name'
```

**Data:**

```json
{
  "resourceId": "123",
  "originalResource": {
    "id": "123",
    "name": "Original Name",
    "status": "active"
  },
  "patchData": {
    "name": "Updated Name"
  },
  "response": {
    "data": {
      "name": "Updated Name"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive PATCH response
4. Is the field in patch data?
5. Update cache with patch
6. Recompute selectors andviews based on updated fields

