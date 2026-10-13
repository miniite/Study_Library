---
type: "practice"
difficulty: "Deep"
---

# How would you architect PII handling for a computer-use / browser-agent that reads whatever is on screen, potentially capturing PII from pages it wasn't meant to?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Controls for Vendor Models](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Controls_for_Vendor_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect robust data privacy and PII (Personally Identifiable Information) handling in an AI system that captures screenshots or visual content from a browser agent, we need to ensure compliance with regulations such as GDPR, CCPA, and others, while also addressing technical challenges. The core idea is to design a solution that can detect, mask, or redact PII before it leaves the user's device.

#### Triage

The question is probing into how we can handle PII in a way that respects privacy regulations and maintains functionality. We need to assume:

The AI system operates on data captured from browser activity.
Sensitive information such as credit card numbers, social security numbers, addresses, etc., must be protected.
Compliance with legal requirements and ethical standards is mandatory.

#### First-Principles Solution
Pre-Capture Masking: Implement a pre-capture masking mechanism that automatically hides PII in the visual content before it's even captured by the AI system.
Real-Time Detection: Use machine learning models to detect PII on the fly and mask it dynamically.
User Consent and Notification: Provide clear mechanisms for users to give consent or block data collection, and notify them about what is being collected.
Data Anonymization: Ensure that any data captured is anonymized to prevent re-identification of individuals.
Step-by-Step Solution

Pre-Capture Masking

Use a library like OpenCV in Python to preprocess images before they are captured by the AI system.
Implement algorithms to detect common PII patterns and mask them using techniques such as blurring or blackboxing specific regions.

Real-Time Detection

Train machine learning models with datasets of known PII formats (e.g., credit card numbers, addresses).
Integrate these models into the AI system’s pipeline to perform real-time detection on captured images.
When a potential PII is detected, mask it immediately.

User Consent and Notification

Implement an informed consent mechanism where users can choose which data collection activities they allow.
Display clear notifications before any data capture occurs, explaining what type of data will be collected and how it will be used.

Data Anonymization

Use techniques like differential privacy or k-anonymity to anonymize the captured data.
Ensure that any data transmitted from the user’s device is encrypted and cannot be linked back to individual users without their explicit consent.

#### Closing Summary

In summary, a comprehensive approach involves both technical solutions (pre-capture masking, real-time detection) and ethical considerations (user consent, anonymization). The goal is to ensure that no PII leaves the user's device in an unmasked form while maintaining the functionality of the AI system. Common mistakes include neglecting user privacy protections or failing to implement effective masking techniques.

#### Common Mistakes
Neglecting User Consent: Not providing clear and transparent mechanisms for users to control their data.
Insufficient Masking Techniques: Using inadequate algorithms that fail to effectively mask PII.
Data Anonymization Issues: Failing to properly anonymize data, leading to potential re-identification risks.

**Comparison:**

```
Step	Action
1	Install OpenCV and other necessary libraries.
2	Train machine learning models for PII detection.
3	Integrate real-time masking into the AI pipeline.
4	Implement user consent mechanisms.
5	Anonymize data before transmission.
```

**Code:**

```python
import cv2

def mask_pii(image):
    # Example of masking a credit card number in an image using OpenCV
    card_mask = create_mask(image, (x1, y1), (x2, y2))  # x1, y1, x2, y2 are coordinates of the detected PII area
    masked_image = cv2.bitwise_and(image, image, mask=card_mask)
    return masked_image

def create_mask(image, start_point, end_point):
    card_mask = np.zeros_like(image)
    card_mask[start_point[1]:end_point[1], start_point[0]:end_point[0]] = 255
    return cv2.bitwise_not(card_mask)

# Example usage
image = cv2.imread('screenshot.png')
masked_image = mask_pii(image)
cv2.imwrite('masked_screenshot.png', masked_image)
```

**Example input / output:**

```text
Input:
- Screenshot of a web page containing sensitive information like credit card number and address.

Output:
- Masked screenshot where the credit card number is blurred or blackboxed, and no personal data can be re-identified.
- Notification to the user about the data being captured and its purpose.
```

**Data:**

```json
{
  " pii_detection_rules": {
    "credit_card_number_pattern": "[0-9]{16}",
    "ssn_pattern": "[0-9]{3}-[0-9]{2}-[0-9]{4}"
  },
  "consent_requirements": [
    { "action": "read", "data_types": ["text", "image"] }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Pre-Capture
6. Capture Screenshot
7. Is PII Present?
8. Mask PII
9. Send Masked Data for Analysis
10. User Provides Consent
11. Show Notification to User
12. Data Collected
13. H
14. Anonymize Data
15. Transmit Anonymized Data
16. Use Pre-capture MaskingTechniques
17. Send Masked Data for Analysis

