---
type: "practice"
difficulty: "Intermediate"
---

# Design a URL preview generator that fetches title and image for a shared link.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design a URL preview generator, we need to break down the problem into several key steps: fetching metadata from the target website, ensuring reliability through retries and error handling, caching results to reduce load, and securing access with proper authentication mechanisms.

The interviewer is probing our understanding of web scraping techniques, HTTP requests, data storage, and system design principles. We must assume that the URL is publicly accessible, we have control over a backend server, and there are no external APIs available for fetching metadata.

First, let's reason through the problem from first principles.

**Comparison:**

```
Fetch Attempt	Status	Action
1	Success	Store and return
2	Failure	Retry
3+	Failure	Return Error
```

**Code:**

```python
# A basic structure of a Python-based URL preview generator
class URLPreviewGenerator:
    def __init__(self):
        self.cache = {}

    def fetch_preview(self, url):
        if url in self.cache:
            return self.cache[url]

        # Fetching the content from the website
        try:
            response = requests.get(url)
            response.raise_for_status()
        except requests.RequestException as e:
            print(f"Failed to fetch {url}: {e}")
            return None

        # Parsing the HTML and extracting title and image URL
        soup = BeautifulSoup(response.text, 'html.parser')
        title = soup.title.string if soup.title else "No Title"
        images = [img['src'] for img in soup.find_all('img', src=True)]
        preview_data = {'title': title, 'images': images}

        # Caching the result
        self.cache[url] = preview_data

        return preview_data
```

**Example input / output:**

```text
# Example input and output
url = "https://example.com"

preview_generator = URLPreviewGenerator()
result = preview_generator.fetch_preview(url)

print("Title:", result['title'])
for img in result['images']:
    print("Image:", img)
```

**Data:**

```json
{
  "url": "https://example.com",
  "expected_response": {
    "title": "Example Domain",
    "images": [
      "https://example.com/image1.png",
      "https://example.com/image2.png"
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. URL Preview Request
6. Fetch URL
7. Cache Hit?
8. Return Cached Data
9. Publish Fetch Task
10. Fetch Metadata
11. Parse HTML for Title andImages
12. Store in Cache & Return
13. Return Error

