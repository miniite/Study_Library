---
type: "practice"
difficulty: "Intermediate"
---

# How would you design the schema and API for a blogging platform?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pagination Strategies](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Pagination_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

"Design the schema and API for a blogging platform" is a scoped low-level / API design question (Medium/WordPress-style product). The interviewer wants to see that you can (1) drive out requirements before touching a table, (2) model entities and relationships cleanly with the right keys, indexes, and constraints, and (3) design a RESTful, consistent, paginated, secure API over that model. At intermediate level they expect real trade-offs — slugs vs IDs, tags many-to-many, soft deletes, draft/publish states, pagination strategy, auth — not an exhaustive enterprise schema. I'll state assumptions, define scope, model the data, then the API, then call out scale/edge concerns.

Clarifying questions / assumptions I'd state: Single-tenant blog or multi-author platform (like Medium)? I'll assume multi-author: users write posts, posts have tags, readers comment and like. Reads vastly outnumber writes (typical 100:1+), so I'll optimize read paths. Relational DB (Postgres) as the primary store — the data is highly relational (users↔posts↔comments↔tags) and needs consistency.

1. Requirements (functional + non-functional)

Functional: register/login; author creates/edits/deletes posts (draft → published); posts have title, body, tags, cover image; readers list/read posts, filter by tag/author, comment (nested optional), like; search. Non-functional: read-heavy → cache + indexes + pagination; SEO-friendly URLs (slugs); consistency for writes; soft delete/audit; authz (only author edits own post).

2. Data model

Core entities: User, Post, Tag, PostTag (join), Comment, Like. Relationships:

User 1—N Post (author).
Post N—M Tag via PostTag join table (a post has many tags, a tag has many posts).
Post 1—N Comment; Comment self-references parent for threads.
Like is an association between User and Post; a UNIQUE(user_id, post_id) prevents double-likes.

Design decisions worth defending:

Surrogate PK (BIGINT/UUID) + a separate unique slug. IDs are stable internal keys; slugs (my-first-post) are human/SEO URLs. Store slug unique (often unique per author) and index it because that's the public read path.
Post status enum (draft/published/archived) instead of a boolean — extensible, and lets you index WHERE status='published'.
Soft delete (deleted_at) so removed content is recoverable and referential history survives.
Denormalize counters (like_count, comment_count) on Post to avoid COUNT(*) on every list render; update them transactionally or via triggers/async — a read-heavy optimization.
Timestamps (created_at, updated_at, published_at) everywhere; published_at is null until published and drives "latest posts" ordering.
Indexes: posts(slug) unique, posts(status, published_at DESC) for the feed, posts(author_id), comments(post_id), post_tags(tag_id) and (post_id).

I'd keep the large body in the same table for a moderate blog; at very large scale you might move body to object storage or a separate table to keep the hot row narrow.

3. API design (REST)

Principles: nouns as resources, HTTP verbs for actions, proper status codes, cursor or page-based pagination on all list endpoints, JWT/session auth, and consistent error envelopes. Versioned under /api/v1.

Key endpoints:

Auth: POST /auth/register, POST /auth/login → returns token.
Posts (collection): GET /posts (list published, filter ?tag=, ?author=, ?q= search, paginate ?page=&limit= or ?cursor=), POST /posts (create draft, auth required).
Post (item): GET /posts/{slug} (public read), PUT/PATCH /posts/{id} (author only), DELETE /posts/{id} (soft delete, author/admin).
Publish is a state transition: POST /posts/{id}/publish — an action endpoint is cleaner than overloading PATCH for a workflow step.
Comments (nested resource): GET /posts/{id}/comments, POST /posts/{id}/comments, DELETE /comments/{id}.
Likes: POST /posts/{id}/like, DELETE /posts/{id}/like (idempotent per user).
Tags: GET /tags, GET /tags/{slug}/posts.

Cross-cutting: POST returns 201 + Location; unauthorized = 401, forbidden (not your post) = 403, not found = 404, validation = 400/422; every list response carries pagination metadata. Rate-limit writes; validate/sanitize body (XSS in rendered Markdown/HTML is the classic blog vuln).

4. Read-heavy optimizations & scale
Cache hot reads (GET /posts/{slug} and the front-page feed) in Redis/CDN; invalidate on edit/publish.
Pagination: page-based is simplest; keyset/cursor pagination on (published_at, id) avoids the OFFSET slowdown on deep pages — important for infinite scroll.
Search: LIKE/ILIKE is fine early; move to Postgres full-text or Elasticsearch as content grows.
Media: store images in S3/object storage; DB keeps only the URL.
5. Edge cases & correctness

Slug collisions (append -2 or suffix a short hash); editing a published post (keep updated_at, maybe versions); deleting a post with comments (cascade or soft-delete children); unpublish; author deletion (reassign/soft-delete their posts); N+1 queries when listing posts with author+tags (eager-load / join). Authorization must be checked server-side on every mutation, not just hidden in UI.

Summary — what a strong answer emphasizes

Start from requirements and read/write ratio, then present a normalized relational schema with the right keys (surrogate PK + slug), a tags join table, a status enum, soft deletes, denormalized counters, and targeted indexes. Layer a clean, versioned, paginated REST API with correct verbs/status codes, action endpoints for state transitions (publish/like), nested comment resources, and server-side authz. Close with read-heavy optimizations (caching, keyset pagination, full-text search) and edge cases (slug collisions, cascades, N+1). Common mistakes: modeling tags as a comma string instead of a join table, using OFFSET pagination blindly, forgetting authz on edits, and putting "publish" as a magic boolean with no audit trail.

**Comparison:**

```
Design choice	Option A	Option B	Chosen & why
Post identifier in URL	numeric id	slug	slug for SEO/readability; id kept as internal PK
Tags storage	comma-separated string on post	join table (post_tags)	join table — enables filtering, indexing, tag pages
Publish flag	boolean is_published	status enum + published_at	enum: extensible (draft/archived) + orderable, auditable
Delete	hard delete	soft delete (deleted_at)	recoverable, preserves referential history
Pagination	OFFSET/page	keyset/cursor	O(1) deep pages, stable for infinite scroll
Counts	COUNT(*) each read	denormalized counters	avoids per-request aggregation on read-heavy feed
```

**Code:**

```

SQL schema (PostgreSQL):

SQL
Copy
CREATE TABLE users (
  id          BIGSERIAL PRIMARY KEY,
  username    VARCHAR(50)  UNIQUE NOT NULL,
  email       VARCHAR(255) UNIQUE NOT NULL,
  pass_hash   VARCHAR(255) NOT NULL,
  created_at  TIMESTAMPTZ  NOT NULL DEFAULT now()
);

CREATE TYPE post_status AS ENUM ('draft','published','archived');

CREATE TABLE posts (
  id            BIGSERIAL PRIMARY KEY,
  author_id     BIGINT NOT NULL REFERENCES users(id),
  slug          VARCHAR(200) UNIQUE NOT NULL,
  title         VARCHAR(300) NOT NULL,
  body          TEXT NOT NULL,
  cover_url     VARCHAR(500),
  status        post_status NOT NULL DEFAULT 'draft',
  like_count    INT NOT NULL DEFAULT 0,
  comment_count INT NOT NULL DEFAULT 0,
  published_at  TIMESTAMPTZ,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at    TIMESTAMPTZ
);
CREATE INDEX idx_posts_feed   ON posts (status, published_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_posts_author ON posts (author_id);

CREATE TABLE tags (
  id   BIGSERIAL PRIMARY KEY,
  slug VARCHAR(80) UNIQUE NOT NULL,
  name VARCHAR(80) NOT NULL
);

CREATE TABLE post_tags (               -- many-to-many join
  post_id BIGINT REFERENCES posts(id) ON DELETE CASCADE,
  tag_id  BIGINT REFERENCES tags(id)  ON DELETE CASCADE,
  PRIMARY KEY (post_id, tag_id)
);
CREATE INDEX idx_posttags_tag ON post_tags (tag_id);

CREATE TABLE comments (
  id         BIGSERIAL PRIMARY KEY,
  post_id    BIGINT NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  author_id  BIGINT NOT NULL REFERENCES users(id),
  parent_id  BIGINT REFERENCES comments(id),   -- threaded replies
  body       TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  deleted_at TIMESTAMPTZ
);
CREATE INDEX idx_comments_post ON comments (post_id);

CREATE TABLE likes (
  user_id BIGINT REFERENCES users(id),
  post_id BIGINT REFERENCES posts(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (user_id, post_id)     -- prevents double-like
);

Express route sketch (auth + ownership check):

JS
Copy
// GET /api/v1/posts?tag=&author=&cursor=&limit=20
router.get('/posts', async (req, res) => {
  const limit = Math.min(+req.query.limit || 20, 100);
  const posts = await Post.feed({ tag: req.query.tag, author: req.query.author,
                                  cursor: req.query.cursor, limit });
  res.json({ data: posts, next_cursor: posts.at(-1)?.cursor ?? null });
});

// PATCH /api/v1/posts/:id  (author only)
router.patch('/posts/:id', requireAuth, async (req, res) => {
  const post = await Post.findById(req.params.id);
  if (!post || post.deleted_at) return res.status(404).json({ error: 'not_found' });
  if (post.author_id !== req.user.id) return res.status(403).json({ error: 'forbidden' });
  const updated = await post.update(req.body);   // validated/sanitized
  res.json({ data: updated });
});
```

**Example input / output:**

```

Create a post (draft), then publish, then read:

Request:

CODE
Copy
POST /api/v1/posts        Authorization: Bearer <jwt>
{ "title": "Designing a Blog Schema", "body": "## Intro ...", "tags": ["databases"] }

Response:

CODE
Copy
HTTP/1.1 201 Created
Location: /api/v1/posts/1024
{ "data": { "id": 1024, "slug": "designing-a-blog-schema", "status": "draft" } }

Publish (state transition):

CODE
Copy
POST /api/v1/posts/1024/publish     ->  200 { "data": { "status": "published",
                                                          "published_at": "2026-06-30T09:12:00Z" } }

Public read + paginated feed:

CODE
Copy
GET /api/v1/posts/designing-a-blog-schema   -> 200 { "data": { ...full post... } }
GET /api/v1/posts?tag=databases&limit=20    -> 200 { "data": [...], "next_cursor": "eyJwdWIiOi4uLn0" }

Ownership failure:

CODE
Copy
PATCH /api/v1/posts/1024   (as a different user)  -> 403 { "error": "forbidden" }
```

**Data:**

```

Post JSON representation returned by the API:

JSON
Copy
{
  "id": 1024,
  "slug": "designing-a-blog-schema",
  "title": "Designing a Blog Schema",
  "author": { "id": 7, "username": "asha" },
  "tags": ["databases", "api-design"],
  "cover_url": "https://cdn.example.com/covers/1024.jpg",
  "status": "published",
  "like_count": 42,
  "comment_count": 5,
  "published_at": "2026-06-30T09:12:00Z",
  "excerpt": "How to model users, posts, tags and comments..."
}
```

**Flow:**

1. authors
2. writes
3. has
4. receives
5. gives
6. parent-of (threads)
7. USERS
8. POSTS
9. COMMENTS
10. LIKES
11. POST_TAGS
12. TAGS

