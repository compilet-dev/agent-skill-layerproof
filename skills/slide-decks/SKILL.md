---
name: slide-decks
description: Public API slide deck operations (X-API-KEY). Generate outlines, update outline, get deck, batch generate slides, generate transcript/image/content, cancel, theme, transcript, duplicate section. Types follow PublicApiSlideDeckController (/api/v2/projects/{projectId}/slide-deck/{slideDeckId}).
---

# Skill: Slide Deck Generation

## Description

Manage AI-generated slide decks and outlines. This skill documents the **public API** at `/api/v2/projects/{projectId}/slide-deck/{slideDeckId}` (PublicApiSlideDeckController). All endpoints require `projectId` and `slideDeckId` in the path. Authenticate with `X-API-KEY` header. Poll async operations via `GET /api/v2/jobs/{activityId}`.

---

## TypeScript types (request / response)

Mirrors `PublicApiSlideDeckController` data classes.

```typescript
// --- Outline section (used in UpdateOutlineRequest and responses) ---
type PublicApiOutlineSection = {
  id: string;
  sectionTitle: string;
  content?: string | null;
  keyPoints?: string[] | null;
  visualSuggestion?: string | null;
  speakerNotes?: string | null;
};

// --- Generate Outline (POST) — async ---
type GenerateOutlineRequest = {
  prompt: string;           // 3–2000 chars, required
  slideCount?: number;     // default 5, min 1
  language?: string;       // 2–10 chars, e.g. "en"
  fileS3Keys?: string[];   // from POST /api/v2/files/prepare
  webSearchEnabled?: boolean;
};
type GenerateOutlineResponse = {
  activityId: string;
  status: string;
  message: string;
  estimatedCompletionSeconds?: number;
};

// --- Update Outline (PUT) ---
type UpdateOutlineRequest = {
  title: string;
  sections: PublicApiOutlineSection[];
};
type PublicApiOutline = {
  id: string;
  title: string;
  sections: PublicApiOutlineSection[];
  totalSections: number;
  updatedAt?: string;
};
type UpdateOutlineResponse = { outline: PublicApiOutline };

// --- Get Deck (GET) ---
type PublicApiSlideDeck = {
  id: string;
  projectId: string;
  title: string;
  aspectRatio: string;
  theme?: string | null;
  outlineGenerationLiveObjectId?: string | null;
  createdAt: string;
  updatedAt: string;
};
type PublicApiSlide = {
  id: string;
  index: number;
  sectionId: string;
  sectionTitle: string;
  transcript?: string | null;
  imageUrl?: string | null;
  imageExpiresAt?: string | null;
  generationStatus: string;
  createdAt: string;
  updatedAt: string;
};
type PublicApiDeckMetadata = {
  totalSlides: number;
  completedSlides: number;
  pendingSlides: number;
  overallProgress: number;
};
type GetDeckResponse = {
  slideDeck: PublicApiSlideDeck;
  outline: PublicApiOutline;
  slides: PublicApiSlide[];
  metadata: PublicApiDeckMetadata;
};

// --- Batch Generate Slides (POST) — async ---
type BatchGenerateSlidesRequest = {
  generationType?: string;   // default "BOTH"
  speakingStyle?: string | null;
  targetDurationMinutes?: number | null;
  transcriptTone?: string | null;
  aspectRatio?: string | null;
  outputLanguage?: string | null;
  textDetailLevel?: string | null;
  tone?: string | null;
};
type BatchGenerateSlidesResponse = {
  activityId: string;
  totalSlides: number;
  status: string;
  message: string;
  estimatedCompletionSeconds: number;
};

// --- Generate Slide Content / Transcript / Image (POST) — async ---
type GenerateSlideContentRequest = {
  slideSectionId: string;
  speakingStyle?: string | null;
  targetDurationMinutes?: number | null;
  generationType?: string | null;
};
type GenerateSlideContentResponse = {
  activityId: string;
  slideSectionId: string;
  transcriptGenLiveObjectId?: string | null;
  imageGenLiveObjectId?: string | null;
  status: string;
  message: string;
};

type GenerateSlideTranscriptRequest = {
  slideSectionId: string;
  speakingStyle?: string | null;
  targetDurationMinutes?: number | null;
  transcriptTone?: string | null;
};
type GenerateSlideTranscriptResponse = {
  activityId: string;
  slideSectionId: string;
  transcriptGenLiveObjectId: string;
  status: string;
  message: string;
  estimatedCompletionSeconds?: number;
};

type GenerateSlideImageRequest = {
  slideSectionId: string;
  aspectRatio?: string;  // "16:9" | "4:3" | "1:1", default "16:9"
};
type GenerateSlideImageResponse = {
  activityId: string;
  slideSectionId: string;
  imageGenLiveObjectId: string;
  status: string;
  message: string;
  estimatedCompletionSeconds?: number;
};

// --- Cancel Generation (POST) ---
type CancelGenerationRequest = { liveObjectIds?: string[] | null };
type CancelGenerationResponse = {
  cancelledLiveObjects: string[];
  failedToCancelLiveObjects: string[];
  totalActivitiesCancelled: number;
  message: string;
};

// --- Generate Theme (POST) — async ---
type GenerateThemeRequest = {
  prompt: string;              // 10–2000 chars
  referenceImagePaths?: string[] | null;
};
type GenerateThemeResponse = {
  activityId: string;
  status: string;
  message: string;
  estimatedCompletionSeconds: number;
};

// --- Update Transcript (PUT) ---
type UpdateTranscriptRequest = { transcript: string };
type UpdateTranscriptResponse = {
  slideSectionId: string;
  versionId: string;
  versionNumber: number;
  transcriptGenLiveObjectId: string;
  transcript: string;
  updatedAt: string;
};

// --- Duplicate Section (POST) ---
type DuplicateSectionRequest = {
  copyContent?: boolean;
  insertAfter?: boolean;
};
type DuplicateSectionResponse = {
  newSectionId: string;
  newSection: PublicApiOutlineSection;
  transcriptGenLiveObjectId?: string | null;
  imageGenLiveObjectId?: string | null;
  message: string;
};
```

---

## Generate Outline (async)

Request body: `GenerateOutlineRequest`. Response (202): `GenerateOutlineResponse`.

Starts outline generation from a prompt. Use `fileS3Keys` from files uploaded via `/api/v2/files/prepare` and confirm. Poll `GET /api/v2/jobs/{activityId}` for status.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/outline/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Create a product launch deck","slideCount":6}'
```

With reference files and language:

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/outline/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Q4 strategy","slideCount":5,"language":"en","fileS3Keys":["public-api/.../file.pdf"],"webSearchEnabled":true}'
```

---

## Update Outline

Request body: `UpdateOutlineRequest`. Response: `UpdateOutlineResponse`.

Updates outline title and sections (add, remove, reorder). At least one section required.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/outline" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"title":"My Deck","sections":[{"id":"section-uuid","sectionTitle":"Intro","content":"...","speakerNotes":"..."}]}'
```

---

## Get Full Slide Deck

Query: `includeImages` (default true), `imageExpirySeconds` (default 3600). Response: `GetDeckResponse`.

Returns deck, outline, slides with presigned image URLs, and metadata.

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/?includeImages=true&imageExpirySeconds=3600" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Batch Generate Slides (async)

Request body: `BatchGenerateSlidesRequest`. Response (202): `BatchGenerateSlidesResponse`.

Generate transcript and/or images for all outline sections. Requires an outline first. Poll jobs with `activityId`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/batch-generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"generationType":"BOTH"}'
```

---

## Generate Slide Content (async)

Request body: `GenerateSlideContentRequest`. Response (202): `GenerateSlideContentResponse`.

Generate transcript and image for one section. Poll `GET /api/v2/jobs/{activityId}`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/generate-content" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slideSectionId":"<section_uuid>"}'
```

---

## Generate Slide Transcript (async)

Request body: `GenerateSlideTranscriptRequest`. Response (202): `GenerateSlideTranscriptResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/generate-transcript" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slideSectionId":"<section_uuid>"}'
```

---

## Generate Slide Image (async)

Request body: `GenerateSlideImageRequest`. Response (202): `GenerateSlideImageResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/generate-image" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"slideSectionId":"<section_uuid>","aspectRatio":"16:9"}'
```

---

## Cancel Generation

Request body: `CancelGenerationRequest`. Response: `CancelGenerationResponse`.

Provide `liveObjectIds` from active generation responses to cancel.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/cancel" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"liveObjectIds":["<live_object_uuid>"]}'
```

---

## Generate Deck Theme (async)

Request body: `GenerateThemeRequest`. Response (202): `GenerateThemeResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/theme/generate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"prompt":"Minimal corporate blue"}'
```

---

## Update Slide Transcript

Request body: `UpdateTranscriptRequest`. Response: `UpdateTranscriptResponse`.

Path includes `slideSectionId`: `PUT .../slides/{slideSectionId}/transcript`.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/<slide_section_id>/transcript" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"transcript":"Updated speaker notes."}'
```

---

## Duplicate Slide Section

Request body: `DuplicateSectionRequest`. Response: `DuplicateSectionResponse`.

Path includes `slideSectionId`: `POST .../slides/{slideSectionId}/duplicate`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/slide-deck/<slide_deck_id>/slides/<slide_section_id>/duplicate" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"copyContent":true,"insertAfter":true}'
```

---

## Agent behavior

When the user asks to work with slide decks (outline, deck, slides, theme, transcript), do the following.

### 1. Choose the right endpoint

Base path: `/api/v2/projects/{projectId}/slide-deck/{slideDeckId}`. All require `projectId` and `slideDeckId`.

| User intent | Endpoint | Method |
|-------------|----------|--------|
| Generate outline from prompt | `.../outline/generate` | POST |
| Update outline structure | `.../outline` | PUT |
| Get full deck (outline + slides + URLs) | `.../` (with optional query) | GET |
| Batch generate all slides | `.../slides/batch-generate` | POST |
| Generate one slide content | `.../slides/generate-content` | POST |
| Generate one slide transcript | `.../slides/generate-transcript` | POST |
| Generate one slide image | `.../slides/generate-image` | POST |
| Cancel generations | `.../cancel` | POST |
| Generate deck theme | `.../theme/generate` | POST |
| Update slide transcript | `.../slides/{slideSectionId}/transcript` | PUT |
| Duplicate section | `.../slides/{slideSectionId}/duplicate` | POST |

Other endpoints (improve section, generate-manual, visual style, transcript versions, restore version, mark read, TTS, audio download URL, update settings, import prepare/pptx, tone settings, batch layout): use same base path and request/response shapes from controller.

### 2. Build and run the request

- **Auth**: Every request must include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **Path**: Resolve `projectId` and `slideDeckId` (and `slideSectionId` where needed) from context or user input. If missing, ask.
- **GET**: Build curl with path and query params (`includeImages`, `imageExpirySeconds` for get deck). Run and show result.
- **POST/PUT**: Build JSON body from the types above. Use `-X POST` or `-X PUT`, `-H "Content-Type: application/json"`, and `-d '...'`. Run and show result.

### 3. After async endpoints

- Responses include `activityId`. Tell the user the job was started and suggest polling `GET $LAYERPROOF_BASE_URL/api/v2/jobs/{activityId}` until `status` is `DONE` or `CANCELED`.
- Typical flow: generate outline → poll until DONE → get deck or update outline → batch generate slides or generate single slide content/transcript/image → poll jobs.

### 4. Response handling

- Always show the **raw JSON response** in a JSON code block; do not convert to a table.
- If the response contains image URLs (e.g. in get deck `slides[].imageUrl`), show images and the JSON.
- On error (4xx/5xx), show the response body and status code; suggest fixing API key, projectId/slideDeckId/sectionId, or request body.

### 5. Example flow

**User**: “Generate an outline for a product launch deck in project X.”

1. Resolve projectId and slideDeckId (e.g. from GET projects, then project.slideDeckId).
2. Choose `POST .../outline/generate`.
3. Build body: `{"prompt":"Product launch deck","slideCount":5}`.
4. Run curl; show JSON. Tell user to poll `GET /api/v2/jobs/{activityId}` and then call get deck or update outline as needed.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).

Example JSON responses:

```json
{"activityId":"job-uuid","status":"PENDING","message":"Outline generation started. Poll /api/v2/jobs/... for status.","estimatedCompletionSeconds":30}
```

```json
{"outline":{"id":"outline-uuid","title":"My Deck","sections":[{"id":"sec-1","sectionTitle":"Intro","content":"...","keyPoints":[],"visualSuggestion":null,"speakerNotes":"..."}],"totalSections":1,"updatedAt":"2026-03-10T08:00:00Z"}}
```

```json
{"slideDeck":{"id":"deck-uuid","projectId":"proj-uuid","title":"My Deck","aspectRatio":"16:9","theme":null,"outlineGenerationLiveObjectId":"...","createdAt":"2026-03-10T08:00:00Z","updatedAt":"2026-03-10T08:00:00Z"},"outline":{...},"slides":[{"id":"slide-uuid","index":0,"sectionId":"sec-1","sectionTitle":"Intro","transcript":"...","imageUrl":"https://...","imageExpiresAt":"...","generationStatus":"COMPLETED","createdAt":"...","updatedAt":"..."}],"metadata":{"totalSlides":1,"completedSlides":1,"pendingSlides":0,"overallProgress":100}}
```

```json
{"activityId":"job-uuid","totalSlides":5,"status":"PENDING","message":"Batch generation started for 5 slides","estimatedCompletionSeconds":60}
```
