---
name: jobs
description: Public API job status polling (X-API-KEY). Poll async operations (outline, slides, exports, etc.) by activityId. Types follow PublicApiJobController (/api/v2/jobs/{activityId}).
---

# Skill: Job Monitoring

## Description

Poll the status of asynchronous operations. This skill documents the **public API** at `/api/v2/jobs/{activityId}` (PublicApiJobController). The `activityId` is returned when you start async operations (e.g. outline/generate, batch-generate, export PNG/PPTX). Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicApiJobController` response.

```typescript
// --- Get Job Status (GET) ---
type JobStatusResponse = {
  activityId: string;
  status: 'SCHEDULED' | 'RUNNING' | 'PENDING' | 'DONE' | 'CANCELED';
  message: string;
  createdAt?: string | null;   // ISO 8601
  updatedAt?: string | null;   // ISO 8601
  output?: string | null;      // JSON string when DONE
  failureReason?: string | null;
};
```

---

## Get Job Status

Response: `JobStatusResponse`.

Poll until `status` is `DONE` or `CANCELED`. When DONE, use `output` (JSON string) for results; when failed, use `failureReason`.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/jobs/<activity_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Agent behavior

When the user asks to check job status or poll an async operation, do the following.

### 1. Endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Check/poll job status | `/api/v2/jobs/{activityId}` | GET |

### 2. Build and run

- **Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read env vars; if missing, tell the user.
- **Path**: Use the `activityId` returned from a previous async call (outline generate, batch generate, export, etc.). Run curl and show the result.

### 3. After response

- If status is SCHEDULED/RUNNING/PENDING, suggest polling again after a few seconds.
- If DONE, parse `output` if present and summarize the result.
- If CANCELED or DONE with failureReason, report the outcome.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block.
- On 404, job not found or user does not own the associated project.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).

