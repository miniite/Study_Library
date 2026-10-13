---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a false positive and a false negative in a defect-detection vision system, and which is worse?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Classification Metrics](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In computer vision applications like defect detection systems, terms like "false positive" and "false negative" are crucial to understand. A false positive occurs when a defect is incorrectly identified as present in an image where no actual defect exists. Conversely, a false negative happens when a real defect is missed by the system, meaning it fails to detect a known issue.

To answer which is worse between false positives and false negatives, we need to consider the context of the application and its implications. Generally, a false positive can be more annoying and costly but less harmful compared to a false negative, where safety or quality concerns are not addressed.

#### Triage
What's Being Probed

The question aims at understanding your grasp of basic computer vision terminology, as well as the trade-offs involved in designing such systems. The interviewer will likely probe for an explanation of why each type is undesirable and which one might be more critical based on the specific use case.

#### First-Principles Solution
False Positive: This happens when a non-defective part is incorrectly flagged as defective, leading to unnecessary actions like rework or scrapping.
False Negative: This occurs when a defective part is not identified, potentially leading to faulty products being released into the market.
Why Are They Important?

False Positives:

Can lead to increased production costs due to unnecessary rework or inspections.
May cause delays in the production line as unaffected parts need to be handled separately.

False Negatives:

Potentially release defective products, which could harm customers and damage brand reputation.
Lead to recalls and further financial losses.
Which Is Worse?

The severity depends on the context. In a medical imaging system where false negatives can lead to misdiagnosing a serious condition like cancer, a false negative is clearly worse than a false positive. On the other hand, in an industrial setting, a false positive might just be inconvenient but not catastrophic.

#### Closing Summary

In summary, while both are undesirable, the choice between which is "worse" depends on the specific application and its criticality. In safety-critical scenarios like medical diagnostics or automotive components, avoiding false negatives (to prevent faulty products) can be prioritized over minimizing false positives (which might lead to unnecessary rework).

#### Common Mistakes/Follow-Ups
Mistake: Assuming one is always worse without considering the context.

#### Follow-Up
The interviewer may ask you about how to balance these trade-offs in a real-world scenario, such as adjusting model thresholds or incorporating additional sensors.

**Comparison:**

```
	Image1	Image2	Image3	Image4
Defect Present	True	False	False	True
Prediction	True	False	True	False
Status	Correct	Correct	Incorrect	Incorrect
```

**Code:**

```python
# Example of handling false positives and negatives in a simple defect detection system

def detect_defect(image):
    # Placeholder function that returns true for defects (negative) and non-defects (positive)
    result = False  # Simulated defect detection logic

    if not result:  # False Negative case
        print("False Negative: Defect missed")
    else:
        print("Defect detected or no defect")

detect_defect(True)  # Example of a true positive
detect_defect(False) # Example of a false negative
```

**Example input / output:**

```text
Input:
- Image1 has a defect; prediction is correct (True Positive)
- Image2 has no defect; prediction is correct (True Negative)
- Image3 has no defect but predicted as defective (False Positive)
- Image4 has a defect but not detected by the system (False Negative)

Output:
- Defect detected correctly in Image1
- No defect identified correctly in Image2
- False Positive: Defect incorrectly flagged in Image3
- False Negative: Defect missed in Image4
```

**Data:**

```json
{
  "image1": {
    "defectPresent": true,
    "prediction": true,  // Correct detection (True Positive)
    "status": "Correct"
  },
  "image2": {
    "defectPresent": false,
    "prediction": false,  // Correct rejection (True Negative)
    "status": "Correct"
  },
  "image3": {
    "defectPresent": false,
    "prediction": true,   // False Positive
    "status": "Incorrect"
  },
  "image4": {
    "defectPresent": true,
    "prediction": false,  // False Negative
    "status": "Incorrect"
  }
}
```

**Flow:**

1. Input Image
2. Defect Present?
3. True Positive
4. False Negative
5. No Defect?
6. False Positive
7. True Negative

