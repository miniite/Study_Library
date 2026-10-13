---
type: "practice"
difficulty: "Deep"
---

# Your detector produces flickering boxes across video frames. How would you add temporal smoothing or tracking to stabilize outputs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address flickering boxes in your computer vision system, adding temporal smoothing or tracking is essential. This involves leveraging information from multiple frames over time to ensure that detections are more consistent and reliable. The core idea is to establish a robust way of associating objects across frames, rather than treating each frame independently.

First, we need to clarify what the flickering problem entails. Flickering boxes often occur when detections in consecutive frames are highly inconsistent or change rapidly without significant changes in the scene. This can be due to noise, object occlusions, or limitations in the detection algorithm itself. To tackle this issue, we will explore a few techniques: optical flow-based tracking, Kalman filters, and particle filtering.

Let's break down each step from first principles:

Understanding Optical Flow: Optical flow is a technique that estimates the motion of objects between two consecutive frames by analyzing pixel intensity changes. By calculating the optical flow vector at each point in the image, we can predict where an object might be in the next frame and use this information to smooth out detections.

Kalman Filters for Smoothing: Kalman filters are a powerful tool for estimating the state of a system over time given noisy measurements. They can help reduce noise by predicting future states based on past predictions and current observations. By integrating optical flow data with detection results, we can refine our object positions.

Particle Filtering for Robustness: Particle filtering is an alternative to Kalman filters that uses a set of random samples (particles) to represent the state distribution. This approach is particularly useful when dealing with non-linear and non-Gaussian systems. By maintaining multiple hypotheses about object states, we can better handle uncertainties in detections.

Step-by-Step Solution

Initialize Tracking System:

For each frame, initialize a set of trackers (e.g., one per detected box).
Each tracker maintains its own state estimate and uncertainty.

Detect Objects:

Run your current detector on the input frames to get initial bounding boxes.

Optical Flow Computation:

Compute optical flow vectors between consecutive frames using a method like Lucas-Kanade or Farneback’s algorithm.
Use these vectors to predict where each object is likely to be in the next frame.

Kalman Filter Update:

For each tracker, use its previous state estimate and the predicted position from optical flow to update its internal state.
Incorporate the detection results as new measurements to further refine the state estimate.

Particle Filtering (Optional):

Maintain a set of particles representing possible states for each object.
Resample particles based on their likelihood given the current frame's detections and predictions from optical flow.

Assign Detections:

Use an assignment algorithm like Hungarian or Centroid Matching to assign detected boxes to trackers based on proximity and confidence scores.

Output Smoothened Boxes:

For each tracker, output its most likely state as the final bounding box for that object in the current frame.

#### Common Mistakes
and Follow-Ups
Over-reliance on Detection Alone: Relying solely on detection results without considering motion can lead to poor performance in dynamic scenes.
Ignoring Outliers: Failing to properly handle outliers or noisy measurements can degrade tracking accuracy.
Choosing the Right Algorithm: Not selecting the appropriate algorithm for your specific use case (Kalman vs. Particle Filter) can result in suboptimal performance.

By combining optical flow, Kalman filters, and particle filtering, you can achieve more stable object detection over time, reducing flickering boxes and improving overall system robustness.

**Comparison:**

```
Frame	Object ID	Initial Position (x,y)	Final Smoothed Position (x,y)
2	1	[98, 199]	[98.5, 199.3]
2	2	[501, 601]	[500.7, 600.4]
```

**Code:**

```python
import cv2
from kalman_filter import KalmanFilter

# Example of initializing a Kalman filter for tracking
kf = KalmanFilter(dim_x=4, dim_z=2)

def track_objects(frame, detections):
    # Predict and update tracker states using optical flow and detection results
    predicted_poses = optical_flow_predictor.predict_next_positions()

    kf.update(detections)
    updated_states = kf.get_current_state()

    return updated_states

# Example of computing optical flow between frames
prev_frame = cv2.imread('frame1.png')
next_frame = cv2.imread('frame2.png')
flow = cv2.calcOpticalFlowFarneback(prev_frame, next_frame, None, 0.5, 3, 15, 3, 5, 1.2, 0)
```

**Example input / output:**

```text
Input:
- Frame 1 with detected boxes: [[100, 200, 300, 400], [500, 600, 700, 800]]
- Optical flow vectors between frame 1 and frame 2

Output:
- Smoothed box positions for each object in Frame 2: [[98, 199, 302, 401], [501, 601, 701, 800]]
```

**Data:**

```json
{
  "detections": [
    {"id": 1, "box": [100, 200, 300, 400], "confidence": 0.9},
    {"id": 2, "box": [500, 600, 700, 800], "confidence": 0.8}
  ]
}
```

**Flow:**

1. Yes
2. Initialize Kalman Filter for EachObject
3. Is Next Frame Available?
4. Compute Optical FlowBetween Frames
5. Predict Positions Using OpticalFlow
6. Update Kalman Filter with NewDetection Results
7. Get Current State of Tracker
8. Output Smoothed BoxPositions

