---
name: projects
description: Public API project management (X-API-KEY). Create, list, get, update, delete, clone, vote projects. Supports cursor pagination. Types follow PublicApiProjectController (/api/v2/projects).
---

# Skill: Projects Management

## Description

Manage Layerproof projects (create, list, get, update, delete, clone, vote). This skill documents the **public API** at `/api/v2/projects` (PublicApiProjectController). Authenticate with `X-API-KEY` header. List uses cursor pagination via `limit` and `cursor` query params.

---

## Requirements

Environment variables:

- `LAYERPROOF_BASE_URL`
- `LAYERPROOF_API_KEY`

All requests must include:

- `X-API-KEY: $LAYERPROOF_API_KEY`

---

## TypeScript types (request / response)

Mirrors `PublicApiProjectController` data classes.

```typescript
// --- Create (POST) — 201 ---
type PublicApiCreateProjectRequest = {
  name: string;                   // required, 1–255 chars
  description?: string | null;    // max 10000 chars
  metadata?: Record<string, any> | null;
  projectKind?: "SLIDE_DECK" | "GENERIC" | "MINDMAP" | "THEME" | "BLOG_POST" | "SOCIAL_CAMPAIGN";  // default: SLIDE_DECK
};

// --- Update (PUT) ---
type PublicApiUpdateProjectRequest = {
  name?: string | null;           // 1–255 chars
  description?: string | null;    // max 10000 chars
  metadata?: Record<string, any> | null;
  isPublic?: boolean | null;
};

// --- Clone (POST /{projectId}/clone) — 201 ---
type CloneProjectRequest = {
  name?: string | null;           // defaults to "Copy of <source name>"
  workspaceId?: string | null;    // UUID; if null, auto-creates hidden workspace
};

// --- Project response ---
type PublicApiProjectResponse = {
  id: string;
  name: string;
  description: string | null;
  status: string;                 // e.g. "ACTIVE", "DRAFT"
  projectKind: string;            // e.g. "SLIDE_DECK"
  metadata: Record<string, any>;
  slideDeckId: string | null;
  createdAt: string;              // ISO 8601
  updatedAt: string;              // ISO 8601
  isPublic: boolean;
  voteCount: number;
  hasVoted?: boolean | null;
};

// --- List response (cursor-paginated) ---
type PublicApiProjectListResponse = {
  data: PublicApiProjectResponse[];
  nextCursor: string | null;      // pass as `cursor` param for next page
  hasMore: boolean;
};
```

---

## List Projects

Query params: `limit` (default 20, max 100), `cursor` (from previous `nextCursor`).

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects?limit=20" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

Example JSON response:

```json
{"data":[{"id":"2b12d232-eca6-4161-96df-9f954fbbb36f","name":"Exit Strategy Overview","description":"...","status":"ACTIVE","projectKind":"SLIDE_DECK","metadata":{},"slideDeckId":"ab2c64a5-0b59-45ec-baef-1d5b68dbf5fe","createdAt":"2026-03-09T03:15:51.548959Z","updatedAt":"2026-03-09T03:16:17.069810Z","isPublic":false,"voteCount":0,"hasVoted":null}],"nextCursor":null,"hasMore":false}
```

---

## Get Project

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

Example JSON response:

```json
{"id":"2b12d232-eca6-4161-96df-9f954fbbb36f","name":"Exit Strategy Overview","description":"...","status":"ACTIVE","projectKind":"SLIDE_DECK","metadata":{"originalPrompt":"Create an exit strategy overview"},"slideDeckId":"ab2c64a5-0b59-45ec-baef-1d5b68dbf5fe","createdAt":"2026-03-09T03:15:51.548959Z","updatedAt":"2026-03-09T03:16:17.069810Z","isPublic":false,"voteCount":0,"hasVoted":false}
```

---

## Create Project

Request body: `PublicApiCreateProjectRequest`. Response (201): `PublicApiProjectResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{
    "name": "<project_name>",
    "description": "<description>",
    "projectKind": "SLIDE_DECK"
  }'
```

Example JSON response:

```json
{"id":"68f8270f-f36a-4817-8d0b-2c05efcd1c9a","name":"tmp-skill-project","description":"tmp","status":"DRAFT","projectKind":"SLIDE_DECK","metadata":{},"slideDeckId":"a877d0c0-9514-40cc-8d27-cee3a1e73934","createdAt":"2026-03-10T07:07:46.211172Z","updatedAt":"2026-03-10T07:07:46.211176Z","isPublic":false,"voteCount":0,"hasVoted":null}
```

---

## Update Project

Request body: `PublicApiUpdateProjectRequest`. Only provided fields are updated. Response: `PublicApiProjectResponse`.

```bash
curl -X PUT "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"<new_name>","description":"<new_description>","isPublic":true}'
```

---

## Delete Project

Soft-deletes the project and all associated resources. Response: 204 No Content.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## List Public Projects

Query params: `limit` (default 20, max 100), `cursor`, `slide_deck_type` (one of `PRESENTATION`, `SOCIAL_POST`, `WEBPAGE`, `VIDEO`; optional).

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/public?limit=20" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Vote for a Project

Records a vote on a public project. Response: 204 No Content.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/vote" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Remove Vote from a Project

Removes a vote from a public project. Response: 204 No Content.

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/vote" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

---

## Clone Project

Clones an owned or public project into a new private project. Request body: `CloneProjectRequest`. Response (201): `PublicApiProjectResponse`.

```bash
curl -X POST "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>/clone" \
  -H "Content-Type: application/json" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY" \
  -d '{"name":"<cloned_project_name>","workspaceId":"<workspace_id>"}'
```

---

## Agent behavior

When the user asks to manage projects, do the following.

### 1. Choose the right endpoint

| User intent | Endpoint | Method |
|-------------|----------|--------|
| List own projects | `/api/v2/projects?limit=20` | GET |
| Get project by ID | `/api/v2/projects/{projectId}` | GET |
| Create project | `/api/v2/projects` | POST |
| Update project | `/api/v2/projects/{projectId}` | PUT |
| Delete project | `/api/v2/projects/{projectId}` | DELETE |
| List public projects | `/api/v2/projects/public?limit=20` | GET |
| Vote for a project | `/api/v2/projects/{projectId}/vote` | POST |
| Remove vote | `/api/v2/projects/{projectId}/vote` | DELETE |
| Clone a project | `/api/v2/projects/{projectId}/clone` | POST |

### 2. Build and run the request

- **Auth**: Include `X-API-KEY: $LAYERPROOF_API_KEY`. Read `LAYERPROOF_BASE_URL` and `LAYERPROOF_API_KEY` from the environment; if missing, tell the user to set them.
- **GET list**: Use `limit` (default 20, max 100) and `cursor` for pagination. Pass `nextCursor` from a previous response as `cursor` to fetch the next page.
- **POST/PUT**: Build JSON body from the appropriate request type. Run curl and show result.
- **DELETE project / vote**: Build path; run curl. Response is 204 with no body.

### 3. Response handling

- Always show the **raw JSON response** in a JSON code block.
- For 204 responses, indicate success and no body.
- On error, show the response body and status code.

### 4. Example flow

**User**: "Create a project called Q3 Roadmap."

1. Choose POST `/api/v2/projects`.
2. Body: `{"name":"Q3 Roadmap","projectKind":"SLIDE_DECK"}`.
3. Run curl; show JSON. The returned `id` is the project ID; `slideDeckId` is the associated slide deck.

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
