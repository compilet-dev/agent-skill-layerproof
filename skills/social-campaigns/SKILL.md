---
name: social-campaigns
description: Public API social campaign operations (X-API-KEY). Create, list, get, update, delete campaigns; topics (create, update, delete, reorder); generate campaign, generate platform post, update platform post, edit platform image; export ZIP (campaign or post); get export status. Types follow PublicApiSocialCampaignController (/api/v2/social-campaigns).
---

# Skill: Social Campaigns

## Description

Manage social campaigns, topics (posts), and platform-specific content. This skill documents the **public API** at `/api/v2/social-campaigns` (PublicApiSocialCampaignController). Endpoints use `campaign_id` and optionally `post_id`, `platform`, `export_id` in the path. Authenticate with `X-API-KEY` header. Poll async operations via `GET /api/v2/jobs/{activity_id}` or, for exports, `GET .../exports/{export_id}`.

---

## TypeScript types (request / response)

Mirrors `PublicApiSocialCampaignController` data classes.

```typescript
// --- Create Campaign (POST) ---
type PublicApiCreateCampaignRequest = {
  name: string;                    // 1–255 chars, required
  description?: string | null;     // max 10000
  workspace_id?: string | null;
};
type PublicApiCampaignResponse = {
  campaign_id: string;
  name: string;
  description?: string | null;
  campaign_live_object_id: string;
  created_at: string;
  updated_at: string;
};

// --- List Campaigns (GET) ---
type PublicApiCampaignListResponse = {
  data: PublicApiCampaignResponse[];
  next_cursor?: string | null;
  has_more: boolean;
};

// --- Get Campaign by ID (GET) ---
type PublicApiGenerationProgress = {
  status: string;
  completed_posts: number;
  total_posts: number;
  current_step?: string | null;
  error_message?: string | null;
};
type PublicApiPostSummary = {
  post_id: string;
  live_object_id: string;
  title?: string | null;
  status: string;
  topic?: string | null;
  core_message?: string | null;
  key_points?: string[] | null;
  visual_suggestion?: string | null;
  version?: number | null;
};
type PublicApiCampaignWithPostsResponse = {
  campaign_id: string;
  name: string;
  description?: string | null;
  campaign_live_object_id: string;
  theme_id?: string | null;
  generation_progress?: PublicApiGenerationProgress | null;
  posts: PublicApiPostSummary[];
  created_at: string;
  updated_at: string;
};

// --- Update Campaign (PUT) ---
type PublicApiUpdateCampaignRequest = {
  name?: string | null;            // 1–255 chars
  description?: string | null;     // max 10000
  workspace_id?: string | null;
};

// --- Update Campaign Settings (PUT) ---
type PublicApiUpdateCampaignSettingsRequest = {
  theme_id?: string | null;
  theme_name?: string | null;
};
type PublicApiUpdateCampaignSettingsResponse = {
  theme_id?: string | null;
  theme_name?: string | null;
};

// --- Generate Campaign (POST) — async ---
type PublicApiToneConfig = {
  output_language?: string | null;
  text_detail_level?: string | null;
  tone?: string | null;
  voice?: string | null;
  audience?: string | null;
};
type PublicApiGenerateCampaignRequest = {
  prompt: string;                   // 1–10000 chars, required
  theme_id?: string | null;
  tone_config?: PublicApiToneConfig | null;
  reference_file_paths?: string[] | null;
  web_search_enabled?: boolean;
  target_topic_count?: number | null;
};
type PublicApiGenerateCampaignResponse = {
  campaign_live_object_id: string;
  activity_id: string;
  workflow_type: string;
};

// --- Create Topic (POST) ---
type PublicApiCreateTopicRequest = {
  topic?: string | null;
  core_message?: string | null;
  key_points?: string[] | null;
  visual_suggestion?: string | null;
};
type PublicApiTopicResponse = {
  topic_id: string;
  live_object_id: string;
  status: string;
  topic?: string | null;
  core_message?: string | null;
  key_points?: string[] | null;
  visual_suggestion?: string | null;
  version: number;
  created_at: string;
  updated_at: string;
};

// --- Update Topic (PUT) ---
type UpdateTopicRequest = {
  topic?: string | null;
  core_message?: string | null;
  key_points?: string[] | null;
  visual_suggestion?: string | null;
};

// --- Reorder Topics (PUT) ---
type PublicApiReorderRequest = {
  topic_ids: string[];               // required, must match campaign topics
};
type ReorderTopicsResponse = {
  topic_ids: string[];
  version: number;
  updated_at: string;
};

// --- Generate Platform Post (POST) — async ---
type PublicApiGeneratePlatformPostRequest = {
  platform: string;                 // required: INSTAGRAM_POST, INSTAGRAM_STORY, LINKEDIN, TWITTER_X, FACEBOOK, TIKTOK
};
type PublicApiGeneratePlatformPostResponse = {
  platform_post_live_object_id: string;
  activity_id: string;
  workflow_type: string;
};

// --- Update Platform Post (PUT) ---
type UpdatePlatformPostRequest = {
  caption?: string | null;
  hashtags?: string[] | null;
  image_path?: string | null;
};
type UpdatePlatformPostResponse = {
  post_id: string;
  platform: string;
  version: number;
  updated_at: string;
};

// --- Edit Platform Image (POST) — async ---
type CroppedRegion = {
  x: number;
  y: number;
  width: number;
  height: number;
};
type EditPlatformImageRequest = {
  instruction: string;              // required
  input_image_path?: string | null;   // omit to use current platform post image
  platform: string;                 // required
  other_reference_image_paths?: string[] | null;
  cropped_region?: CroppedRegion | null;
};
type EditPlatformImageResponse = {
  live_object_id: string;
  activity_id: string;
  workflow_type: string;
};

// --- Export ZIP (POST) — async ---
type PublicApiCampaignExportStartedResponse = {
  export_id: string;
  status: string;                   // "PENDING"
};

// --- Get Export Status (GET) ---
type PublicApiCampaignExportStatusResponse = {
  export_id: string;
  status: string;                   // IN_PROGRESS | COMPLETED | FAILED
  download_url?: string | null;
  expires_at?: string | null;
  file_size_bytes?: number | null;
  error_message?: string | null;
};
```

---

## Create Campaign

Request body: `PublicApiCreateCampaignRequest`. Response (201): `PublicApiCampaignResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"Q1 Launch","description":"Product launch posts","workspace_id":"<workspace_uuid>"}'
```

---

## List Campaigns

Query: `limit` (default 20, max 100), `cursor` (optional). Response: `PublicApiCampaignListResponse`.

Cursor-paginated list of campaigns owned by the authenticated user.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/social-campaigns?limit=20&cursor=<next_cursor>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Get Campaign by ID

Path: `campaign_id`. Response: `PublicApiCampaignWithPostsResponse`.

Returns campaign detail with topics (posts) and generation progress. 403 if not owner.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Update Campaign

Path: `campaign_id`. Request body: `PublicApiUpdateCampaignRequest`. Response: `PublicApiCampaignResponse`.

Only provided fields are updated (name, description, workspace_id).

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"Updated name","description":"New description"}'
```

---

## Update Campaign Settings

Path: `campaign_id`. Request body: `PublicApiUpdateCampaignSettingsRequest`. Response: `PublicApiUpdateCampaignSettingsResponse`.

Updates theme (theme_id, theme_name). Applied immediately.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/settings" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"theme_id":"<theme_uuid>","theme_name":"My theme"}'
```

---

## Generate Campaign (async)

Path: `campaign_id`. Request body: `PublicApiGenerateCampaignRequest`. Response (201): `PublicApiGenerateCampaignResponse`.

Starts campaign generation from a prompt. Poll `GET /api/v2/jobs/{activity_id}` for status; get campaign by ID to see `generation_progress` and posts.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Launch our new app","target_topic_count":5,"web_search_enabled":true}'
```

---

## Delete Campaign

Path: `campaign_id`. Response: 204 No Content.

Soft-deletes the campaign.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Create Topic

Path: `campaign_id`. Request body: `PublicApiCreateTopicRequest` or omit (blank topic). Response (201): `PublicApiTopicResponse`.

Add a topic (post) to the campaign.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"topic":"Feature highlight","core_message":"Key benefit","key_points":["Point 1"],"visual_suggestion":"Minimal icon"}'
```

---

## Update Topic

Path: `campaign_id`, `post_id`. Request body: `UpdateTopicRequest`. Response: `PublicApiTopicResponse`.

Only provided fields are updated.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"topic":"Updated title","core_message":"Updated message"}'
```

---

## Delete Topic

Path: `campaign_id`, `post_id`. Response: 204 No Content.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Reorder Topics

Path: `campaign_id`. Request body: `PublicApiReorderRequest`. Response: `ReorderTopicsResponse`.

Body: `{ "topic_ids": ["uuid", ...] }`. Order must match all topic IDs in the campaign.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/reorder" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"topic_ids":["<uuid1>","<uuid2>"]}'
```

---

## Generate Platform Post (async)

Path: `campaign_id`, `post_id`. Request body: `PublicApiGeneratePlatformPostRequest`. Response (201): `PublicApiGeneratePlatformPostResponse`.

Platform: `INSTAGRAM_POST`, `INSTAGRAM_STORY`, `LINKEDIN`, `TWITTER_X`, `FACEBOOK`, `TIKTOK`. Poll `GET /api/v2/jobs/{activity_id}`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>/generate-platform" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"platform":"LINKEDIN"}'
```

---

## Update Platform Post

Path: `campaign_id`, `post_id`, `platform`. Request body: `UpdatePlatformPostRequest`. Response: `UpdatePlatformPostResponse`.

Update caption, hashtags, or image path for a platform-specific post.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>/platforms/LINKEDIN" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"caption":"New caption","hashtags":["#product"]}'
```

---

## Edit Platform Image (async)

Path: `campaign_id`, `post_id`. Request body: `EditPlatformImageRequest`. Response (201): `EditPlatformImageResponse`.

Start AI image edit for a platform post. Omit `input_image_path` to use current platform post image. Edited image is auto-applied. Poll `GET /api/v2/jobs/{activity_id}`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>/image-edit" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"instruction":"Add a sunset background","platform":"INSTAGRAM_POST"}'
```

---

## Export Campaign as ZIP (async)

Path: `campaign_id`. Response (202): `PublicApiCampaignExportStartedResponse`.

Exports all completed platform post images by platform folder. Poll `GET .../exports/{export_id}` for status and download URL.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/exports/zip" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Export Post as ZIP (async)

Path: `campaign_id`, `post_id`. Query: `platform` (optional, repeat for multiple; omit for all). Response (202): `PublicApiCampaignExportStartedResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/posts/<post_id>/exports/zip?platform=LINKEDIN&platform=INSTAGRAM_POST" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Get Export Status

Path: `campaign_id`, `export_id`. Response: `PublicApiCampaignExportStatusResponse`.

Poll until `status` is `COMPLETED` (then use `download_url`) or `FAILED` (then use `error_message`).

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/exports/<export_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Agent behavior

When the user asks to work with social campaigns (campaigns, topics, posts, platform posts, exports), do the following.

### 1. Choose the right endpoint

Base path: `/api/v2/social-campaigns`. All require `X-API-KEY`.

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Create campaign | `/api/v2/social-campaigns` | POST |
| List campaigns | `/api/v2/social-campaigns` (limit, cursor) | GET |
| Get campaign with posts | `/api/v2/social-campaigns/{campaign_id}` | GET |
| Update campaign | `/api/v2/social-campaigns/{campaign_id}` | PUT |
| Update campaign settings | `/api/v2/social-campaigns/{campaign_id}/settings` | PUT |
| Generate campaign | `/api/v2/social-campaigns/{campaign_id}/generate` | POST |
| Delete campaign | `/api/v2/social-campaigns/{campaign_id}` | DELETE |
| Create topic | `/api/v2/social-campaigns/{campaign_id}/posts` | POST |
| Update topic | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}` | PUT |
| Delete topic | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}` | DELETE |
| Reorder topics | `/api/v2/social-campaigns/{campaign_id}/reorder` | PUT |
| Generate platform post | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/generate-platform` | POST |
| Update platform post | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/platforms/{platform}` | PUT |
| Edit platform image | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/image-edit` | POST |
| Export campaign ZIP | `/api/v2/social-campaigns/{campaign_id}/exports/zip` | POST |
| Export post ZIP | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/exports/zip` | POST |
| Get export status | `/api/v2/social-campaigns/{campaign_id}/exports/{export_id}` | GET |

### 2. Build and run the request

- **Auth**: Every request must include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **Path**: Resolve `campaign_id`, `post_id`, `platform`, `export_id` from context or user input. If missing, ask.
- **GET**: Build curl with path and query params (`limit`, `cursor` for list; `platform` for post export). Run and show result.
- **POST/PUT**: Build JSON body from the types above. Use `-X POST` or `-X PUT`, `-H "Content-Type: application/json"`, and `-d '...'`. Run and show result.

### 3. After async endpoints

- **Generate campaign / Generate platform post / Edit platform image**: Responses include `activity_id`. Poll `GET $LAYERPROOF_BASE_URL/api/v2/jobs/{activity_id}` until `status` is `DONE` or `CANCELED`.
- **Export ZIP**: Response (202) includes `export_id`. Poll `GET $LAYERPROOF_BASE_URL/api/v2/social-campaigns/{campaign_id}/exports/{export_id}` until `status` is `COMPLETED` or `FAILED`; then use `download_url` or `error_message`.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block; do not convert to a table.
- If the response contains image URLs, show images and the JSON.
- On error (4xx/5xx), show the response body and status code; suggest fixing API key, IDs, or request body.

### 5. Show preview images for social posts

When presenting social campaign data that includes **posts** or **platform-specific content**, the agent **must** show preview images so the user can see how posts will look.

- **Get campaign with posts** (GET `.../social-campaigns/{campaign_id}`): If the response includes per-post or per-platform image URLs (e.g. in `posts[]`, or in platform post details such as `image_url`, `image_path`, `preview_url`, or similar), **render each image** as a preview. Label each preview with the post topic/title and platform (e.g. "Post 1 — LinkedIn", "Post 2 — Instagram") when multiple platforms or posts exist. Then show the raw JSON in a code block.
- **After generate platform post or edit platform image**: When you fetch the campaign or post to show the result (e.g. GET campaign by ID or a dedicated get-platform-post endpoint), **display the preview image(s)** for the updated post/platform. Do not only show JSON when image URLs are present — show the images first or alongside, then the JSON.
- **Listing or comparing posts**: If the user asks to "show my posts", "preview the campaign", or "show how the LinkedIn posts look", resolve the campaign and any endpoints that return platform post assets, then **show all available preview images** (one per post and/or per platform) with clear labels, followed by the JSON.
- If the API returns a full URL (e.g. presigned or `image_url`), **render/show the image** in the agent response. If it returns only a path, build the full URL from `LAYERPROOF_BASE_URL` or use a documented download/preview endpoint if available, then show the image.

### 6. Example workflows

**Workflow A — User**: "Create a social campaign and generate 3 topics from a prompt."

1. POST `/api/v2/social-campaigns` with `{"name":"My Campaign"}`; get `campaign_id` and `campaign_live_object_id`.
2. POST `.../generate` with `{"prompt":"...", "target_topic_count":3}`; get `activity_id`.
3. Tell user to poll `GET /api/v2/jobs/{activity_id}` until DONE, then GET campaign by ID to see posts.

**Workflow B — User**: "Full campaign: create campaign, generate topics, reorder them, generate LinkedIn and Instagram for the first two posts, then export the campaign as ZIP."

1. POST `/api/v2/social-campaigns` with `{"name":"Q2 Launch", "description":"..."}`; capture `campaign_id`.
2. POST `.../generate` with `{"prompt":"...", "target_topic_count":4, "tone_config": {...}}`; get `activity_id`.
3. Poll `GET /api/v2/jobs/{activity_id}` until DONE. If `failure_reason` present, report and stop.
4. GET `.../social-campaigns/{campaign_id}`; from `posts` collect `post_id` and desired order (e.g. swap first two).
5. PUT `.../reorder` with `{"topic_ids": [id2, id1, id3, id4]}` so order matches the desired sequence.
6. For first post: POST `.../posts/{post_id_1}/generate-platform` with `{"platform":"LINKEDIN"}`; capture `activity_id_1`. For second post: same with `LINKEDIN`, then repeat for `INSTAGRAM_POST` for both if needed. Run and capture each `activity_id`.
7. Poll each `GET /api/v2/jobs/{activity_id}` until DONE (handle multiple jobs; if any fail, report which post/platform failed).
8. Optional: PUT `.../posts/{post_id}/platforms/LINKEDIN` to tweak `caption` or `hashtags` for one or more posts.
9. POST `.../social-campaigns/{campaign_id}/exports/zip`; get `export_id`.
10. Poll `GET .../social-campaigns/{campaign_id}/exports/{export_id}` until `status` is COMPLETED; show `download_url` (or FAILED and `error_message`).

**Workflow C — User**: "Edit the Instagram image for post X with instruction 'add sunset background'."

1. Resolve `campaign_id` and `post_id` (from list campaigns → get campaign with posts, or user input).
2. POST `.../posts/{post_id}/image-edit` with `{"instruction":"Add a sunset background", "platform":"INSTAGRAM_POST"}`; get `activity_id` and optionally `live_object_id`.
3. Poll `GET /api/v2/jobs/{activity_id}` until DONE; the edited image is auto-applied (no separate accept step for this API). GET campaign/post to show updated asset.

---

## Response format (required)

- **Preview images for social posts**: When the response contains image URLs for social posts (campaign posts, platform posts, or post exports), **always show the preview images** — render each image and label it (e.g. post index, topic, platform). Then show the raw JSON in a code block. Do not convert image-bearing responses to a table; show images + JSON.
- For any other response that contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
- Always show the **raw JSON response** (verbatim) in a JSON code block.
