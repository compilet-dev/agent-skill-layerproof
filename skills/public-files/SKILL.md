---
name: public-files
description: Public API reference files (X-API-KEY). Prepare upload, confirm, delete, get download URL. Use s3_key from prepare in outline generation. Types follow PublicApiFileController (/api/v2/files).
---

# Skill: Public Files

## Description

Upload reference files (images, documents) for use in outline generation. This skill documents the **public API** at `/api/v2/files` (PublicApiFileController). Flow: 1) POST /prepare → get `upload_url` and `s3_key`; 2) PUT file to `upload_url`; 3) POST /confirm; use `s3_key` in `file_s3_keys` when calling slide-deck outline/generate. Authenticate with `X-API-KEY` header. Max 20 MB; allowed types: PDF, DOCX, PPTX, TXT, MD, PNG, JPEG, WebP, SVG, BMP, TIFF, HEIC, HEIF, AVIF, ICO.

---

## TypeScript types (request / response)

Mirrors `PublicApiFileController` (/api/v2/files) data classes.

```typescript
// --- Prepare (POST) ---
type PrepareFileUploadRequest = {
  file_name: string;
  mime_type: string;
  size: number;  // 1 to 20*1024*1024
};
type PrepareFileUploadResponse = {
  upload_url: string;
  s3_key: string;
  expires_at: string;  // ISO 8601, ~10 min
};

// --- Confirm (POST) ---
type PublicApiConfirmFileUploadRequest = { s3_key: string };
type PublicApiConfirmFileUploadResponse = { s3_key: string; uploaded: boolean };

// --- Delete (POST) ---
type PublicApiDeleteFileRequest = { s3_key: string };
type PublicApiDeleteFileResponse = { success: boolean };

// --- Download URL (POST) ---
type PublicApiDownloadUrlRequest = {
  s3_key: string;
  expiry_seconds?: number;  // 60–604800, default 3600
};
type PublicApiDownloadUrlResponse = {
  download_url: string;
  expires_at: string;
};
```

---

## Prepare Public File Upload

Request body: `PrepareFileUploadRequest`. Response: `PrepareFileUploadResponse`. Then PUT file to `upload_url` with matching Content-Type.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/files/prepare" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"file_name":"doc.pdf","mime_type":"application/pdf","size":1024}'
```

---

## Confirm Public File Upload

Request body: `PublicApiConfirmFileUploadRequest`. Response: `PublicApiConfirmFileUploadResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/files/confirm" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"s3_key":"<s3_key_from_prepare>"}'
```

---

## Delete Public File

Request body: `PublicApiDeleteFileRequest`. Response: `PublicApiDeleteFileResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/files/delete" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"s3_key":"<s3_key>"}'
```

---

## Get Public File Download URL

Request body: `PublicApiDownloadUrlRequest`. Response: `PublicApiDownloadUrlResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/files/download-url" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"s3_key":"<s3_key>","expiry_seconds":3600}'
```

---

## Agent behavior

When the user asks to upload or manage reference files (for outlines), do the following.

### 1. Choose the right endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Get upload URL for a file | `/api/v2/files/prepare` | POST |
| Confirm file uploaded to S3 | `/api/v2/files/confirm` | POST |
| Delete a file by s3_key | `/api/v2/files/delete` | POST |
| Get download URL for a file | `/api/v2/files/download-url` | POST |

### 2. Build and run the request

- **Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **Body**: All endpoints use JSON body (`file_name`, `mime_type`, `size` for prepare; `s3_key` for confirm/delete/download-url). Run curl and show the result.

### 3. Upload flow

After prepare: tell the user to PUT the file to `upload_url` with `Content-Type: <mime_type>`, then call confirm with the returned `s3_key`. Use that `s3_key` in `file_s3_keys` when calling slide-deck outline/generate.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block.
- If the response contains a URL for an image, show the image and the JSON.
- On error, show the response body and status code.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
