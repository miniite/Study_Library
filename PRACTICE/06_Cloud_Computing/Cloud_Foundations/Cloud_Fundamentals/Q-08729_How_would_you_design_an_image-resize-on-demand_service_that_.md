---
type: "practice"
difficulty: "Intermediate"
---

# How would you design an image-resize-on-demand service that caches derivatives at the CDN so the origin only computes each size once?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CDN Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/CDN_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust and efficient image-resize-on-demand service with caching, we need to ensure that resized images are cached at the Content Delivery Network (CDN) level to minimize requests to the origin server. This will reduce load on our backend while maintaining high performance for users. The key components of this solution include:

Service Architecture: We'll design a service that handles image upload, resizing, and caching.
Caching Strategy: Use a CDN to cache resized images so they can be served directly from the edge.
Backend Handling: Ensure our backend efficiently resizes images only when necessary.

The core idea is to leverage modern cloud infrastructure to minimize compute resources at the origin while maximizing performance through caching.

**Comparison:**

```
Key	Value
CloudFront Distribution ID	E1XYZ2ABCDEF
S3 Bucket Name	images-bucket
Original Image URL	https://example.com/original-image.jpg
Resized Image URL	https://cdn.example.com/300x200/example-image.jpg
```

**Code:**

```python
# Example of a simplified function to handle image resizing and caching
import boto3

def resize_image(image_url, width, height):
    # Assume S3 bucket and CloudFront CDN are already set up
    s3 = boto3.client('s3')
    cloudfront = boto3.client('cloudfront')

    resized_key = f"{width}x{height}/{image_url.split('/')[-1]}"

    if not is_image_in_cache(resized_key):
        # Download image from S3, resize it using a library like Pillow
        img_data = s3.get_object(Bucket='images-bucket', Key=image_url)['Body'].read()
        resized_img = resize_with_pillow(img_data, width, height)

        # Upload resized image to S3
        s3.upload_fileobj(resized_img, 'images-bucket', resized_key)

        # Invalidate CloudFront distribution for the new key
        invalidate_cloudfront_distribution(cloudfront, 'images-bucket', resized_key)

    return f"https://cdn.example.com/{resized_key}"

def is_image_in_cache(key):
    # Check if the image is in cache using CloudFront's API or S3's access logs
    pass

def resize_with_pillow(image_data, width, height):
    from PIL import Image
    img = Image.open(io.BytesIO(image_data))
    resized_img = img.resize((width, height), Image.ANTIALIAS)
    return io.BytesIO(resized_img.tobytes())
```

**Example input / output:**

```text
Input:
- `resize_image("https://example.com/original-image.jpg", 300, 200)`

Output:
- URL for resized image: `https://cdn.example.com/300x200/example-image.jpg`
```

**Data:**

```json
{
  "cloudfront_distribution_id": "E1XYZ2ABCDEF",
  "s3_bucket_name": "images-bucket",
  "image_url": "https://example.com/original-image.jpg"
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Image in Cache?
5. Cdn Serve Image
6. Fetch Image from S3
7. Resize Image with Pillow
8. Upload Resized Image to S3
9. Invalidate CloudFrontDistribution
10. CDN Serves New URL

