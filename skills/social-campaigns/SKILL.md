---
name: social-campaigns
description: Public API social campaigns (X-API-KEY). CRUD campaigns, generate, confirm outline, topics, variations, captions, exports, theme, citations. PublicApiSocialCampaignController (/api/v2/social-campaigns).
---

# Skill: Social Campaigns

## Description

**Social campaigns** are projects with `project_kind` **SOCIAL_CAMPAIGN**. This skill documents **PublicApiSocialCampaignController** at `/api/v2/social-campaigns`. Authenticate with `X-API-KEY`.

- **Campaign generation** returns `activity_id` — poll **`GET /api/v2/jobs/{activity_id}`** (same as slide workflows).
- **ZIP exports** return `export_id` — poll **`GET /api/v2/social-campaigns/{campaign_id}/exports/{export_id}`** (not the jobs endpoint).

---

## TypeScript types (selected)

```typescript
// --- Create / list / get ---
type PublicApiCreateCampaignRequest = {
  name: string;
  description?: string | null;
  status?: string | null;
  tags?: string[] | null;
  metadata?: Record<string, unknown> | null;
  workspace_id?: string | null;
  theme_id?: string | null;
  aspect_ratio?: string | null;
};
type PublicApiCampaignResponse = {
  campaign_id: string;
  name: string;
  description?: string | null;
  campaign_live_object_id: string;
  created_at: string;
  updated_at: string;
};
type PublicApiCampaignListResponse = {
  data: PublicApiCampaignResponse[];
  next_cursor?: string | null;
  has_more: boolean;
};

// --- Generate campaign ---
type PublicApiTrendSnapshot = {
  key: string;
  label: string;
  platform: string;
  region?: string | null;
  score?: number | null;
  // ... additional optional analytics fields
};
type PublicApiGenerateCampaignRequest = {
  prompt: string;
  theme_id?: string | null;
  tone_config?: {
    output_language?: string | null;
    text_detail_level?: string | null;
    tone?: string | null;
    voice?: string | null;
    audience?: string | null;
  } | null;
  reference_file_paths?: string[] | null;
  web_search_enabled?: boolean;
  target_topic_count?: number | null;
  trend_snapshot?: PublicApiTrendSnapshot | null;
  trend_snapshots?: PublicApiTrendSnapshot[] | null;
  auto_select?: boolean;  // default false — if true, skips outline confirmation
};
type PublicApiGenerateCampaignResponse = {
  campaign_live_object_id: string;
  activity_id: string;
  workflow_type: string;
};

// --- Confirm outline (after outline step when auto_select is false) ---
type PublicApiConfirmOutlineRequest = {
  selections: { topic_index: number; option_index: number }[];
};

// --- Topics (“posts” in paths) ---
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

// --- Export ---
type PublicApiCampaignExportStartedResponse = {
  export_id: string;
  status: string;
};
type PublicApiCampaignExportStatusResponse = {
  export_id: string;
  status: 'IN_PROGRESS' | 'COMPLETED' | 'FAILED';
  download_url?: string | null;
  expires_at?: string | null;
  file_size_bytes?: number | null;
  error_message?: string | null;
};
```

---

## Campaign CRUD

| Action | Method | Path |
|--------|--------|------|
| Create | POST | `/api/v2/social-campaigns` |
| List (cursor) | GET | `/api/v2/social-campaigns?limit=20&cursor=` |
| Get + posts | GET | `/api/v2/social-campaigns/{campaign_id}` |
| Update | PUT | `/api/v2/social-campaigns/{campaign_id}` |
| Update settings (theme) | PUT | `/api/v2/social-campaigns/{campaign_id}/settings` |
| Delete | DELETE | `/api/v2/social-campaigns/{campaign_id}` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"Q1 Launch","workspace_id":"<workspace_uuid>"}'
```

---

## Generate & confirm

| Action | Method | Path |
|--------|--------|------|
| Generate from prompt | POST | `/api/v2/social-campaigns/{campaign_id}/generate` |
| Confirm outline picks | POST | `/api/v2/social-campaigns/{campaign_id}/confirm-outline` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Holiday campaign","target_topic_count":5,"web_search_enabled":true,"auto_select":false}'
```

Poll **`GET /api/v2/jobs/{activity_id}`**. If `auto_select` is false, user picks outline options, then **POST confirm-outline** with `selections`.

---

## Topics (paths use `/posts`)

| Action | Method | Path |
|--------|--------|------|
| Create topic | POST | `/api/v2/social-campaigns/{campaign_id}/posts` |
| Update topic | PUT | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}` |
| Delete topic | DELETE | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}` |
| Reorder | PUT | `/api/v2/social-campaigns/{campaign_id}/reorder` body `{"topic_ids":["uuid",...]}` |

---

## Variations & images

| Action | Method | Path |
|--------|--------|------|
| Generate variations | POST | `.../posts/{post_id}/generate-variations` |
| More variations | POST | `.../posts/{post_id}/generate-more-variations` |
| Delete variation | DELETE | `.../posts/{post_id}/variations/{variation_id}` |
| Retry variation | POST | `.../posts/{post_id}/variations/{variation_id}/retry` |
| Aspect ratio variant | POST | `.../posts/{post_id}/variations/{variation_id}/generate-aspect-ratio` |
| Edit variation image | POST | `.../posts/{post_id}/variations/{variation_id}/edit-image` |
| Accept edit (if needed) | POST | `.../posts/{post_id}/variations/{variation_id}/accept-edit` |

Poll **`GET /api/v2/jobs/{activity_id}`** for async variation work.

---

## Captions

| Action | Method | Path |
|--------|--------|------|
| Update caption / hashtags | PUT | `.../posts/{post_id}/caption` |
| Generate caption (async) | POST | `.../posts/{post_id}/generate-caption` |

---

## Theme

| Action | Method | Path |
|--------|--------|------|
| Generate campaign theme (async) | POST | `/api/v2/social-campaigns/{campaign_id}/generate-theme` |

Poll jobs with returned activity id.

---

## Exports (ZIP)

| Action | Method | Path |
|--------|--------|------|
| Export full campaign ZIP | POST | `/api/v2/social-campaigns/{campaign_id}/exports/zip` |
| Export one topic ZIP | POST | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/exports/zip` |
| Poll export | GET | `/api/v2/social-campaigns/{campaign_id}/exports/{export_id}` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/exports/zip" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Citations & history

| Action | Method | Path |
|--------|--------|------|
| Campaign citations | GET | `/api/v2/social-campaigns/{campaign_id}/citations` |
| Post citations | GET | `.../posts/{post_id}/citations` |
| Platform post citations | GET | `.../posts/{post_id}/platforms/{platform}/citations` |
| Platform generation history | GET | `.../posts/{post_id}/platforms/{platform}/history` |
| Platform edit history | GET | `.../posts/{post_id}/platforms/{platform}/edits` |

---

## Agent behavior

**Before any API call, verify environment variables are set:**

```bash
if [[ -z "${LAYERPROOF_BASE_URL}" ]]; then
  echo "ERROR: LAYERPROOF_BASE_URL is not set."
  return 1
fi
if [[ -z "${LAYERPROOF_API_KEY}" ]]; then
  echo "ERROR: LAYERPROOF_API_KEY is not set."
  return 1
fi

# Load .env.local if present
if [[ -f .env.local ]]; then
  set -a
  source .env.local
  set +a
fi
```

1. Resolve **`campaign_id`** (create or list). **`campaign_live_object_id`** in responses is the campaign root live object.
2. **Generate** → poll **jobs** until `DONE` / handle `failure_reason`.
3. **Exports** → poll **`.../social-campaigns/{id}/exports/{export_id}`** until `COMPLETED` or `FAILED`.
4. **Topics** are listed under **`posts`** in API paths; treat `post_id` as **topic id**.
5. There is **no** `generate-platform` or per-platform image-edit in this public controller; platform-specific data appears in **get** campaign and **citations/history** endpoints when present.

---

## Response format (required)

- When responses include **image URLs** for posts or variations, **render previews** and show raw JSON.
- Always show **verbatim JSON** in a code block.
