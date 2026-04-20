---
name: exports
description: Public API export (X-API-KEY). Export PNG ZIP, PPTX, or video (async), get status, cancel. PublicApiExportController.
---

# Skill: Export Slides

## Description

Export presentations as PNG ZIP, PPTX, or **narrated video** (Pro / credits). This skill documents the **public API** at `/api/v2/projects/{projectId}/exports` (PublicApiExportController). Exports are async: POST returns `export_id` (same id used for job-style polling); poll GET `.../exports/{export_id}` for status; when COMPLETED, use `download_url` (presigned, ~1 hour). Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicApiExportController` data classes.

```typescript
// --- Export PNG / PPTX / Video (POST) — 202 ---
type PublicExportStartedResponse = {
  export_id: string;  // UUID — poll GET .../exports/{export_id}
  status?: string;   // default "PENDING"
};

// --- Video export body (POST .../exports/video) — optional ---
type PublicVideoExportRequest = {
  quality?: string | null;       // export quality enum as returned by API
  include_notes?: boolean;      // default false
  voice?: string | null;         // default "Puck"
  language_code?: string | null; // default "en-US"
};

// --- Get Export Status (GET) ---
type PublicExportProgress = { current: number; total: number };
type PublicExportStatusData = {
  export_id: string;
  status: string;
  format: string;
  download_url?: string | null;
  expires_at?: string | null;
  file_size_bytes?: number | null;
  progress?: PublicExportProgress | null;
  error_message?: string | null;
};
type PublicExportStatusResponse = { data: PublicExportStatusData };

// --- Cancel Export (POST) ---
type PublicCancelExportResponse = {
  export_id: string;
  status: string;  // "CANCELLED" or "ALREADY_TERMINAL"
};
```

---

## Export PNG ZIP (async)

Response (202): `PublicExportStartedResponse`. Poll GET .../exports/{exportId} for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/exports/png" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Export PPTX (async)

Response (202): `PublicExportStartedResponse`. Poll GET .../exports/{exportId} for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/exports/pptx" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Export Video (async)

Narrated video with TTS. May return **402** (quota) or **403** (Pro plan required). Optional JSON body: `PublicVideoExportRequest`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/exports/video" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"include_notes":false,"voice":"Puck","language_code":"en-US"}'
```

---

## Get Export Status

Response: `PublicExportStatusResponse`. When status is COMPLETED, `data.downloadUrl` and `data.expiresAt` are set.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/exports/<export_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Cancel Export

Response: `PublicCancelExportResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/exports/<export_id>/cancel" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Agent behavior

When the user asks to export a project (PNG ZIP or PPTX) or check/cancel an export, do the following.

### 1. Choose the right endpoint

Base path: `/api/v2/projects/{projectId}/exports`. All require `projectId`.

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Export as PNG ZIP | `.../exports/png` | POST |
| Export as PPTX | `.../exports/pptx` | POST |
| Export as video (TTS) | `.../exports/video` | POST |
| Get export status / download URL | `.../exports/{exportId}` | GET |
| Cancel in-progress export | `.../exports/{exportId}/cancel` | POST |

### 2. Build and run

**Step 1 — Check environment variables first.** Before running any curl command, verify both `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` are set on the user's machine:

```bash
if [[ -z "${LAYERPROOF_BASE_URL}" ]]; then
  echo "ERROR: LAYERPROOF_BASE_URL is not set."
  return 1
fi
if [[ -z "${LAYERPROOF_API_KEY}" ]]; then
  echo "ERROR: LAYERPROOF_API_KEY is not set."
  return 1
fi
```

If running from a project directory with a `.env.local` file, load it first:
```bash
if [[ -f .env.local ]]; then
  set -a
  source .env.local
  set +a
fi
```

**Step 2 — Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read env vars; if missing, tell the user.
**Step 3 — Path**: Resolve projectId from context; exportId from the 202 response of export PNG/PPTX.
**Step 4 — Run**: Run curl and show the result.

### 3. After starting export

- Response contains `export_id`. Tell the user to poll GET .../exports/{export_id} until status is COMPLETED (or FAILED). When COMPLETED, use `data.download_url` (valid ~1 hour).

### 4. Response handling

- Always show raw JSON in a code block.
- If `download_url` is present, show the URL and mention the file can be downloaded.
- On 410, download URL expired; suggest triggering a new export.

### 5. Example workflows

**Workflow A — User**: "Export my project as PPTX and give me the download link."

1. Resolve projectId. POST `.../projects/{projectId}/exports/pptx`; capture `export_id` from 202 response.
2. Poll GET `.../projects/{projectId}/exports/{export_id}` until `data.status` is COMPLETED or FAILED. If FAILED, show `data.error_message` and suggest retry or checking project/slides.
3. When COMPLETED, show `data.download_url` and `data.expires_at`; remind user the URL is temporary (~1 hour).

**Workflow B — User**: "Export the same project as both PNG ZIP and PPTX; if one fails, still get the other."

1. POST `.../exports/png`; get `export_id_png`. POST `.../exports/pptx`; get `export_id_pptx`.
2. Poll both: GET `.../exports/{export_id_png}` and GET `.../exports/{export_id_pptx}` (e.g. in sequence or inform user to poll). For each: when COMPLETED, show `download_url`; when FAILED, show `error_message` and which format failed.
3. Optionally: if user wants to cancel the slower one, POST `.../exports/{export_id}/cancel` for the relevant export id.

**Workflow C — User**: "Start a PPTX export; I might cancel it if it takes too long."

1. POST `.../exports/pptx`; get `export_id`. Tell user polling has started.
2. If user says "cancel the export" before COMPLETED: POST `.../exports/{export_id}/cancel`; show response (status CANCELLED or ALREADY_TERMINAL).
3. If not canceled: poll until COMPLETED and show `download_url`, or FAILED and show `error_message`.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
