---
name: themes
description: Public API theme management (X-API-KEY). List, get, save, update, delete, generate, regenerate, apply, unapply. PublicThemeController (/api/v2/themes).
---

# Skill: Theme Management

## Description

Themes define visual styling for slides. This skill documents the **public API** at `/api/v2/themes` (PublicThemeController). Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicThemeController` (/api/v2/themes) data classes.

```typescript
// --- List Themes (GET) ---
// Query: offset (default 0), limit (default 20), search (optional)
type PublicThemeListResponse = {
  data: PublicThemeResponse[];
  total: number;
  offset: number;
  limit: number;
};

// --- Get Theme By ID (GET) ---
type PublicThemeResponse = {
  id: string;
  name: string;
  description: string | null;
  visibility: string;       // "PRIVATE" | "SYSTEM" | "SHARED"
  preview_url: string;
  created_at: string;       // ISO 8601
  updated_at: string;      // ISO 8601
};

// --- Generate Theme (POST) — async ---
type PublicGenerateThemeRequest = {
  prompt: string;           // required, max 5000 chars
  project_id?: string | null;  // optional THEME project UUID; if omitted, a project is created automatically
};
type PublicGenerateThemeResponse = {
  activity_id: string;      // UUID – poll GET /api/v2/jobs/{activity_id}
  theme_id: string;        // UUID
};

// --- Apply Theme (POST) ---
type PublicApplyThemeRequest = {
  slide_deck_id: string;   // UUID, required
  theme_id: string;        // UUID, required
  regenerate_slides?: boolean;  // default false; if true, triggers batch image regeneration – poll activity_id
};
type PublicApplyThemeResponse = {
  theme_id: string;
  theme_name: string;
  slide_deck_id: string;
  applied: boolean;        // true when applied successfully
  activity_id: string | null;  // set when regenerate_slides is true – poll GET /api/v2/jobs/{activity_id}
};

// --- Save theme (POST /api/v2/themes) — 201 ---
type PublicSaveThemeRequest = {
  name: string;                              // required, max 256
  visual_style_description: string;         // required, max 5000
  preview_s3_key: string;                    // required — S3 key of preview image
  description?: string | null;              // max 1000
  source_prompt?: string | null;
  source_project_id?: string | null;
  tags?: string[] | null;
};

// --- Update theme (PUT /{theme_id}) ---
type PublicUpdateThemeRequest = {
  name?: string | null;
  description?: string | null;
  tags?: string[] | null;
  visual_style_description?: string | null;
};

// --- Regenerate theme (POST /{theme_id}/regenerate) — 202 ---
type PublicRegenerateThemeRequest = {
  project_id: string;  // UUID — project whose working dir is used
  adjustment_prompt: string;
  reference_image_paths?: string[] | null;
  preview_only?: boolean;
};
type PublicRegenerateThemeResponse = {
  activity_id: string;
  theme_id: string;
  live_object_id: string;
};

// --- Unapply (POST /unapply) ---
type PublicUnapplyThemeRequest = {
  project_id: string;
  slide_deck_id: string;
};
```

---

## List Themes

Query: `offset` (default 0), `limit` (default 20), `search` (optional). Response: `PublicThemeListResponse`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/themes?offset=0&limit=20" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Get Theme By ID

Response: `PublicThemeResponse`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/themes/<theme_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Generate Theme

Request body: `PublicGenerateThemeRequest`. Response (202): `PublicGenerateThemeResponse`.

Only `prompt` is required. Optional `project_id` uses an existing THEME project; if omitted, one is created automatically. Poll `GET /api/v2/jobs/{activityId}` for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Clean, minimal style with SF Pro"}'
```

With optional `project_id`:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Clean, minimal style","project_id":"<theme_project_uuid>"}'
```

---

## Apply Theme

Request body: `PublicApplyThemeRequest`. Response: `PublicApplyThemeResponse`.

Theme and slide deck are specified in the body. When `regenerate_slides` is true, poll `GET /api/v2/jobs/{activityId}` for batch image regeneration status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/apply" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slide_deck_id":"<slide_deck_uuid>","theme_id":"<theme_uuid>"}'
```

With slide regeneration:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/apply" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slide_deck_id":"<slide_deck_uuid>","theme_id":"<theme_uuid>","regenerate_slides":true}'
```

---

## Save Theme (manual)

Creates a saved private theme. Requires `visual_style_description` and `preview_s3_key` (upload preview via project/public files first). Response (201): `PublicThemeResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"My brand","visual_style_description":"Minimal dark UI, rounded cards","preview_s3_key":"<s3_key>"}'
```

---

## Update Theme

Request body: `PublicUpdateThemeRequest`. Response: `PublicThemeResponse`.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/themes/<theme_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"Renamed theme"}'
```

---

## Delete Theme

Soft-deletes a user-owned theme. Response: 204 No Content.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/themes/<theme_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Regenerate Theme (async)

Request body: `PublicRegenerateThemeRequest`. Response (202): `PublicRegenerateThemeResponse`. Poll `GET /api/v2/jobs/{activity_id}`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/<theme_id>/regenerate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"project_id":"<project_uuid>","adjustment_prompt":"More contrast, warmer palette"}'
```

---

## Unapply Theme from Slide Deck

Removes applied theme from a deck. Request body: `PublicUnapplyThemeRequest`. Response: 204 No Content.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/unapply" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"project_id":"<project_uuid>","slide_deck_id":"<slide_deck_uuid>"}'
```

---

## List My Themes (by user)

Query: `offset`, `limit`, optional `visibility` (`PRIVATE` \| `SYSTEM` \| `SHARED`), optional `search`. Response: `PublicThemeListResponse`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/themes/by-user-id?offset=0&limit=20" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Agent behavior

When the user asks to work with themes (list, get, generate, apply), do the following.

### 1. Choose the right endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| List/browse themes, search themes | `/api/v2/themes` | GET |
| Get one theme by ID | `/api/v2/themes/<theme_id>` | GET |
| Save a theme manually | `/api/v2/themes` | POST |
| Update theme metadata | `/api/v2/themes/<theme_id>` | PUT |
| Delete theme | `/api/v2/themes/<theme_id>` | DELETE |
| Generate theme from prompt (async) | `/api/v2/themes/generate` | POST |
| Regenerate / adjust theme (async) | `/api/v2/themes/<theme_id>/regenerate` | POST |
| Apply a theme to a slide deck | `/api/v2/themes/apply` | POST |
| Unapply theme from deck | `/api/v2/themes/unapply` | POST |
| List current user’s themes | `/api/v2/themes/by-user-id` | GET |

### 2. Build and run the request

- **Auth**: Every request must include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **GET**: Build `curl` with the chosen path and query params (`offset`, `limit`, `search` for list). Run the curl and show the result.
- **POST**: Build a JSON body from the user's input (prompt, theme ID, slide deck ID, etc.). Use `-X POST`, `-H "Content-Type: application/json"`, and `-d '...'`. Run the curl and show the result.

### 3. After generate or apply (with regeneration)

- **Generate theme**: Response includes `activity_id` and `theme_id`. Tell the user the theme was started and give `theme_id`. Optionally poll `GET $LAYERPROOF_BASE_URL/api/v2/jobs/<activity_id>` until `status` is `DONE` or `CANCELED`, then report outcome.
- **Apply theme with `regenerate_slides: true`**: Response may include `activity_id`. If present, tell the user regeneration was started and optionally poll `GET .../api/v2/jobs/<activity_id>` for status.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block; do not convert to a table.
- If the response contains a URL for an image (e.g. `preview_url`), show the image and the JSON.
- On error (4xx/5xx), show the response body and status code; suggest fixing missing/invalid API key, IDs, or request body.

### 5. Example workflows

**Workflow A — User**: "Generate a theme with prompt: minimal dark mode."

1. Choose `POST /api/v2/themes/generate`.
2. Build body: `{"prompt":"minimal dark mode"}`.
3. Run: `curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" -H "Content-Type: application/json" -H "X-API-KEY: $LAYERPROOF_API_KEY" -d '{"prompt":"minimal dark mode"}'`.
4. Show the JSON response; if it contains `activity_id`, mention they can poll `/api/v2/jobs/{activityId}` for status and use `theme_id` once done.

**Workflow B — User**: "List themes, generate a new 'corporate blue' theme, wait for it to finish, then apply it to my slide deck and regenerate slides."

1. GET `/api/v2/themes` with optional `limit`, `offset`, `search`; show list. User may pick existing or request new.
2. POST `/api/v2/themes/generate` with `{"prompt":"corporate blue"}`; capture `activity_id` and `theme_id`.
3. Poll `GET /api/v2/jobs/{activity_id}` until status is DONE (or CANCELED). If DONE, theme is ready; if failed, report `failure_reason`.
4. Resolve projectId and slideDeckId (projects + slide-deck). POST or PUT the slide-deck theme/settings endpoint with `theme_id` (e.g. PUT `.../slide-deck/.../settings` with `{"theme_id":"<theme_id>"}`).
5. Use **POST `/api/v2/themes/apply`** with `regenerate_slides: true` if you want slide images regenerated; capture `activity_id` and poll jobs until DONE. Otherwise apply with `regenerate_slides: false` or omit.

**Workflow C — User**: "I have a theme ID; apply it to deck X and only update the look (no slide regeneration)."

1. **POST `/api/v2/themes/apply`** with `{"slide_deck_id":"<slide_deck_uuid>","theme_id":"<theme_uuid>"}` (omit `regenerate_slides` or set `false`).
2. Confirm with GET deck; no job polling needed unless `regenerate_slides` was true.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
