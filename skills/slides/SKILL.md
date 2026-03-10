---
name: slides
description: Public API slide editing (X-API-KEY). Edit slide images with AI, accept/revert edits, object removal, text extraction, save Konva nodes. Types follow PublicApiSlideController (/api/v2/projects/{projectId}/slides).
---

# Skill: Slide Editing

## Description

Modify slides after generation. This skill documents the **public API** at `/api/v2/projects/{projectId}/slides` (PublicApiSlideController). All slide endpoints require `projectId` and `slideId` in the path. Authenticate with `X-API-KEY` header.

---

## TypeScript types (request / response)

Mirrors `PublicApiSlideController` (/api/v2/projects/{projectId}/slides) data classes.

```typescript
// Shared: cropped region for region-based editing
type CroppedRegion = {
  x: number;
  y: number;
  width: number;
  height: number;
};

// --- Edit Slide Image (POST) — async, poll jobs ---
type SlideImageEditRequest = {
  /** AI instruction for the edit (required) */
  instruction: string;
  /** Path to input image in project working dir (required) */
  inputImagePath: string;
  /** Optional extra reference image paths */
  otherReferenceImagePaths?: string[];
  /** Optional region to edit; only this region is modified */
  croppedRegion?: CroppedRegion;
};

// --- Accept Image Edit (POST) ---
type AcceptImageEditRequest = {
  liveObjectId: string;   // UUID from image-edit or object-removal response
  targetNodeId?: string;
};

// --- Revert Slide (POST) ---
type RevertSlideRequest = {
  historyEntryId: string;  // UUID from slide history
  nodeId?: string;
};

// --- Object Removal (POST) — async, poll jobs ---
type ObjectRemovalRequest = {
  inputImagePath: string;
  maskPath?: string;
  croppedRegion?: CroppedRegion;
};

// --- Extract Text (POST) — async, poll jobs ---
type ExtractTextRequest = {
  nodeId?: string;
  nodeImagePath?: string;
};

// --- Save Konva Nodes (PUT) ---
type SaveKonvaNodesRequest = {
  konvaNodes: Record<string, unknown>;  // required
  konvaOrder: string[];                  // required
  flattenedImagePath?: string;
  baseSnapshotId?: string;
};

// --- Responses ---
type TriggerWorkflowResponse = {
  activityId: string;   // UUID – poll GET /api/v2/jobs/{activityId}
  workflowType: string;
  liveObjectId: string; // UUID – pass to accept-image-edit when done
};

type AcceptImageEditResponse = {
  id: string;
  slideSectionId: string;
  imagePath: string | null;
  status: string;
};
```

---

## Edit Slide Image (async)

Request body: `SlideImageEditRequest`. Response: `TriggerWorkflowResponse`.

Triggers AI-powered slide image editing. Upload input image first, then call with `instruction` and `inputImagePath`. Poll `GET /api/v2/jobs/{activityId}` for status; when DONE, call accept-image-edit with `liveObjectId`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/image-edit" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"instruction":"Make the background blue","inputImagePath":"/slides/slide-1.png"}'
```

With optional region and reference images:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/image-edit" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"instruction":"Replace this area","inputImagePath":"/slides/slide-1.png","croppedRegion":{"x":0,"y":0,"width":200,"height":100},"otherReferenceImagePaths":["/ref.png"]}'
```

---

## Accept Image Edit

Request body: `AcceptImageEditRequest`. Response: `AcceptImageEditResponse`.

Call after the image-edit or object-removal workflow completes successfully. Use `liveObjectId` from the trigger response.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/accept-image-edit" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"liveObjectId":"<live_object_uuid>"}'
```

---

## Revert Slide

Request body: `RevertSlideRequest`. Response: `AcceptImageEditResponse`.

Reverts the slide to a previous version using `historyEntryId` from slide history.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/revert" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"historyEntryId":"<history_entry_uuid>"}'
```

---

## Remove Objects (async)

Request body: `ObjectRemovalRequest`. Response: `TriggerWorkflowResponse`.

Triggers AI-powered object removal. Optional `maskPath` for mask-based removal. Poll jobs; when DONE, call accept-image-edit with `liveObjectId`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/object-removal" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"inputImagePath":"/slides/slide-1.png"}'
```

With mask:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/object-removal" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"inputImagePath":"/slides/slide-1.png","maskPath":"/masks/mask.png"}'
```

---

## Extract Text (async)

Request body: `ExtractTextRequest` (optional). Response: `TriggerWorkflowResponse`.

Performs OCR, removes text from image via inpainting, creates editable Konva text nodes. Poll `GET /api/v2/jobs/{activityId}` for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/extract-text" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{}'
```

With optional node:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/extract-text" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"nodeId":"node-1","nodeImagePath":"/nodes/node1.png"}'
```

---

## Save Konva Nodes

Request body: `SaveKonvaNodesRequest`. Response: 200, no body.

Persists Konva canvas nodes and order (positions, layer order, flattened image path).

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slides/<slide_id>/konva-nodes" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"konvaNodes":{},"konvaOrder":[]}'
```

---

## Agent behavior

When the user asks to edit slides (image edit, accept edit, revert, remove objects, extract text, save canvas), do the following.

### 1. Choose the right endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Edit slide image with AI instruction | `/api/v2/projects/{projectId}/slides/{slideId}/image-edit` | POST |
| Accept edited image after workflow completes | `/api/v2/projects/{projectId}/slides/{slideId}/accept-image-edit` | POST |
| Revert slide to previous version | `/api/v2/projects/{projectId}/slides/{slideId}/revert` | POST |
| Remove objects from slide image (AI) | `/api/v2/projects/{projectId}/slides/{slideId}/object-removal` | POST |
| Extract text from slide (OCR + inpainting) | `/api/v2/projects/{projectId}/slides/{slideId}/extract-text` | POST |
| Save Konva canvas nodes/order | `/api/v2/projects/{projectId}/slides/{slideId}/konva-nodes` | PUT |

All paths require `projectId` and `slideId` (replace placeholders with actual UUIDs).

### 2. Build and run the request

- **Auth**: Every request must include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **Path**: Resolve `project_id` and `slide_id` from context (e.g. from project/slide-deck list or user input). If missing, ask the user.
- **POST/PUT**: Build JSON body from the types above. Use `-X POST` or `-X PUT`, `-H "Content-Type: application/json"`, and `-d '...'`. Run the curl and show the result.

### 3. After async endpoints (image-edit, object-removal, extract-text)

- Response includes `activityId` and `liveObjectId`. Tell the user the workflow was started.
- Suggest polling `GET $LAYERPROOF_BASE_URL/api/v2/jobs/<activityId>` until `status` is `DONE` or `CANCELED`.
- For image-edit and object-removal: when DONE, tell the user to call **accept-image-edit** with `liveObjectId` to apply the result.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block; do not convert to a table.
- If the response contains a URL for an image (e.g. `imagePath`), show the image and the JSON.
- On error (4xx/5xx), show the response body and status code; suggest fixing API key, projectId/slideId, or request body.

### 5. Example flow

**User**: “Edit slide image in project X, slide Y: make the background darker.”

1. Choose `POST /api/v2/projects/{projectId}/slides/{slideId}/image-edit`.
2. Resolve projectId and slideId (from user or ask).
3. Build body: `{"instruction":"Make the background darker","inputImagePath":"/path/to/slide/image"}` (user may need to provide image path).
4. Run curl; show JSON. Mention polling jobs with `activityId` and calling accept-image-edit with `liveObjectId` when done.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
