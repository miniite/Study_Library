---
type: "practice"
difficulty: "Deep"
---

# How would you write a Python test harness that replays a recorded incident's telemetry through your detector to verify it would have fired at the right time and not before?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Flapping Detection](<../../../../TOOLS/Python/Python_Language/Flapping_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a robust Python test harness, we need to ensure that our detection logic can accurately identify incidents based on historical telemetry data. The goal is to replay recorded incident telemetry through the detector and verify its timing accuracy. This involves setting up a structured approach where we can feed in recorded data points and observe whether the detector correctly triggers at the expected moments.

**Code:**

```python
import unittest
from datetime import datetime

class TestDetector(unittest.TestCase):
    def setUp(self):
        self.detector = Detector()  # Instantiate your detector here

    def test_replay_incident(self):
        recorded_data = [
            {"timestamp": "2023-10-01T14:30:00Z", "metric_value": 5.0},
            {"timestamp": "2023-10-01T14:31:00Z", "metric_value": 6.0},
            # More recorded data points
        ]

        expected_trigger_times = [datetime(2023, 10, 1, 14, 31), datetime(2023, 10, 1, 14, 35)]

        for index, data_point in enumerate(recorded_data):
            timestamp = datetime.fromisoformat(data_point["timestamp"])
            metric_value = data_point["metric_value"]

            # Simulate feeding the detector with the current telemetry
            self.detector.update(timestamp, metric_value)

            if index + 1 < len(expected_trigger_times):
                next_expected_time = expected_trigger_times[index + 1]
                self.assertFalse(self.detector.has_triggered(), f"Detector triggered at {timestamp}, but should not have.")
            else:
                # Final check to ensure the detector triggers exactly at the last expected time
                current_time = datetime.now()
                if (current_time - next_expected_time).total_seconds() < 10:  # Allow a small buffer for clock skew
                    self.assertTrue(self.detector.has_triggered(), f"Detector did not trigger at {next_expected_time}.")
```

