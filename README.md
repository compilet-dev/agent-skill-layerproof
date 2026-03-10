# Layerproof Agent Skills

A collection of AI agent skills for interacting with the **Layerproof API** (public API, `X-API-KEY` auth). Each skill teaches the agent the correct endpoints, request/response types, curl commands, and behavior for one area of the API.

Works with **Claude Code**, **Cursor**, and any agent that reads SKILL.md files.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Claude Code](#claude-code)
  - [Cursor](#cursor)
- [Available Skills](#available-skills)
- [Typical Workflows](#typical-workflows)
- [Environment Variables](#environment-variables)
- [How Skills Work](#how-skills-work)
- [Skill Reference](#skill-reference)

---

## Prerequisites

You need a **Layerproof API key** and the base URL. Persist these for all future sessions:

```bash
echo 'export LAYERPROOF_BASE_URL=https://api.layerproof.com' >> ~/.zshrc
echo 'export LAYERPROOF_API_KEY=your-api-key-here' >> ~/.zshrc
source ~/.zshrc
```

The agent reads these from your environment automatically. Every API request uses:

```
X-API-KEY: $LAYERPROOF_API_KEY
```

---

## Installation

### Claude Code

1. **Clone this repository** into your project or home directory:

   ```bash
   git clone https://github.com/your-org/agent-skill-layerproof ~/.layerproof-skills
   ```

2. **Register the skill pack** by adding it to your `CLAUDE.md` (project root or `~/.claude/CLAUDE.md`):

   ```markdown
   ## Agent Skills

   Load the following skill files before performing any Layerproof API operations:

   - ~/.layerproof-skills/skills/workspaces/SKILL.md
   - ~/.layerproof-skills/skills/projects/SKILL.md
   - ~/.layerproof-skills/skills/public-files/SKILL.md
   - ~/.layerproof-skills/skills/project-files/SKILL.md
   - ~/.layerproof-skills/skills/slide-decks/SKILL.md
   - ~/.layerproof-skills/skills/slides/SKILL.md
   - ~/.layerproof-skills/skills/themes/SKILL.md
   - ~/.layerproof-skills/skills/exports/SKILL.md
   - ~/.layerproof-skills/skills/jobs/SKILL.md
   ```

3. **Or use the marketplace.json** to install via Claude Code CLI:

   ```bash
   claude skills install ./marketplace.json
   ```

4. **Set environment variables** (persist for future sessions):

   ```bash
   echo 'export LAYERPROOF_BASE_URL=https://api.layerproof.com' >> ~/.zshrc
   echo 'export LAYERPROOF_API_KEY=your-api-key-here' >> ~/.zshrc
   source ~/.zshrc
   ```

5. **Start a session** and ask the agent to use a skill:

   ```
   Create a new workspace called "Q4 Campaign"
   ```

   The agent will read the relevant SKILL.md, build the correct `curl` command, run it, and show you the JSON response.

---

### Cursor

1. **Clone this repository** anywhere accessible:

   ```bash
   git clone https://github.com/your-org/agent-skill-layerproof ~/.layerproof-skills
   ```

2. **Add to `.cursor/settings.json`** or via Cursor Settings → Agent → Skills:

   ```json
   {
     "agent.skillPaths": [
       "~/.layerproof-skills/skills"
     ]
   }
   ```

3. **Or reference individual SKILL.md files** using `@` in the chat:

   ```
   @~/.layerproof-skills/skills/slide-decks/SKILL.md generate an outline for a product launch
   ```

4. **Set environment variables** in your shell profile and restart Cursor.

---

## Available Skills

| Skill | Path | What it does |
|-------|------|--------------|
| **workspaces** | `skills/workspaces/SKILL.md` | Create, list, get, update, delete workspaces |
| **projects** | `skills/projects/SKILL.md` | Create, list, get, update, delete slide deck projects |
| **public-files** | `skills/public-files/SKILL.md` | Upload reference files for use in outline generation (prepare → PUT → confirm) |
| **project-files** | `skills/project-files/SKILL.md` | Manage files inside project directories (prepare, update, confirm, get, delete) |
| **slide-decks** | `skills/slide-decks/SKILL.md` | Generate outlines, batch/single slide content, transcripts, images, themes, audio, import PPTX |
| **slides** | `skills/slides/SKILL.md` | AI image editing, object removal, text extraction, accept edits, revert, save Konva canvas |
| **themes** | `skills/themes/SKILL.md` | List, get, generate, and apply visual themes to slide decks |
| **exports** | `skills/exports/SKILL.md` | Export projects as PNG ZIP or PPTX; poll status; download when complete |
| **jobs** | `skills/jobs/SKILL.md` | Poll any async job by activityId (outline, batch generate, export, theme, etc.) |

---

## Typical Workflows

### 1. Generate a full slide deck from scratch

```
1. Create project         → projects skill     → POST /api/v2/projects
2. Generate outline       → slide-decks skill  → POST .../outline/generate
3. Poll until DONE        → jobs skill         → GET  /api/v2/jobs/{activityId}
4. Batch generate slides  → slide-decks skill  → POST .../slides/batch-generate
5. Poll until DONE        → jobs skill         → GET  /api/v2/jobs/{activityId}
6. Get full deck          → slide-decks skill  → GET  .../   (with imageUrls)
```

### 2. Generate a deck with reference files

```
1. Create project         → projects skill
2. Upload reference docs  → public-files skill → POST /api/v2/files/prepare → PUT → POST /confirm
3. Generate outline       → slide-decks skill  → POST .../outline/generate  (pass s3Keys)
4. Batch generate slides  → slide-decks skill
5. Export as PPTX         → exports skill      → POST .../exports/pptx
6. Poll export & download → exports + jobs     → GET  .../exports/{exportId}
```

### 3. Apply a custom theme

```
1. List themes            → themes skill       → GET  /api/v2/themes
2. Generate new theme     → themes skill       → POST /api/v2/themes/generate
3. Poll theme job         → jobs skill
4. Apply to slide deck    → themes skill       → POST /api/v2/themes/apply
5. (Optionally) batch regenerate slides         regenerateSlides: true + poll job
```

### 4. Edit a slide image

```
1. Get deck slides        → slide-decks skill
2. Edit slide image       → slides skill       → POST .../slides/{slideId}/image-edit
3. Poll until DONE        → jobs skill
4. Accept edit            → slides skill       → POST .../slides/{slideId}/accept-image-edit
```

### 5. Export slides

```
1. Export PNG ZIP         → exports skill      → POST .../exports/png
2. Poll status            → exports skill      → GET  .../exports/{exportId}
3. Download URL           → from response data.downloadUrl (valid 1 hour)
```

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `LAYERPROOF_BASE_URL` | Yes | Base URL, e.g. `https://api.layerproof.com` |
| `LAYERPROOF_API_KEY` | Yes | Your API key; sent as `X-API-KEY` header |

If not set, the agent will ask you to set them before making any calls.

---

## How Skills Work

Each `SKILL.md` file contains:

1. **YAML frontmatter** — machine-readable `name` and `description` for skill discovery.
2. **TypeScript types** — request and response shapes (from the actual Kotlin data classes in `service-poc`), so the agent knows exactly what to send and expect.
3. **Endpoint sections** — correct path, HTTP method, request body, and a `curl` example.
4. **Agent behavior** — a decision table (user intent → endpoint → method) and step-by-step instructions the agent follows to choose the right endpoint, build the request, run the curl, and handle the response.
5. **Response format rules** — the agent always shows raw JSON; shows images if response contains image URLs.
6. **Example JSON responses** — real-shape examples from the actual API.

The agent reads the relevant SKILL.md, executes `curl` commands directly against the API, and shows you the raw JSON response.

---

## Skill Reference

### workspaces — `/api/v2/workspaces`

| Operation | Method | Path |
|-----------|--------|------|
| Create | POST | `/api/v2/workspaces` |
| List | GET | `/api/v2/workspaces?page=0&pageSize=20` |
| Get | GET | `/api/v2/workspaces/{workspaceId}` |
| Update | PUT | `/api/v2/workspaces/{workspaceId}` |
| Delete | DELETE | `/api/v2/workspaces/{workspaceId}` |

### projects — `/api/v2/projects`

| Operation | Method | Path |
|-----------|--------|------|
| Create | POST | `/api/v2/projects` |
| List | GET | `/api/v2/projects` |
| Get | GET | `/api/v2/projects/{projectId}` |
| Update | PUT | `/api/v2/projects/{projectId}` |
| Delete | DELETE | `/api/v2/projects/{projectId}` |

> The response includes `slideDeckId` — use this as `{slideDeckId}` in slide-deck endpoints.

### public-files — `/api/v2/files`

| Operation | Method | Path |
|-----------|--------|------|
| Prepare upload | POST | `/api/v2/files/prepare` |
| Confirm upload | POST | `/api/v2/files/confirm` |
| Delete | POST | `/api/v2/files/delete` |
| Get download URL | POST | `/api/v2/files/download-url` |

> Use the returned `s3Key` in `fileS3Keys` when calling outline/generate.

### project-files — `/api/v2/projects/{projectId}`

| Operation | Method | Path |
|-----------|--------|------|
| Prepare upload | POST | `.../directories/{directoryId}/files/prepare` |
| Prepare update | POST | `.../directories/{directoryId}/files/prepare-update` |
| Confirm | POST | `.../files/{fileId}/confirm` |
| Get | GET | `.../files/{fileId}` |
| Download URL | GET | `.../files/{fileId}/download-url` |
| Delete | DELETE | `.../files/{fileId}` |

### slide-decks — `/api/v2/projects/{projectId}/slide-deck/{slideDeckId}`

| Operation | Method | Path (relative to base) |
|-----------|--------|--------------------------|
| Generate outline (async) | POST | `.../outline/generate` |
| Update outline | PUT | `.../outline` |
| Get full deck | GET | `.../` |
| Batch generate slides (async) | POST | `.../slides/batch-generate` |
| Generate slide content (async) | POST | `.../slides/generate-content` |
| Generate slide transcript (async) | POST | `.../slides/generate-transcript` |
| Generate slide image (async) | POST | `.../slides/generate-image` |
| Cancel generation | POST | `.../cancel` |
| Generate deck theme (async) | POST | `.../theme/generate` |
| Update slide transcript | PUT | `.../slides/{slideSectionId}/transcript` |
| Duplicate section | POST | `.../slides/{slideSectionId}/duplicate` |
| Get tone settings | GET | `.../tone-settings` |
| Update tone settings | PUT | `.../tone-settings` |
| Update deck settings | PATCH | `.../` |
| Import PPTX (prepare) | POST | `.../import/prepare-upload` |
| Import PPTX | POST | `.../import` |
| Batch generate layout (async) | POST | `.../slides/batch-generate-layout` |

### slides — `/api/v2/projects/{projectId}/slides`

| Operation | Method | Path |
|-----------|--------|------|
| Edit slide image (async) | POST | `.../slides/{slideId}/image-edit` |
| Accept image edit | POST | `.../slides/{slideId}/accept-image-edit` |
| Revert slide | POST | `.../slides/{slideId}/revert` |
| Object removal (async) | POST | `.../slides/{slideId}/object-removal` |
| Extract text (async) | POST | `.../slides/{slideId}/extract-text` |
| Save Konva nodes | PUT | `.../slides/{slideId}/konva-nodes` |

> Async endpoints return `activityId` and `liveObjectId`. Poll jobs; when DONE, call accept-image-edit with `liveObjectId`.

### themes — `/api/v2/themes`

| Operation | Method | Path |
|-----------|--------|------|
| List | GET | `/api/v2/themes?offset=0&limit=20` |
| Get by ID | GET | `/api/v2/themes/{themeId}` |
| Generate (async) | POST | `/api/v2/themes/generate` |
| Apply | POST | `/api/v2/themes/apply` |

### exports — `/api/v2/projects/{projectId}/exports`

| Operation | Method | Path |
|-----------|--------|------|
| Export PNG ZIP (async) | POST | `.../exports/png` |
| Export PPTX (async) | POST | `.../exports/pptx` |
| Get export status | GET | `.../exports/{exportId}` |
| Cancel export | POST | `.../exports/{exportId}/cancel` |

### jobs — `/api/v2/jobs`

| Operation | Method | Path |
|-----------|--------|------|
| Get job status | GET | `/api/v2/jobs/{activityId}` |

| Status | Meaning |
|--------|---------|
| `SCHEDULED` | Queued, not started |
| `RUNNING` | Currently executing |
| `PENDING` | Waiting for signal |
| `DONE` | Completed (check `output`, `failureReason`) |
| `CANCELED` | Was cancelled |

---

## Contributing

1. Each skill lives in `skills/<name>/SKILL.md`.
2. Follow the format: YAML frontmatter → description → TypeScript types → endpoint sections with curl → agent behavior → response format.
3. Always align types with the actual controller data classes in `service-poc`.
