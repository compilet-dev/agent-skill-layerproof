---
name: themes
description: Public API theme management (X-API-KEY). List, get, generate, and apply themes. Types follow PublicThemeController (/api/v2/themes).
---

# Skill: Theme Management

## Description

Themes define visual styling for slides. This skill documents the **public API** at `/api/v2/themes` (PublicThemeController). Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicThemeController` (/api/v2/themes) data classes.

```typescript
// --- List Themes (GET) ---
// Query: offset?: number (default 0), limit?: number (default 20), search?: string
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
  visibility: string;
  previewUrl: string;
  createdAt: string;   // ISO 8601
  updatedAt: string;   // ISO 8601
};

// --- Generate Theme (POST) — PublicGenerateThemeRequest / PublicGenerateThemeResponse ---
type PublicGenerateThemeRequest = {
  /** Theme generation prompt (required, max 5000 characters) */
  prompt: string;
  /** Optional THEME project ID; if omitted, a project is created automatically */
  projectId?: string;
};

type PublicGenerateThemeResponse = {
  activityId: string;  // UUID – poll GET /api/v2/jobs/{activityId}
  themeId: string;      // UUID
};

// --- Apply Theme (POST) — PublicApplyThemeRequest / PublicApplyThemeResponse ---
type PublicApplyThemeRequest = {
  slideDeckId: string;   // UUID
  themeId: string;       // UUID
  /** If true, triggers batch image regeneration; poll activityId when set */
  regenerateSlides?: boolean;  // default false
};

type PublicApplyThemeResponse = {
  themeId: string;
  themeName: string;
  slideDeckId: string;
  applied: boolean;
  activityId: string | null;  // set when regenerateSlides is true – poll GET /api/v2/jobs/{activityId}
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

Only `prompt` is required. Optional `projectId` uses an existing THEME project; if omitted, one is created automatically. Poll `GET /api/v2/jobs/{activityId}` for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Clean, minimal style with SF Pro"}'
```

With optional `projectId`:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Clean, minimal style","projectId":"<theme_project_uuid>"}'
```

---

## Apply Theme

Request body: `PublicApplyThemeRequest`. Response: `PublicApplyThemeResponse`.

Theme and slide deck are specified in the body. When `regenerateSlides` is true, poll `GET /api/v2/jobs/{activityId}` for batch image regeneration status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/apply" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slideDeckId":"<slide_deck_uuid>","themeId":"<theme_uuid>"}'
```

With slide regeneration:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/apply" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slideDeckId":"<slide_deck_uuid>","themeId":"<theme_uuid>","regenerateSlides":true}'
```

---

## Agent behavior

When the user asks to work with themes (list, get, generate, apply), do the following.

### 1. Choose the right endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| List/browse themes, search themes | `/api/v2/themes` | GET |
| Get one theme by ID | `/api/v2/themes/<theme_id>` | GET |
| Create a new theme from a prompt | `/api/v2/themes/generate` | POST |
| Apply a theme to a slide deck | `/api/v2/themes/apply` | POST |

### 2. Build and run the request

- **Auth**: Every request must include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **GET**: Build `curl` with the chosen path and query params (`offset`, `limit`, `search` for list). Run the curl and show the result.
- **POST**: Build a JSON body from the user’s input (prompt, theme ID, slide deck ID, etc.). Use `-X POST`, `-H "Content-Type: application/json"`, and `-d '...'`. Run the curl and show the result.

### 3. After generate or apply (with regeneration)

- **Generate theme**: Response includes `activityId` and `themeId`. Tell the user the theme was started and give `themeId`. Optionally poll `GET $LAYERPROOF_BASE_URL/api/v2/jobs/<activityId>` until `status` is `DONE` or `CANCELED`, then report outcome.
- **Apply theme with `regenerateSlides: true`**: Response may include `activityId`. If present, tell the user regeneration was started and optionally poll `GET .../api/v2/jobs/<activityId>` for status.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block; do not convert to a table.
- If the response contains a URL for an image (e.g. `previewUrl`), show the image and the JSON.
- On error (4xx/5xx), show the response body and status code; suggest fixing missing/invalid API key, IDs, or request body.

### 5. Example flow

**User**: “Generate a theme with prompt: minimal dark mode.”

1. Choose `POST /api/v2/themes/generate`.
2. Build body: `{"prompt":"minimal dark mode"}`.
3. Run: `curl -X POST "$LAYERPROOF_BASE_URL/api/v2/themes/generate" -H "Content-Type: application/json" -H "X-API-KEY: $LAYERPROOF_API_KEY" -d '{"prompt":"minimal dark mode"}'`.
4. Show the JSON response; if it contains `activityId`, mention they can poll `/api/v2/jobs/{activityId}` for status and use `themeId` once done.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
