---
type: "practice"
difficulty: "Deep"
---

# You want to prove to auditors which exact commit trained the model running across five inference clusters at any moment. Compare baking the SHA into the image, exposing it via a health endpoint, and reading it from the model registry, and which is authoritative.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prove to auditors which exact commit trained the model running across five inference clusters at any moment, we need to compare three approaches: baking the SHA into the image, exposing it via a health endpoint, and reading it from the model registry. Each approach has its trade-offs in terms of security, traceability, and operational complexity.

Baking the SHA into the Image: This method involves embedding the commit SHA directly into the Docker image metadata. The main advantage is simplicity; once the image is baked, the SHA cannot be changed without rebuilding the image from scratch. However, this approach lacks dynamic updates because the SHA must be hardcoded at build time.

Exposing it via a Health Endpoint: Another option is to expose the commit SHA through a health endpoint in the running model. This allows for real-time verification and updating of the information without requiring a rebuild. The downside is that it introduces an additional service layer, which could add complexity and potential points of failure.

Reading from the Model Registry: Using a centralized model registry to track and retrieve the commit SHA associated with each deployment provides robust traceability. This method ensures that all relevant metadata, including the commit SHA, is stored in a consistent, version-controlled manner. However, it requires a separate registry service and additional steps for updating and querying.

Given these considerations, we need to determine which approach is authoritative based on the following criteria:

Security: The authoritative source should be immutable and resistant to tampering.
Traceability: It must provide a clear link between the model in use and its training data.
Operational Simplicity: The chosen method should minimize complexity and potential failure points.

Let's analyze each approach step-by-step:

Baking the SHA into the Image
Immutable Metadata: Embedding the SHA directly into the Docker image ensures that the information is immutable once the image is built.
Traceability: The SHA provides a direct link to the specific commit used for training.
Simplicity: This approach is straightforward and doesn't require additional services or endpoints.

However, it lacks real-time updates because the SHA must be hardcoded at build time. If a new commit is made after the image is built, the deployed model will still report the old SHA.

Exposing via a Health Endpoint
Dynamic Updates: The health endpoint can be updated in real-time without requiring a rebuild of the Docker image.
Traceability: Provides up-to-date information about the model's training commit.
Complexity: Introduces an additional service layer, which could add complexity and potential failure points.
Reading from the Model Registry
Immutable Metadata: The registry ensures that all stored metadata is immutable once committed.
Traceability: Provides a clear link between the model in use and its training data through versioning.
Simplicity: Minimizes additional complexity by leveraging existing infrastructure.

However, it requires setting up and maintaining a separate registry service.

#### Conclusion

Based on the criteria of security, traceability, and operational simplicity, the most authoritative approach would be to read the commit SHA from the model registry. This method ensures that all metadata is stored in a consistent, version-controlled manner, providing robust traceability without introducing additional complexity or potential failure points.

In summary:

Baking into the Image: Simple but lacks real-time updates.
Health Endpoint: Real-time but introduces additional service layer.
Model Registry: Authoritative and minimizes operational overhead.

