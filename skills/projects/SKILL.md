# Skill: Projects Management

## Description

Manage Layerproof projects (create, list, fetch, delete).

---

## Requirements

Environment variables:

- `LAYERPROOF_BASE_URL`
- `LAYERPROOF_API_KEY`

All requests must include:

- `X-API-KEY: $LAYERPROOF_API_KEY`

---

## List Projects

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

Example JSON response:

```json
{"data":[{"id":"2b12d232-eca6-4161-96df-9f954fbbb36f","name":"Exit Strategy Overview","description":"...","status":"ACTIVE","project_kind":"SLIDE_DECK","slide_deck_id":"ab2c64a5-0b59-45ec-baef-1d5b68dbf5fe","created_at":"2026-03-09T03:15:51.548959Z","updated_at":"2026-03-09T03:16:17.069810Z","is_public":false,"vote_count":0}],"has_more":false}
```

---

## Get Project

```bash
curl "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

Example JSON response:

```json
{"id":"2b12d232-eca6-4161-96df-9f954fbbb36f","name":"Exit Strategy Overview","description":"...","status":"ACTIVE","project_kind":"SLIDE_DECK","metadata":{"originalPrompt":"Create an exit strategy overview"},"slide_deck_id":"ab2c64a5-0b59-45ec-baef-1d5b68dbf5fe","created_at":"2026-03-09T03:15:51.548959Z","updated_at":"2026-03-09T03:16:17.069810Z","is_public":false,"vote_count":0,"has_voted":false}
```

---

## Create Project

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
{"id":"68f8270f-f36a-4817-8d0b-2c05efcd1c9a","name":"tmp-skill-project","description":"tmp","status":"DRAFT","project_kind":"SLIDE_DECK","metadata":{},"slide_deck_id":"a877d0c0-9514-40cc-8d27-cee3a1e73934","created_at":"2026-03-10T07:07:46.211172Z","updated_at":"2026-03-10T07:07:46.211176Z","is_public":false,"vote_count":0}
```

---

## Delete Project

```bash
curl -X DELETE "$LAYERPROOF_BASE_URL/api/v2/projects/<project_id>" \
  -H "X-API-KEY: $LAYERPROOF_API_KEY"
```

Example JSON response (may be empty body):

```json
null
```

---

## Response format (required)

- (if response contains url to show image) please show image and show json response instead of table
- Always show the **raw JSON response** (verbatim) in a JSON code block.
- If the response contains a URL for an image, **render/show the image** and also show the **JSON response** (do not convert to a table).
