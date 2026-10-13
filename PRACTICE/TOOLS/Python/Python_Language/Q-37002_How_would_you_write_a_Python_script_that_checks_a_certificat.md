---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a Python script that checks a certificate's expiry date and warns if it expires within 30 days?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Comparing Time Windows](<../../../../TOOLS/Python/Python_Language/Comparing_Time_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a Python script to check a certificate’s expiry date, we need to perform several steps. The script will use Python’s ssl module to fetch the certificate from a URL and then parse its expiry date using the datetime module. If the certificate expires within 30 days, it should raise a warning.

Let's break down the solution step by step:

Fetch the Certificate: Use the ssl.create_default_context method to get the SSL context and socket.getpeername to connect to the target URL.
Parse the Expiry Date: Use the context.wrap_socket method with a socket object to get the certificate, then parse its expiry date from the returned certificate object.
Check the Expiry Date: Calculate the difference between the current date and the expiry date. If it is less than 30 days, raise an alert.

Now let’s implement this in Python:

