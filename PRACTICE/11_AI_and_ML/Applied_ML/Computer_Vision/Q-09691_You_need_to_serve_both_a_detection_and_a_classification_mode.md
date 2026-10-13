---
type: "practice"
difficulty: "Deep"
---

# You need to serve both a detection and a classification model in one request under a tight SLA. How would you architect the pipeline to minimize latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of serving both a detection and a classification model in a single request while maintaining tight Service Level Agreement (SLA) on latency, we need to carefully design an efficient pipeline. The key is to ensure that each component minimizes its processing time and that there are no unnecessary bottlenecks.

Firstly, let's break down the problem:

We have two models: a detection model for locating objects in images, and a classification model to identify what those objects are.
Both models need to process the input image(s) quickly enough to meet the SLA.
The goal is to serve both models efficiently in one request.

#### Triaging the Question

The interviewer is probing your understanding of:

Parallel Processing: Can you design a system that can run multiple models concurrently?
Efficient Model Serving: How do you optimize model inference times?
Resource Management: Are you aware of how to manage resources across different stages in the pipeline?
Step-by-Step Solution
1. Preprocessing

First, we preprocess the input image(s) if necessary. This step is generally lightweight and can be done quickly.

PYTHON
def preprocess_image(image):
    # Convert to RGB, resize, normalize, etc.
    return preprocessed_image
2. Detection Model Inference

Next, we run the detection model on the preprocessed image(s). The output will provide bounding boxes for each detected object.

PYTHON
detection_model = load_detection_model()
detected_objects = detection_model.predict(preprocessed_images)
3. Classification Model Inference

For each object detected in step 2, we then run the classification model to identify what those objects are.

PYTHON
classification_model = load_classification_model()
classifications = [classification_model.predict(detected_object) for detected_object in detected_objects]
4. Postprocessing

Finally, postprocess the results by combining the bounding box information and classifications into a single output format that can be easily consumed by the application.

PYTHON
def postprocess_results(detections, classifications):
    # Combine detections and classifications
    return processed_results

output = postprocess_results(detected_objects, classifications)
Full Solution Summary

To achieve our goal, we need to:

Parallelize Models: Run both models concurrently.
Optimize Inference Time: Use efficient inference techniques for each model.
Resource Management: Ensure that resources are allocated effectively across the pipeline.
Parallelization Strategy
Run the detection and classification models on separate threads or processes to avoid sequential execution bottlenecks.
Optimization Techniques
Batch Processing: If multiple images are input, consider batching them together to reduce overhead.
Model Optimization: Use techniques like quantization, pruning, and model compression to reduce inference time without significantly compromising accuracy.
Caching: Cache frequently used models or precomputed results if applicable.

#### Closing Summary

By leveraging parallel processing, optimizing the models, and managing resources effectively, we can ensure that both detection and classification tasks are performed efficiently within the required SLA. Common pitfalls include underestimating the overhead of model serving or failing to consider resource constraints during high-traffic periods.

**Code:**

```python
def preprocess_image(image):
    # Convert to RGB, resize, normalize, etc.
    return preprocessed_image

detection_model = load_detection_model()
detected_objects = detection_model.predict(preprocessed_images)

classification_model = load_classification_model()
classifications = [classification_model.predict(detected_object) for detected_object in detected_objects]

def postprocess_results(detections, classifications):
    # Combine detections and classifications
    return processed_results

output = postprocess_results(detected_objects, classifications)
```

**Example input / output:**

```text
# Example input
preprocessed_images = [load_image("image1.jpg"), load_image("image2.jpg")]

# Example output
detected_objects = [{"box": (10, 10, 100, 100), "class_prob": 0.95}, {"box": (20, 30, 80, 70), "class_prob": 0.8}]
classifications = ["cat", "dog"]

output = postprocess_results(detected_objects, classifications)
```

**Flow:**

1. Preprocess Image
2. Detection Model
3. Classify Detected Objects
4. Postprocess Results

