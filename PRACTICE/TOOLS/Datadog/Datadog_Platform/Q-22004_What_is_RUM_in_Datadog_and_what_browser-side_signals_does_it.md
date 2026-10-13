---
type: "practice"
difficulty: "Basic"
---

# What is RUM in Datadog and what browser-side signals does it capture?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Database Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Database_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

RUM, or Real User Monitoring, is a key feature of Datadog that captures real-world user interactions with your web applications to help you understand their performance from an end-user perspective. In this context, RUM involves monitoring browser-side signals and events that can give insights into how users interact with the application.

#### Triage

The question asks for an explanation of RUM in Datadog and the specific browser-side signals it captures. This is essentially about understanding the role of RUM within Datadog's broader suite of tools, as well as the data points collected to monitor user experience effectively.

#### First-Principles Solution

First, let’s break down what Datadog is: It's a comprehensive monitoring and analytics platform that helps organizations manage their infrastructure, applications, and services. RUM within this context is specifically designed to focus on the end-user experience of web applications. To achieve this, RUM captures various browser-side signals and events in real-time.

Key Browser-Side Signals Captured by Datadog's RUM
Performance Metrics: These include time-to-first-byte (TTFB), response times for different parts of a web page (like JavaScript execution or network requests), and overall load times.
Error Tracking: This includes capturing uncaught exceptions, crashes, and other runtime errors that can impact user experience.
User Behavior Events: Datadog RUM also tracks specific user actions like clicks, scrolls, and form submissions to understand how users interact with the application.
Geolocation Data: Capturing where users are located geographically helps in understanding regional performance differences or issues.
Session Information: This includes details about the session such as user ID, device type, operating system, browser version, etc.

#### Closing Summary

In summary, Datadog's RUM is a powerful tool for monitoring real-world user interactions with web applications by capturing key performance metrics and events directly from browsers. These signals help in diagnosing issues, optimizing application performance, and enhancing the overall user experience.

#### Common Mistakes/Follow-Ups

A common mistake beginners might make is not understanding that these browser-side signals are critical for identifying where performance bottlenecks lie. RUM can reveal insights that traditional server-side monitoring cannot capture. Additionally, it’s important to note that while RUM is powerful, it should be used in conjunction with other Datadog tools to get a comprehensive view of the system.

#### Follow-Up
Questions
How does RUM differ from other types of monitoring?
Can you explain how Datadog's RUM integrates with its other services for a holistic view?

**Comparison:**

| Signal          | Description                                                                                                                                                                                                                         |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TTFB            | Time to first byte is the time taken for a web page or resource to load from the server after the initial request.                                                                                                                   |
| Response Time   | It measures the latency of network requests, which includes fetching resources like images, CSS, and JavaScript files.                                                                                                               |
| Error Tracking  | Tracks uncaught exceptions, crashes, and other runtime errors that can significantly affect user experience.                                                                                                                       |
| User Behavior   | Captures specific actions taken by users such as clicks, scrolls, and form submissions to understand how they interact with the application.                                                                                     |
| Geolocation Data| Provides information about where users are located geographically, which helps in understanding regional performance differences or issues related to location-based services.             |
| Session Information | Includes details like user ID, device type, operating system, browser version etc., which can be used to segment and analyze data based on specific user attributes.                                                                 |

**Example input / output:**

| Input | Output |
|-------|--------|
| User interaction with a web page | Performance metrics, error logs, user behavior events captured and displayed in Datadog dashboard. |

**Data:**

```json
{
  "RUM_Signals": [
    { "Signal": "TTFB", "Description": "Time to first byte" },
    { "Signal": "Response Time", "Description": "Latency of network requests" },
    { "Signal": "Error Tracking", "Description": "Uncaught exceptions and crashes" },
    { "Signal": "User Behavior Events", "Description": "Clicks, scrolls, form submissions" },
    { "Signal": "Geolocation Data", "Description": "User location information" },
    { "Signal": "Session Information", "Description": "Device type, OS, browser version etc." }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Interaction
4. Is it a Web Page?
5. Capture RUM Signals
6. Display Data in DatadogDashboard
7. Other Type of Monitoring

