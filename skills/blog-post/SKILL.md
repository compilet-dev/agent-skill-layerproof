---
name: blog-post
description: Public API blog posts (X-API-KEY). CRUD blog posts, AI generation (outline/full/section), AI operations (expand/shorten/rewrite), images, exports, and social-campaign-from-blog. PublicApiBlogPostController (/api/v2/blog-posts).
---

# Skill: Blog Posts

## Description

Manage blog posts and AI-powered writing workflows. This skill documents **PublicApiBlogPostController** at `/api/v2/blog-posts`. Authenticate with `X-API-KEY`.

Key async behaviors:

- **Outline/full/section generation** returns `activityId` → poll **`GET /api/v2/jobs/{activityId}`**.
- **Blog post “generate”** returns `aiEditLiveObjectId` → poll **`GET /api/v2/blog-posts/{blog_post_id}/generation/{ai_edit_id}`**, then **POST accept** to commit.
- **AI operations** return `aiOperationId` → poll **`GET /api/v2/blog-posts/{blog_post_id}/ai-operations/{ai_operation_id}`**, then **POST accept** to apply.
- **Exports** return `exportId` → poll **`GET /api/v2/blog-posts/{blog_post_id}/exports/{export_id}`** (not the jobs endpoint).

---

## TypeScript types (selected)

Mirrors the request/response DTOs in `service-poc/service/app/module-client/.../publicapi/blogpost/*`.

```typescript
// --- CRUD ---
type PublicApiBlogPostSummary = {
  id: string;         // UUID
  projectId: string;  // UUID
  title: string;
  description?: string | null;
  status: string;
  createdAt: string; // ISO 8601
  updatedAt: string; // ISO 8601
};
type PublicApiBlogPostListResponse = {
  data: PublicApiBlogPostSummary[];
  nextCursor?: string | null;
  hasMore: boolean;
};
type PublicApiBlogPostResponse = {
  id: string;         // UUID
  projectId: string;  // UUID
  title: string;
  description?: string | null;
  content: string;
  status: string;
  createdAt: string;
  updatedAt: string;
};
type PublicApiUpdateBlogPostContentRequest = { content: string };
type PublicApiUpdateBlogPostMetaRequest = { title?: string | null; description?: string | null };

// --- “Generate” (creates + generates via AI edit) ---
type PublicApiGenerateBlogPostRequest = {
  prompt: string;                 // 1..10000
  title?: string | null;          // 1..255
  description?: string | null;    // max 10000
  outputLanguage?: string | null;
  textDetailLevel?: string | null;
  tone?: string | null;
};
type PublicApiBlogPostGenerateResponse = {
  blogPostId: string;         // UUID
  projectId: string;          // UUID
  aiEditLiveObjectId: string; // UUID (use as ai_edit_id)
  activityId: string;         // UUID (job id)
};
type PublicApiBlogPostGenerationStatusResponse = {
  aiEditLiveObjectId: string; // UUID
  status: string;             // PENDING | STREAMING | ACCEPTED | FAILED (etc)
  proposedContent?: string | null;
};

// --- Outline + section generation (jobs-polling) ---
type PublicApiBlogPostGenerationRequest = { prompt?: string | null }; // max 2000
type PublicApiBlogPostGenerationResponse = {
  activityId: string;
  liveObjectId: string;
  workflowType: string;
  message?: string | null;
};
type PublicApiBlogPostOutlineSection = { id: string; title: string; content?: string | null };
type PublicApiBlogPostOutlineResponse = {
  liveObjectId: string;
  status: string;
  title?: string | null;
  sections: PublicApiBlogPostOutlineSection[];
};

// --- AI operations (selection-based) ---
type PublicApiAiOperationRequest = {
  command: string;                 // expand | shorten | rewrite
  selection: string;               // 1..50000
  prompt?: string | null;          // max 2000
  idempotencyKey?: string | null;  // max 256 (aka idempotency_key)
};
type PublicApiAiOperationResponse = { aiOperationId: string; activityId: string };
type PublicApiAiOperationStatusResponse = {
  aiOperationId: string;
  status: string;                  // PENDING | STREAMING | ACCEPTED | FAILED | CANCELLED | REJECTED
  proposedContent?: string | null;
};

// --- Images ---
type PublicApiGenerateBlogImageRequest = { prompt?: string | null; fileId?: string | null };
type PublicApiGenerateBlogImageResponse = { liveObjectId: string; activityId?: string | null };
type PublicApiBlogPostImagesResponse = {
  images: { liveObjectId: string; status: string; presignedUrl?: string | null }[];
};

// --- Exports ---
type PublicBlogExportStartedResponse = { exportId: string; status: string };
type PublicBlogExportStatusResponse = {
  data: {
    exportId: string;
    status: string;    // PENDING | IN_PROGRESS | COMPLETED | FAILED (server-defined)
    format: string;    // html | markdown | pdf
    downloadUrl?: string | null;
    expiresAt?: string | null;
    errorMessage?: string | null;
  };
};

// --- Social campaign from blog ---
type PublicApiCreateSocialCampaignFromBlogRequest = {
  targetTopicCount?: number | null;
  webSearchEnabled?: boolean;  // default false
  themeId?: string | null;
  outputLanguage?: string | null;
  textDetailLevel?: string | null;
  tone?: string | null;
};
type PublicApiCreateSocialCampaignFromBlogResponse = {
  projectId: string;
  activityId: string;
  workflowType: string;
  message: string;
};
```

---

## Blog post CRUD

| Action | Method | Path |
|--------|--------|------|
| List (cursor) | GET | `/api/v2/blog-posts?limit=20&cursor=` |
| Get by id | GET | `/api/v2/blog-posts/{blog_post_id}` |
| Update content | PUT | `/api/v2/blog-posts/{blog_post_id}/content` |
| Update meta | PATCH | `/api/v2/blog-posts/{blog_post_id}` |
| Delete | DELETE | `/api/v2/blog-posts/{blog_post_id}` |

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/blog-posts?limit=20" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Generate blog post (create + AI edit)

| Action | Method | Path |
|--------|--------|------|
| Start generation | POST | `/api/v2/blog-posts/generate` |
| Poll generation | GET | `/api/v2/blog-posts/{blog_post_id}/generation/{ai_edit_id}` |
| Accept generation | POST | `/api/v2/blog-posts/{blog_post_id}/generation/{ai_edit_id}/accept` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Write a blog post about cursor pagination in APIs","outputLanguage":"en"}'
```

Notes:

- Use `aiEditLiveObjectId` from the response as `{ai_edit_id}`.
- When poll status returns `ACCEPTED`, call accept to commit it to the blog post.

---

## Outline + section generation (jobs)

| Action | Method | Path | Polling |
|--------|--------|------|---------|
| Generate outline | POST | `/api/v2/blog-posts/{blog_post_id}/generation/outline` | `/api/v2/jobs/{activityId}` |
| Get outline | GET | `/api/v2/blog-posts/{blog_post_id}/outline` | n/a |
| Generate full post | POST | `/api/v2/blog-posts/{blog_post_id}/generation/full-post` | `/api/v2/jobs/{activityId}` |
| Generate section content | POST | `/api/v2/blog-posts/{blog_post_id}/sections/{section_id}/generation/content` | `/api/v2/jobs/{activityId}` |
| Expand section | POST | `/api/v2/blog-posts/{blog_post_id}/sections/{section_id}/generation/expand` | `/api/v2/jobs/{activityId}` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/generation/outline" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Outline for a technical post with examples"}'
```

---

## AI operations (expand / shorten / rewrite a selection)

| Action | Method | Path |
|--------|--------|------|
| Start operation | POST | `/api/v2/blog-posts/{blog_post_id}/ai-operations` |
| Poll status | GET | `/api/v2/blog-posts/{blog_post_id}/ai-operations/{ai_operation_id}` |
| Cancel | POST | `/api/v2/blog-posts/{blog_post_id}/ai-operations/{ai_operation_id}/cancel` |
| Accept (apply) | POST | `/api/v2/blog-posts/{blog_post_id}/ai-operations/{ai_operation_id}/accept` |

Behavior details:

- `command` values: `expand`, `shorten`, `rewrite`
- Server maps `shorten` → internal `summarize`.
- Use `idempotencyKey` (`idempotency_key`) to safely retry start calls.
- Accept will return **409** if the original selection no longer exists in the current content.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/ai-operations" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"command":"rewrite","selection":"<exact text to replace>","prompt":"Make it more concise","idempotency_key":"rewrite-1"}'
```

---

## Images

| Action | Method | Path |
|--------|--------|------|
| Generate image | POST | `/api/v2/blog-posts/{blog_post_id}/generate-image` |
| List resolved images | GET | `/api/v2/blog-posts/{blog_post_id}/images` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/generate-image" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Hero image: abstract gradient waves"}'
```

---

## Cached content (draft autosave)

| Action | Method | Path |
|--------|--------|------|
| Save cached markdown | PUT | `/api/v2/blog-posts/{blog_post_id}/cached-content` |

Returns **204 No Content**.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/cached-content" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"markdown":"# Draft\n\nHello..."}'
```

---

## Exports (async)

| Action | Method | Path |
|--------|--------|------|
| Export HTML | POST | `/api/v2/blog-posts/{blog_post_id}/export/html` |
| Export Markdown | POST | `/api/v2/blog-posts/{blog_post_id}/export/markdown` |
| Export PDF | POST | `/api/v2/blog-posts/{blog_post_id}/export/pdf` |
| Poll export | GET | `/api/v2/blog-posts/{blog_post_id}/exports/{export_id}` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/export/pdf" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Social campaign from blog post

| Action | Method | Path |
|--------|--------|------|
| Start campaign generation | POST | `/api/v2/blog-posts/{blog_post_id}/social-campaign` |

Poll **`GET /api/v2/jobs/{activityId}`**.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/blog-posts/<blog_post_id>/social-campaign" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"targetTopicCount":5,"webSearchEnabled":true}'
```

---

## Agent behavior

1. Resolve `blog_post_id` (list or create via generate).
2. For **jobs-based** endpoints (outline/full/section/expand), poll `/api/v2/jobs/{activityId}` until terminal.
3. For **generate** (ai edit) endpoints, poll `/api/v2/blog-posts/{id}/generation/{ai_edit_id}` until `ACCEPTED`, then call accept.
4. For **AI operations**, poll `/ai-operations/{ai_operation_id}` until `ACCEPTED`, then call accept.
5. For **exports**, poll `/exports/{export_id}` until `COMPLETED`, then use `downloadUrl` (presigned) before `expiresAt`.

---

## Response format (required)

- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL to an image (`presignedUrl`, etc), **render/show the image** and also show the JSON response (do not convert to a table).
