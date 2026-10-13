---
type: "practice"
difficulty: "Deep"
---

# How would you implement a custom Trigger to emit a window result both on completion and early once a count threshold is crossed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Multi-granularity Windows](<../../../../TOOLS/Apache_Flink/Flink_Core/Multi-granularity_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing a custom trigger in Apache Flink, especially one that emits results based on both a time-based window completion or an event-count threshold, you need to define a stateful mechanism to track the number of elements and potentially the duration they are in the window. This is crucial for making decisions about when to emit the result. The key idea here is leveraging Flink's Trigger API to implement logic that checks both conditions.

The question is probing your understanding of how to combine time-based and count-based triggers within Flink’s streaming architecture, as well as your ability to manage state in a distributed environment efficiently. Let's break this down step by step from first principles.

#### Triage

We need to create a custom Trigger that emits the window result both when the window completes (by time) and whenever the number of elements in the window reaches a specified threshold. This involves implementing a combination of TimeTrigger and CountTrigger, but since Flink doesn’t provide a built-in composite trigger, we need to implement this functionality manually using stateful operations.

Step-by-Step Solution
Define State for Count: We will maintain the count of elements in the window as part of our state.
Implement Custom Trigger Logic:
Check if the current element brings us over the count threshold.
Check if the time-based condition (e.g., 5 minutes have passed) is met.
Emit Results: If either condition is true, emit the result and reset or update state accordingly.

#### First-Principles Solution
JAVA
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.triggers.Trigger;
import org.apache.flink.streaming.api.windowing.windows.Window;
import org.apache.flink.util.Collector;

public class CustomCountAndTimeTrigger extends Trigger<YourType, YourWindowType> {
    private int countThreshold;
    private long timeThreshold; // e.g., 5 minutes
    private boolean resultEmitted = false;

    public CustomCountAndTimeTrigger(int countThreshold, long timeThreshold) {
        this.countThreshold = countThreshold;
        this.timeThreshold = timeThreshold * 1000L; // convert to milliseconds if necessary
    }

    @Override
    public TriggerResult onElement(YourType element, long timestamp, Window window, Collector<YourType> out) {
        // Increment the counter
        int currentCount = (int) getInternalState().getCounter();
        currentCount++;
        setInternalState(currentCount);

        if (currentCount >= countThreshold || (System.currentTimeMillis() - (long) window.getEnd()) > timeThreshold) {
            resultEmitted = true;
            return TriggerResult.FIRE_AND_PURGE; // Fire the event and purge state
        } else {
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public TriggerResult onProcessingTimeTimeout(Window window, Collector<YourType> out) {
        if (resultEmitted || getInternalState().getCounter() >= countThreshold) { // Check both conditions here
            resultEmitted = true;
            return TriggerResult.FIRE_AND_PURGE; // Fire the event and purge state
        } else {
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public void clear(Window window, Collector<YourType> out) {
        if (!resultEmitted && getInternalState().getCounter() >= countThreshold) { // Final check before clearing
            resultEmitted = true; // Ensure we fire and purge only once even in case of multiple timeouts
            out.collect(window.all());
        }
    }

    private class InternalState {
        private int counter;

        public void setCounter(int counter) {
            this.counter = counter;
        }

        public int getCounter() {
            return counter;
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we've created a custom Trigger that combines both count-based and time-based conditions to emit results. The key steps involve maintaining state for the element count and periodically checking against our thresholds.

#### Common Mistakes

Overlooking State Management: Ensuring the correct state is managed across multiple elements and timeouts.
Incorrect Trigger Logic: Failing to properly handle the logic in onElement and onProcessingTimeTimeout.
Missing Final Check in Clear Method: Not ensuring that results are emitted correctly when necessary.

For further discussion, we could explore trade-offs between different implementation strategies (e.g., using a global state vs. local window state) or consider how to optimize the trigger for large data volumes.

**Code:**

```java
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.triggers.Trigger;
import org.apache.flink.streaming.api.windowing.windows.Window;
import org.apache.flink.util.Collector;

public class CustomCountAndTimeTrigger extends Trigger<YourType, YourWindowType> {
    private int countThreshold;
    private long timeThreshold; // e.g., 5 minutes
    private boolean resultEmitted = false;

    public CustomCountAndTimeTrigger(int countThreshold, long timeThreshold) {
        this.countThreshold = countThreshold;
        this.timeThreshold = timeThreshold * 1000L; // convert to milliseconds if necessary
    }

    @Override
    public TriggerResult onElement(YourType element, long timestamp, Window window, Collector<YourType> out) {
        int currentCount = (int) getInternalState().getCounter();
        currentCount++;
        setInternalState(currentCount);

        if (currentCount >= countThreshold || (System.currentTimeMillis() - (long) window.getEnd()) > timeThreshold) {
            resultEmitted = true;
            return TriggerResult.FIRE_AND_PURGE; // Fire the event and purge state
        } else {
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public TriggerResult onProcessingTimeTimeout(Window window, Collector<YourType> out) {
        if (resultEmitted || getInternalState().getCounter() >= countThreshold) {
            resultEmitted = true;
            return TriggerResult.FIRE_AND_PURGE; // Fire the event and purge state
        } else {
            return TriggerResult.CONTINUE;
        }
    }

    @Override
    public void clear(Window window, Collector<YourType> out) {
        if (!resultEmitted && getInternalState().getCounter() >= countThreshold) {
            resultEmitted = true; // Ensure we fire and purge only once even in case of multiple timeouts
            out.collect(window.all());
        }
    }

    private class InternalState {
        private int counter;

        public void setCounter(int counter) {
            this.counter = counter;
        }

        public int getCounter() {
            return counter;
        }
    }
}
```

