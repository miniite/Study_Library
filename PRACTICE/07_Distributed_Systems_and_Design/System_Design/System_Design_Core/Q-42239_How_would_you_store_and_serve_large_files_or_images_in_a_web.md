---
type: "practice"
difficulty: "Intermediate"
---

# How would you store and serve large files or images in a web app?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ URL Shortener](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/URL_Shortener.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: In a typical web app, where do large files and images physically live, and how do we serve them fast to users worldwide? This is the practical, everyday cousin of "design Dropbox." The interviewer wants to hear the standard, correct pattern: store the bytes in object storage, keep only a URL/metadata in your database, upload via pre-signed URLs, and serve through a CDN — plus image-specific concerns (thumbnails/derivatives, responsive formats, lazy loading). They're checking that you won't store images as BLOBs in the DB or stream them through your API servers.

Clarifying questions I'd ask:

User-uploaded images (avatars, product photos) or static assets shipped with the app? (Both, handled slightly differently — static assets go in the build/CDN; user content goes to object storage.)
Scale and read pattern? Almost always read-heavy — an image is uploaded once and viewed many times → this is what makes a CDN essential.
Public or private/authorized content? (Public → cache freely on CDN; private → signed URLs / signed cookies.)
Do we need multiple sizes/formats (thumbnail, medium, full; WebP/AVIF)? (For images, yes — derivatives + responsive delivery.)

Plan: state the anti-pattern to avoid, give the canonical storage layer, walk the upload path (pre-signed) and the serve path (CDN + caching), then cover image-specific optimizations, then privacy/security and cost.

What NOT to do (and why)
Don't store files in the database as BLOBs. It bloats the DB, kills backup/restore times, wastes expensive DB storage on bytes you never query, and makes caching hard. The DB should hold metadata and a URL, not megabytes of image data.
Don't serve files through your application servers. Proxying bytes through Node/Django/Rails ties up app threads/memory, wastes egress bandwidth, and doesn't scale. App servers should be stateless and handle logic, not act as a file pipe.
The canonical pattern

Bytes → object storage. URL/metadata → your DB. Delivery → CDN.

Object storage (Amazon S3, Google Cloud Storage, Azure Blob, or Cloudinary/imgix for images specifically). It's cheap, effectively infinite, ~11-nines durable, and designed exactly for this. Each file gets a stable key; the DB stores that key/URL plus size, content-type, owner, dimensions, etc.
CDN in front (CloudFront, Cloudflare, Fastly, Akamai). The CDN caches files at edge locations near users, so a user in Mumbai gets the image from a nearby edge instead of a bucket in us-east-1. This cuts latency dramatically and offloads bandwidth from the origin — critical because serving is read-heavy.
DB row ties the file to your domain objects (this avatar belongs to user 42; this photo belongs to product 991).
Upload path — pre-signed URLs

Don't send the upload through your API. Instead:

Client asks your backend for permission to upload; backend authenticates, checks type/size limits, and returns a short-lived pre-signed URL (or signed policy) scoped to a single object + PUT operation.
Client uploads the bytes directly to object storage using that URL. Your servers never touch the bytes.
On success, the client (or an S3 event) notifies the backend, which writes the metadata row (key, size, content-type, dimensions) and marks it usable.
Validate on the server side too: cap size, restrict content types (reject executables masquerading as images), and re-check dimensions — never trust the client. For images, run them through an image pipeline to strip EXIF and re-encode (defends against malicious payloads).

For very large files, use multipart/chunked resumable uploads so a dropped connection resumes instead of restarting.

Serve path — CDN + caching
Return the file's CDN URL to the browser; the browser fetches directly from the edge.
Set long Cache-Control headers (public, max-age=31536000, immutable) and use content-hashed / versioned URLs (avatar.a1b2c3.webp) so you can cache aggressively yet bust the cache instantly by changing the URL when the file changes.
The CDN caches on first request; subsequent requests are served from the edge — fast and cheap.
Image-specific optimizations

Images are where most of the real work is:

Generate derivatives. On upload, create multiple sizes (thumbnail, medium, full) so a 4000×3000 phone photo isn't shipped to a 100 px avatar slot. Do this async (a worker triggered by the upload) or on-the-fly with an image CDN.
Modern formats. Serve WebP/AVIF to browsers that support them (via Accept negotiation or an image CDN) — often 30–60% smaller than JPEG/PNG at equal quality.
Responsive delivery. Use srcset/sizes (or an image CDN with URL params like ?w=400) so each device downloads an appropriately sized image.
Lazy loading (loading="lazy") so off-screen images don't block initial page load; add width/height or aspect-ratio to prevent layout shift.
Compression. Strip metadata and re-encode at sensible quality (e.g. 80). An image CDN (Cloudinary/imgix/Cloudflare Images) can do resize + format + compress on the fly from one master, which simplifies the pipeline a lot.
Public vs. private content
Public (product photos, blog images): cache freely on the CDN, plain URLs.
Private/authorized (a user's private documents, paid content): use signed URLs or signed cookies with short expiries so only authorized users fetch; keep the CDN's cache key aware of authorization, or serve private content through signed-URL-only paths. Never make a "private" file just an unguessable public URL if it's truly sensitive.
Cost & operations
Object storage is cheap per GB; the bigger cost is often egress bandwidth, which the CDN reduces (edge caching + sometimes cheaper egress).
Use lifecycle rules to move cold files to cheaper tiers (S3 IA/Glacier) and to auto-delete orphaned/temp uploads.
Track orphaned blobs (uploaded but never attached to a DB row) with a TTL cleanup job.
Summary — what a strong answer emphasizes

Say it crisply: "Put the bytes in object storage (S3/GCS), store just the URL and metadata in the DB, upload with pre-signed URLs so the API never proxies bytes, and serve everything through a CDN with long-lived cache headers on versioned URLs." For images specifically, add derivatives/thumbnails, WebP/AVIF, responsive srcset, and lazy loading — or lean on an image CDN to do it. Cover private content via signed URLs. Common mistakes: storing images as DB BLOBs; serving files through app servers; no CDN (slow, expensive at scale); shipping full-resolution originals to every device; and trusting client-declared type/size. Follow-up probes: How do you cache-bust? (content-hashed URLs + immutable). How do you secure private images? (signed URLs/cookies, short TTL). How do you make thumbnails without blocking upload? (async worker or on-the-fly image CDN). How do you handle huge uploads? (multipart/resumable).

**Comparison:**

```
Concern	Wrong / naive approach	Recommended approach
Where bytes live	BLOB column in the DB	Object storage (S3/GCS/Azure Blob)
Byte transfer on upload	Through the API server	Direct to object store via pre-signed URL
Global delivery	Serve from origin/app server	CDN edge caching, long-lived cache headers
Cache busting	Query string tweaks, short TTLs	Content-hashed URLs + immutable
Image sizes	Ship full-res original	Derivatives + srcset / image-CDN resize
Formats	JPEG/PNG only	WebP/AVIF with negotiation
Page load	Load all images eagerly	loading="lazy" + reserved dimensions
Private files	Unguessable public URL	Signed URLs / cookies, short expiry
Cold/large storage	One tier forever	Lifecycle rules → IA/Glacier; multipart for big files
```

**Code:**

```

// UPLOAD: backend issues a short-lived pre-signed PUT URL (bytes never touch API) // AWS SDK v3 import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3"; import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const s3 = new S3Client({ region: "ap-south-1" });

app.post("/uploads/presign", auth, async (req, res) => { const { contentType, contentLength } = req.body; if (!/^image/(png|jpe?g|webp)$/.test(contentType)) return res.status(415).end(); if (contentLength > 10 * 1024 * 1024) return res.status(413).end(); // 10 MB cap const key = uploads/${req.user.id}/${crypto.randomUUID()}; const url = await getSignedUrl( s3, new PutObjectCommand({ Bucket: "app-media", Key: key, ContentType: contentType }), { expiresIn: 300 } // 5-minute window ); res.json({ url, key }); // client PUTs bytes directly to S3 });

// Client uploads straight to object storage, then tells the API to save metadata: // await fetch(url, { method: "PUT", body: file, headers: { "Content-Type": file.type }}); // await fetch("/uploads/commit", { method:"POST", body: JSON.stringify({ key }) });

CODE
Copy

```html
<!-- SERVE: CDN URL + responsive, modern-format, lazy-loaded image -->
<img
  src="https://cdn.app.com/uploads/u42/photo.a1b2c3.jpg"   <!-- content-hashed -->
  srcset="https://cdn.app.com/.../photo.a1b2c3.jpg?w=400 400w,
          https://cdn.app.com/.../photo.a1b2c3.jpg?w=800 800w,
          https://cdn.app.com/.../photo.a1b2c3.jpg?w=1600 1600w"
  sizes="(max-width: 600px) 100vw, 800px"
  width="800" height="600"          <!-- reserve space, avoid layout shift -->
  loading="lazy" decoding="async" alt="Product photo" />
```

**Example input / output:**

```text
Upload an avatar:
  POST /uploads/presign {image/jpeg, 480KB}
    -> {url: "https://s3/app-media/uploads/u42/uuid?X-Amz-Signature=...", key: "..."}
  PUT bytes directly to S3 via url               -> 200 (API saw 0 bytes)
  POST /uploads/commit {key}                     -> media row saved, cdn_url returned

Serve to a phone (600px viewport):
  browser picks srcset ?w=400 variant  -> ~40 KB WebP from nearest CDN edge
  first request: CDN pulls from S3 origin (cache MISS)
  later requests: served from edge (cache HIT) -> ~15 ms, no origin egress

Serve a PRIVATE document:
  GET /media/9/url -> signed URL, Expires in 300s -> only that user can fetch it
```

**Data:**

```

-- DB stores metadata + URL, NOT the bytes CREATE TABLE media ( id BIGINT PRIMARY KEY, owner_id BIGINT NOT NULL, storage_key TEXT NOT NULL, -- object-storage key cdn_url TEXT NOT NULL, -- https://cdn.app.com/... content_type TEXT, -- image/webp, ... size_bytes BIGINT, width INT, height INT, -- for responsive/layout visibility TEXT, -- 'public' | 'private' created_at TIMESTAMPTZ DEFAULT now() );

Cache headers for served objects (set on the object / CDN)

Cache-Control: public, max-age=31536000, immutable # versioned URL -> safe forever Content-Type: image/webp

Private objects instead use signed URLs with a short Expires + Signature.
```

**Flow:**

1. Serve
2. Upload
3. 1. ask to upload
4. 2. pre-signed PUT URL
5. 3. PUT bytes directly
6. 4. commit metadata
7. async
8. GET image
9. cache MISS -> origin pull
10. cache HIT (fast, cheap)
11. Browser
12. App Server
13. Object Storage
14. ('DB: url + metadata')
15. Worker: thumbnails / WebP
16. User worldwide
17. CDN edge

