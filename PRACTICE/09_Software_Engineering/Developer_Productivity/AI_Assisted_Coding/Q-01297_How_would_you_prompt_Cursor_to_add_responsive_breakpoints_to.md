---
type: "practice"
difficulty: "Basic"
---

# How would you prompt Cursor to add responsive breakpoints to a layout and verify them at mobile and desktop widths?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Agentic Coding with Cursor](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Agentic_Coding_with_Cursor.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To prompt Cursor (or any AI-assistant coding tool like Copilot) to add responsive breakpoints to a layout, we need to provide clear instructions on where to place these breakpoints and how to verify their effectiveness at different screen sizes. The core idea is to ensure that the UI looks good and functions correctly across both mobile and desktop views.

First, let’s break down what the question is asking:

We need to instruct Cursor to insert responsive CSS media queries.
These media queries will define different breakpoints where the layout should adapt its behavior (e.g., changing column count, adding or removing margins).
Additionally, we must validate these breakpoints by checking how the layout renders at both mobile and desktop widths.

Here are some key assumptions:

The codebase already has basic HTML structure with a grid layout.
We have access to a development environment where we can run and preview the changes.

Now, let's walk through this step-by-step:

Identify Breakpoints: Typically, breakpoints for responsive layouts are at common screen sizes such as 375px (iPhone SE), 425px (iPhone X/XR), 768px (tablet), and 1024px (large desktop). We will specify these in our prompt.

#### Cons
truct Media Queries: We need to write CSS media queries that target these breakpoints. For example:

CSS
/* Tablet portrait and up */
@media only screen and (min-width: 768px) {
    .container {
        grid-template-columns: repeat(3, 1fr);
    }
}

/* Desktop */
@media only screen and (min-width: 1024px) {
    .container {
        grid-template-columns: repeat(4, 1fr);
    }
}

Prompt Cursor: Provide a clear instruction to Cursor:

CODE
Add responsive breakpoints for the layout at 768px (tablet portrait and up) and 1024px (desktop). Update the CSS to reflect these breakpoints.

Verification Steps: After inserting the media queries, we need to verify their correctness by checking how the layout looks on different devices or emulators:

BASH
# On a desktop environment
# Open Chrome DevTools and resize the browser window to 768px and 1024px to check if the layout adjusts as expected.

# On a mobile emulator
# Use Android Studio's built-in emulator or another tool like BrowserStack to simulate various device sizes.

Detailed Instructions: Provide more detailed instructions to ensure clarity:

CODE
Add responsive media queries for the layout at 768px (tablet portrait and up) and 1024px (desktop). The container element should change its grid-template-columns property as follows:
- At 768px, use repeat(3, 1fr).
- At 1024px, use repeat(4, 1fr).

After adding the media queries, verify that:
- On a desktop browser (resize to 1024px), the layout should have four columns.
- In a mobile emulator (set to 768px width), the layout should have three columns.

Ensure the layout looks good and functions correctly in both environments.

#### Closing Summary
By providing clear, specific instructions, we can guide Cursor to effectively add responsive breakpoints. Proper verification steps ensure that the changes are applied correctly and work as intended across different screen sizes.

#### Common Mistakes/Follow-Ups

Not specifying exact breakpoint values.
Failing to provide detailed verification steps, leading to potential implementation issues.
Overlooking edge cases like small screens (e.g., tablets in landscape mode).

