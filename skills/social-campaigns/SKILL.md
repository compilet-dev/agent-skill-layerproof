---
name: social-campaigns
description: Public API social campaigns (X-API-KEY). CRUD campaigns, outline topics, generate/cancel generation, confirm outline, topics, variations/captions/images/carousel, exports (ZIP + bulk rich), theme, citations/history. PublicApiSocialCampaignController (/api/v2/social-campaigns).
---

# Skill: Social Campaigns

## Description

**Social campaigns** are projects with `project_kind` **SOCIAL_CAMPAIGN**. This skill documents **PublicApiSocialCampaignController** at `/api/v2/social-campaigns`. Authenticate with `X-API-KEY`.

- **Campaign generation, variations, captions, theme, image edits, carousel** return `activity_id` — poll **`GET /api/v2/jobs/{activity_id}`** (same as slide workflows).
- **ZIP exports** return `export_id` — poll **`GET /api/v2/social-campaigns/{campaign_id}/exports/{export_id}`** (not the jobs endpoint).
- JSON uses **snake_case** property names (e.g. `topic_ids`, `live_object_ids`).
- **Carousel variations** are multi-slide image sets (convert-to-carousel or generate-carousel-variation).

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

// --- Get campaign: posts include variant_images (presigned URLs, ~1h) ---
type PublicApiPostVariantImage = {
  variation_id: string;
  aspect_ratio_image_urls: Record<string, string>; // ratio key -> presigned GET URL; legacy single path may use key "default"
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
  variant_images?: PublicApiPostVariantImage[];
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
  auto_select?: boolean; // MUST be false (do not use true)
};
type PublicApiGenerateCampaignResponse = {
  campaign_live_object_id: string;
  activity_id: string;
  workflow_type: string;
};

// --- Cancel generation ---
type CancelSocialCampaignGenerationRequest = {
  live_object_ids: string[]; // non-empty; e.g. generation_stream, image_variation, social_post_image_edit roots
};

// --- Confirm outline (after outline step when auto_select is false) ---
type PublicApiConfirmOutlineRequest = {
  selections: { topic_index: number; option_index: number }[];
};

// --- Carousel variations ---
type GenerateCarouselVariationRequest = {
  image_count?: number | null;
  prompt?: string | null;
  reference_image_paths?: string[] | null;
  text_position?: string | null;
  outline_slides?: string[] | null;
  default_aspect_ratio?: string | null;
};
type GenerateCarouselVariationResponse = {
  topic_live_object_id: string;
  carousel_live_object_id: string;
  activity_id: string;
};
type GenerateCarouselAspectRatioRequest = {
  aspect_ratio: string;
  force_overwrite?: boolean;
};
type GenerateCarouselAspectRatioResponse = {
  carousel_live_object_id: string;
  activity_id: string;
};
type GenerateMoreCarouselImagesRequest = {
  additional_count: number;
  prompt?: string | null;
  reference_image_paths?: string[] | null;
};
type GenerateMoreCarouselImagesResponse = {
  carousel_live_object_id: string;
  activity_id: string;
};
type ConvertVariationToCarouselRequest = {
  text_position?: string | null;
};
type ConvertVariationToCarouselResponse = {
  topic_live_object_id: string;
  carousel_variation_id: string;
  carousel_live_object_id: string;
  version: number;
};

// --- Update campaign settings ---
type PublicApiUpdateCampaignSettingsRequest = {
  theme_id?: string | null;
  theme_name?: string | null;
  image_generation_model_preset?: string | null;
  openai_gpt_image_quality?: string | null;
};
type PublicApiUpdateCampaignSettingsResponse = {
  theme_id: string | null;
  theme_name: string | null;
  image_generation_model_preset: string | null;
  openai_gpt_image_quality: string | null;
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

type PostExportItem = {
  topic_id: string;
  variation_ids?: string[] | null;
};
type ExportPostsBulkRequest = {
  posts?: PostExportItem[] | null; // omit or null = all posts
  format?: string; // default "MD"
  aspect_ratios?: string[] | null;
};

// --- Theme ---
type PublicApiGenerateCampaignThemeRequest = {
  prompt: string;
  reference_image_paths?: string[] | null;
};
type PublicApiGenerateCampaignThemeResponse = {
  activity_id: string;
  live_object_id: string;
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

## Outline topics

| Action | Method | Path |
|--------|--------|------|
| List outline (topic options, etc.) | GET | `/api/v2/social-campaigns/{campaign_id}/outline-topics` |

Returns the campaign `outline_topics` array (per-topic options with `core_message`, `key_points`, etc.). Empty when no outline exists or after confirm cleared it. `topic_id` may be synthesized for legacy rows.

---

## Generate, cancel & confirm

| Action | Method | Path |
|--------|--------|------|
| Generate from prompt | POST | `/api/v2/social-campaigns/{campaign_id}/generate` |
| Cancel generation | POST | `/api/v2/social-campaigns/{campaign_id}/cancel-generation` |
| Confirm outline picks | POST | `/api/v2/social-campaigns/{campaign_id}/confirm-outline` |

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/social-campaigns/<campaign_id>/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Holiday campaign","target_topic_count":5,"web_search_enabled":true,"auto_select":false}'
```

Poll **`GET /api/v2/jobs/{activity_id}`** for **`generate`** (and other async work below). If `auto_select` is false, user picks outline options, then **POST confirm-outline** with `selections`.

**Cancel:** body **`{"live_object_ids":["uuid",...]}`**. Response **`200`** full success or **`202`** if some IDs could not be cancelled (`partial_cancellation`).

**Confirm-outline** returns `campaign_live_object_id` and a `workflow_type` of `ConfirmOutline`; use **GET campaign** or **GET outline-topics** to verify state rather than treating `activity_id` like a long-running job.

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
| Convert variation to carousel | POST | `.../posts/{post_id}/variations/{variation_id}/convert-to-carousel` |
| Generate carousel variation | POST | `.../posts/{post_id}/generate-carousel-variation` |
| Generate carousel aspect ratio | POST | `.../posts/{post_id}/carousel-variations/{carousel_variation_id}/generate-aspect-ratio` |
| Generate more carousel images | POST | `.../posts/{post_id}/carousel-variations/{carousel_variation_id}/generate-more-images` |
| Aspect ratio variant | POST | `.../posts/{post_id}/variations/{variation_id}/generate-aspect-ratio` |
| Edit variation image | POST | `.../posts/{post_id}/variations/{variation_id}/edit-image` |
| Accept edit | POST | `.../posts/{post_id}/variations/{variation_id}/accept-edit` |

Poll **`GET /api/v2/jobs/{activity_id}`** for async variation work. **edit-image** typically **auto-applies** the result; **accept-edit** applies a completed edit when needed.

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

Poll **`GET /api/v2/jobs/{activity_id}`** with the returned `activity_id`.

---

## Exports (ZIP)

| Action | Method | Path |
|--------|--------|------|
| Export full campaign ZIP | POST | `/api/v2/social-campaigns/{campaign_id}/exports/zip` |
| Export one topic ZIP | POST | `/api/v2/social-campaigns/{campaign_id}/posts/{post_id}/exports/zip` |
| Bulk rich ZIP (text + images) | POST | `/api/v2/social-campaigns/{campaign_id}/posts/exports/bulk-rich` |
| Poll export | GET | `/api/v2/social-campaigns/{campaign_id}/exports/{export_id}` |

**bulk-rich:** optional body with `posts` (list of `{ topic_id, variation_ids? }`), `format` (default `MD`), `aspect_ratios`. Omit `posts` to export all posts.

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
5. This controller does **not** expose `generate-platform`, per-platform PUT posts, legacy `image-edit` on the topic, or PNG platform exports; platform-specific read APIs are **citations** and **history** above.

---

## Response format (required)

- When responses include **image URLs** for posts or variations, **render previews** and show raw JSON.
- Always show **verbatim JSON** in a code block.
