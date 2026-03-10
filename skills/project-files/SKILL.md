---
name: project-files
description: Public API project file management (X-API-KEY). Prepare upload/update, confirm, get file, download URL, delete. Types follow PublicApiProjectFileController (/api/v2/projects/{projectId}/...).
---

# Skill: Project Files

## Description

Manage files in project directories. This skill documents the **public API** at `/api/v2/projects/{projectId}` (PublicApiProjectFileController). Upload flow: 1) POST prepare with directoryId; 2) PUT file to uploadUrl; 3) POST confirm with fileId. All paths require `projectId`; prepare/prepare-update also need `directoryId`. Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicApiProjectFileController` data classes.

```typescript
// --- Prepare upload (POST) — 201 ---
type PrepareFileUploadRequest = {
  path: string;
  fileName: string;
  mimeType: string;
  size: number;
};
type PrepareFileUploadResponse = {
  uploadUrl: string;
  fileId: string;
  s3Key: string;
  expiresAt: string;
};

// --- Prepare update (POST) ---
type PrepareUpdateFileRequest = {
  path: string;
  mimeType?: string | null;
  size?: number | null;
};
type PrepareUpdateFileResponse = {
  uploadUrl: string;
  fileId: string;
  s3Key: string;
  expiresAt: string;
};

// --- Confirm (POST) ---
type ConfirmFileUploadRequest = { metadata?: Record<string, unknown> | null };
type FileResponse = {
  id: string;
  name: string;
  fileType: string;
  mimeType: string;
  s3Key: string;
  size: number;
  uploadedAt: string;
  uploadedBy: string;
  status: string;
  metadata: Record<string, unknown> | null;
};

// --- Download URL (GET), Get file (GET), Delete (DELETE) ---
type DownloadUrlResponse = {
  downloadUrl: string;
  expiresAt: string;
};
```

---

## Prepare Project File Upload

Request body: `PrepareFileUploadRequest`. Response (201): `PrepareFileUploadResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/directories/<directory_id>/files/prepare" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"path":"/","fileName":"doc.pdf","mimeType":"application/pdf","size":2048}'
```

---

## Prepare Project File Update

Request body: `PrepareUpdateFileRequest`. Response: `PrepareUpdateFileResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/directories/<directory_id>/files/prepare-update" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"path":"/doc.pdf","mimeType":"application/pdf","size":2048}'
```

---

## Confirm Project File Upload

Request body: `ConfirmFileUploadRequest`. Response: `FileResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/files/<file_id>/confirm" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"metadata":{}}'
```

---

## Get File Download URL

Response: `DownloadUrlResponse`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/files/<file_id>/download-url" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Get File Details

Response: `FileResponse`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/files/<file_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Delete Project File

Response: 204 No Content.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/files/<file_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Agent behavior

When the user asks to manage project files (upload, update, confirm, download, delete), do the following.

### 1. Choose the right endpoint

Base path: `/api/v2/projects/{projectId}`. Replace `project_id`, `directory_id`, `file_id` as needed.

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Get upload URL for new file | `.../directories/{directoryId}/files/prepare` | POST |
| Get upload URL to update file | `.../directories/{directoryId}/files/prepare-update` | POST |
| Confirm upload after PUT | `.../files/{fileId}/confirm` | POST |
| Get download URL | `.../files/{fileId}/download-url` | GET |
| Get file metadata | `.../files/{fileId}` | GET |
| Delete file | `.../files/{fileId}` | DELETE |

### 2. Build and run the request

- **Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read env vars; if missing, tell the user.
- **Path**: Resolve projectId, directoryId, fileId from context or user input.
- **POST**: Build JSON body from types above; run curl and show result.
- **GET/DELETE**: Build path only; run curl and show result.

### 3. Upload flow

Prepare → PUT file to uploadUrl → confirm with fileId. Then use fileId for get/download/delete.

### 4. Response handling

- Always show raw JSON in a code block; show image + JSON if image URL present.
- On error, show body and status code.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).

