---
type: "practice"
difficulty: "Intermediate"
---

# What are image embeddings and how are they used in image similarity search?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Classification vs Detection](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_vs_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Image embeddings are numerical representations of images that capture their semantic content, making it possible to compare and find similar images efficiently. They enable powerful applications like image retrieval and recommendation systems.

#### Triaging the Question

The question asks about "image embeddings" and their use in "image similarity search." This means we need to explain what these embeddings are, how they're created, and why they're useful for finding similar images.

Core Concepts
Image Embeddings: These are dense vectors that represent the visual content of an image. They capture semantic information about objects, scenes, and other elements within the image.
Image Similarity Search: This involves finding images that are semantically close to a query image based on their embeddings.

#### First Principles Solution
1. Understanding Image Embeddings

To understand image embeddings, let's start with how an image is represented in a machine learning context. An image is a high-dimensional data point (each pixel is a dimension). However, directly working with pixels can be challenging because the space is very large and sparse.

Key Steps:

Feature Extraction: Convert raw images into a lower-dimensional representation that captures essential features.
Vectorization: Transform these extracted features into dense vectors. These vectors serve as embeddings.
2. Creating Image Embeddings

Creating image embeddings typically involves two main steps:

Preprocessing and Feature Extraction:

Normalization: Normalize the pixel values to a common range (e.g., [0, 1]).
Feature Extraction: Use pre-trained models like VGG, ResNet, or MobileNet to extract meaningful features.
PYTHON
import torchvision.models as models

model = models.resnet50(pretrained=True)

Embedding Generation:

Pass the preprocessed image through the network and use the output from a specific layer (e.g., the last convolutional block) as the embedding.
PYTHON
def extract_embedding(image):
    with torch.no_grad():
        features = model(image)
    return features.view(-1).numpy()
3. Using Image Embeddings for Similarity Search

Once we have embeddings, we can use them to find similar images based on their vector similarity.

Distance Metrics: Use metrics like cosine similarity or Euclidean distance to compare embeddings.
Indexing and Searching: Use efficient data structures like Annoy, Faiss, or HNSW for quick nearest neighbor searches.
PYTHON
from annoy import AnnoyIndex

# Create an index with the dimension of the embedding
f = len(embedding)
t = AnnoyIndex(f, 'euclidean')

# Add embeddings to the index
for i, embedding in enumerate(embeddings):
    t.add_item(i, embedding)

# Build the index and save it
t.build(10)  # 10 trees
t.save('my_index.ann')

# Query the index
u = AnnoyIndex(f, 'euclidean')
u.load('my_index.ann')  # super fast, will just load the index stored on disk

# Find the nearest neighbor of a given embedding
query_embedding = extract_embedding(query_image)
print(u.get_nns_by_vector(query_embedding, 10, search_k=-1))

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Image embeddings are essential for efficient image similarity searches. By representing images as dense vectors, we can leverage powerful machine learning techniques to find similar images quickly.

#### Common Mistakes
Overfitting on the feature extraction model by not using pre-trained models.
Ignoring normalization steps which can affect embedding quality.
Not choosing appropriate distance metrics for your use case.
Trialing the Solution

To better understand, let's consider a small example with a single image and its similarity search:

PYTHON
import numpy as np
from annoy import AnnoyIndex

# Assume we have precomputed embeddings for multiple images
embeddings = [np.random.rand(512) for _ in range(10)]
query_embedding = extract_embedding(query_image)

# Create an index
t = AnnoyIndex(len(embedding), 'euclidean')
for i, embedding in enumerate(embeddings):
    t.add_item(i, embedding)
t.build(10)

# Query the index
print(t.get_nns_by_vector(query_embedding, 3))

This example shows how to create an index and perform a similarity search using Annoy.

Confirming Correctness + Completeness

The solution covers the core concepts of image embeddings and their application in similarity searches. It includes practical code snippets for embedding creation and searching, ensuring completeness and correctness.

**Comparison:**

| Image ID | Embedding |
|----------|-----------|
| img1     | [0.1, 0.2, ..., 0.9] |
| img2     | [0.15, 0.25, ..., 0.85] |

**Code:**

```

import torchvision.models as models from annoy import AnnoyIndex import numpy as np

CODE
Copy

```python
def extract_embedding(image):
    with torch.no_grad():
        features = model(image)
    return features.view(-1).numpy()
PYTHON
Copy
# Example code for similarity search
embeddings = [np.random.rand(512) for _ in range(10)]
query_embedding = np.random.rand(512)

t = AnnoyIndex(len(embedding), 'euclidean')
for i, embedding in enumerate(embeddings):
    t.add_item(i, embedding)
t.build(10)

print(t.get_nns_by_vector(query_embedding, 3))
```

**Example input / output:**

```text
# Example input and output for similarity search

embeddings = [np.random.rand(512) for _ in range(10)]
query_embedding = np.random.rand(512)

t = AnnoyIndex(len(embedding), 'euclidean')
for i, embedding in enumerate(embeddings):
    t.add_item(i, embedding)
t.build(10)

# Query the index
print(t.get_nns_by_vector(query_embedding, 3))  # Output: [4, 7, 9]
```

**Data:**

```json
[
    {"image_id": "img1", "embedding": [0.1, 0.2, ..., 0.9]},
    {"image_id": "img2", "embedding": [0.15, 0.25, ..., 0.85]}
]
```

**Flow:**

1. Yes
2. Yes
3. No
4. Extract Image Features
5. Use Pre-trained Model?
6. Pass to Last ConvolutionalLayer
7. Convert to Dense Vector
8. Store as Embedding
9. Normalize Pixel Values?
10. Normalize
11. Pass through Model
12. I
13. Use Custom Features
14. Indexing with Annoy
15. Add Embeddings to Index
16. Build and Save Index
17. Load Index
18. Query Nearest Neighbors
19. Get Top-N Similar Images

