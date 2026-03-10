---
name: exports
description: Public API export (X-API-KEY). Export project as PNG ZIP or PPTX (async), get status, cancel. Types follow PublicApiExportController (/api/v2/projects/{projectId}/exports).
---

# Skill: Export Slides

## Description

Export presentations as PNG ZIP or PPTX. This skill documents the **public API** at `/api/v2/projects/{projectId}/exports` (PublicApiExportController). Exports are async: POST returns exportId (activityId); poll GET .../exports/{exportId} for status; when COMPLETED, use downloadUrl (presigned, ~1 hour). Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicApiExportController` data classes.

```typescript
// --- Export PNG / PPTX (POST) — 202 ---
type PublicExportStartedResponse = {
  exportId: string;  // UUID, use to poll status
  status?: string;   // default "PENDING"
};

// --- Get Export Status (GET) ---
type PublicExportProgress = { current: number; total: number };
type PublicExportStatusData = {
  exportId: string;
  status: string;
  format: string;
  downloadUrl?: string | null;
  expiresAt?: string | null;
  fileSizeBytes?: number | null;
  progress?: PublicExportProgress | null;
  errorMessage?: string | null;
};
type PublicExportStatusResponse = { data: PublicExportStatusData };

// --- Cancel Export (POST) ---
type PublicCancelExportResponse = {
  exportId: string;
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
| Get export status / download URL | `.../exports/{exportId}` | GET |
| Cancel in-progress export | `.../exports/{exportId}/cancel` | POST |

### 2. Build and run

- **Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read env vars; if missing, tell the user.
- **Path**: Resolve projectId from context; exportId from the 202 response of export PNG/PPTX.
- Run curl and show the result.

### 3. After starting export

- Response contains `exportId`. Tell the user to poll GET .../exports/{exportId} until status is COMPLETED (or FAILED). When COMPLETED, use `data.downloadUrl` (valid ~1 hour).

### 4. Response handling

- Always show raw JSON in a code block.
- If downloadUrl is present, show the URL and mention the file can be downloaded.
- On 410, download URL expired; suggest triggering a new export.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
