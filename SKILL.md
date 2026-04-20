---

type: skill-pack
name: layerproof-api
description: Skill pack for interacting with the Layerproof API including workspaces, projects, slide decks, files, themes, exports, and jobs.
author: compilet-dev
version: 1.1.0

includes:

* skills/workspaces/SKILL.md
* skills/projects/SKILL.md
* skills/public-files/SKILL.md
* skills/project-files/SKILL.md
* skills/slide-decks/SKILL.md
* skills/slides/SKILL.md
* skills/themes/SKILL.md
* skills/tones/SKILL.md
* skills/exports/SKILL.md
* skills/social-campaigns/SKILL.md
* skills/jobs/SKILL.md

---

# Layerproof API Skill Pack

> ⚠️ **Skill Pack Manifest**
>
> This file is **NOT a skill**.
>
> It is the **entry point for the Layerproof skill pack**.
>
> Individual skills are located in:
>
> `skills/*/SKILL.md`
>
> AI agents and crawlers should **only index those files as skills**.

---

# Overview

This repository provides a **collection of AI agent skills for the Layerproof API**.

These skills allow AI agents to interact with Layerproof to:

* Manage **workspaces**
* Create and manage **projects**
* Upload **public and project files**
* Generate **slide decks using AI**
* Edit **slides and images**
* Apply **themes**
* **Export** presentations
* Monitor **asynchronous jobs**

Each skill contains structured instructions that teach an AI agent how to interact with the Layerproof API using `curl`.

---

# Required Environment Variables

Before performing any API operations, the agent must verify that these environment variables are set on the user's machine.

## Check before every API call

Always verify the following environment variables exist before making any API request:

```bash
# Check if LAYERPROOF_BASE_URL is set
if [[ -z "${LAYERPROOF_BASE_URL}" ]]; then
  echo "ERROR: LAYERPROOF_BASE_URL is not set."
  echo "Please set it in your shell profile or .env file, then re-run."
  return 1
fi

# Check if LAYERPROOF_API_KEY is set
if [[ -z "${LAYERPROOF_API_KEY}" ]]; then
  echo "ERROR: LAYERPROOF_API_KEY is not set."
  echo "Please set it in your shell profile or .env file, then re-run."
  return 1
fi
```

## Setting environment variables

If not already set, instruct the user to add them:

```bash
echo 'export LAYERPROOF_BASE_URL=https://api.layerproof.com' >> ~/.zshrc
echo 'export LAYERPROOF_API_KEY=your-api-key-here' >> ~/.zshrc
source ~/.zshrc
```

Alternatively, for project-specific settings, create a `.env.local` file in the project root:

```bash
LAYERPROOF_BASE_URL=https://api.layerproof.com
LAYERPROOF_API_KEY=your-api-key-here
```

When using a `.env.local` file, ensure it is loaded before running any curl commands:

```bash
# For zsh, load .env.local if it exists
if [[ -f .env.local ]]; then
  set -a
  source .env.local
  set +a
fi
```

## Authentication header

All API requests must include the following header:

```
X-API-KEY: $LAYERPROOF_API_KEY
```

---

# How Agents Should Use This Skill Pack

When a user asks to perform an operation related to Layerproof:

1. Identify the **resource type** involved.
2. Load the corresponding `skills/<resource>/SKILL.md`.
3. Follow the endpoint definitions in that skill file.
4. Construct a valid `curl` request.
5. Execute the request using the environment variables.
6. Display the **raw JSON response** to the user.

If the response contains image URLs, the agent may render them.

---

# Skill Modules

Each module corresponds to a Layerproof API resource.

| Resource      | Skill File                      | Purpose                             |
| ------------- | ------------------------------- | ----------------------------------- |
| Workspaces    | `skills/workspaces/SKILL.md`    | Manage workspaces                   |
| Projects      | `skills/projects/SKILL.md`      | Create and manage projects          |
| Public Files  | `skills/public-files/SKILL.md`  | Upload reference documents          |
| Project Files | `skills/project-files/SKILL.md` | Manage files inside projects        |
| Slide Decks   | `skills/slide-decks/SKILL.md`   | Generate outlines and slide content |
| Slides        | `skills/slides/SKILL.md`        | Edit slide images and content       |
| Themes        | `skills/themes/SKILL.md`        | Generate and apply visual themes    |
| Tones         | `skills/tones/SKILL.md`         | Tone presets and apply to deck      |
| Exports       | `skills/exports/SKILL.md`       | Export PNG, PPTX, or video          |
| Social campaigns | `skills/social-campaigns/SKILL.md` | Social campaign projects       |
| Jobs          | `skills/jobs/SKILL.md`          | Poll async job status               |

---

# Handling Asynchronous Operations

Many Layerproof API endpoints run asynchronously.

When an API response returns an `activityId`, the agent should:

1. Poll the job status:

```
GET /api/v2/jobs/{activityId}
```

2. Continue polling until:

```
status = DONE
```

3. If `failureReason` exists, report the error.

---

# Typical Agent Workflow

Example workflows (the agent should automatically select the correct skill module for each step).

## Workflow A: Full presentation from scratch (with reference files and theme)

1. **Workspaces** — List workspaces (GET), then create one if needed (POST) or resolve `workspace_id` from list.
2. **Projects** — Create project (POST) with `name`, `project_kind: "SLIDE_DECK"`, optional `workspace_id`; capture `id` and `slide_deck_id`.
3. **Public files** (optional) — If the user provides reference docs: POST `/api/v2/files/prepare` → user PUTs file to `upload_url` → POST `/api/v2/files/confirm` with `s3_key`; collect `s3_key`(s) for outline.
4. **Themes** (optional) — List themes (GET); if user wants a new look: POST `/api/v2/themes/generate` with `prompt` → poll **Jobs** until DONE → note `theme_id`.
5. **Slide decks** — POST `.../outline/generate` with `prompt`, `slide_count`, optional `file_s3_keys`, `language`; capture `activity_id`.
6. **Jobs** — Poll `GET /api/v2/jobs/{activity_id}` until `status` is DONE (or CANCELED); on failure, report `failure_reason`.
7. **Slide decks** — GET deck to read `outline`; optionally PUT `.../outline` to tweak sections (title, sections, key_points, visual_suggestion).
8. **Themes / slide decks** — If a theme was generated, POST `/api/v2/themes/apply` with `slide_deck_id` and `theme_id` (optional `regenerate_slides`). Then POST `.../slides/batch-generate` (optionally with `generation_type`, `aspect_ratio`); capture `activity_id`.
9. **Jobs** — Poll `GET /api/v2/jobs/{activity_id}` until DONE.
10. **Slide decks** — GET deck again; if a single slide needs regeneration, POST `.../slides/{sectionId}/generate-transcript` or `.../generate-image` etc., then poll job.
11. **Slides** (optional) — For a specific slide: POST image-edit or object-removal → poll job → POST accept-image-edit with `live_object_id`.
12. **Exports** — POST `.../projects/{projectId}/exports/pptx`, `.../exports/png`, or `.../exports/video`; capture `export_id`; poll GET `.../exports/{export_id}` until COMPLETED; present `download_url` to user.

## Workflow B: Multi-format export and error handling

1. **Projects** — GET project by ID (or list and pick) to ensure it exists and has `slide_deck_id`.
2. **Slide decks** — GET deck; confirm `metadata.completed_slides` meets expectations before export.
3. **Exports** — Start PNG export (POST `.../exports/png`); get `exportId`. In parallel or after, start PPTX export (POST `.../exports/pptx`); get second `exportId`.
4. **Exports** — Poll each `GET .../exports/{exportId}` until status is COMPLETED or FAILED. If FAILED, show `errorMessage` and suggest retry or check project. When COMPLETED, show `downloadUrl` and `expiresAt`.
5. If user cancels: POST `.../exports/{exportId}/cancel` for in-progress exports.

## Workflow C: Project files and outline with project-scoped assets

1. **Projects** — Create or get project; note `project_id`. Get or create a directory in the project (per project-files API if list directories exists) or use a known `directory_id`.
2. **Project files** — POST prepare with `path`, `file_name`, `mime_type`, `size` → user uploads to `upload_url` → POST confirm with `file_id`. Optionally GET file or download-url to verify.
3. **Slide decks** — Use project’s `slide_deck_id`; generate outline with `file_s3_keys` from project file S3 keys (if API accepts them) or use public files flow and pass `file_s3_keys` from public prepare/confirm.
4. Continue with outline → poll job → update outline → batch generate → poll → export as in Workflow A.

The agent should automatically select the correct skill module for each step and handle polling, error responses, and multi-step dependencies.

---

# Skill Discovery

Agents should load the following files when interacting with Layerproof:

* skills/workspaces/SKILL.md
* skills/projects/SKILL.md
* skills/public-files/SKILL.md
* skills/project-files/SKILL.md
* skills/slide-decks/SKILL.md
* skills/slides/SKILL.md
* skills/themes/SKILL.md
* skills/tones/SKILL.md
* skills/exports/SKILL.md
* skills/social-campaigns/SKILL.md
* skills/jobs/SKILL.md

Each skill file contains:

* API endpoint definitions
* request and response types
* curl examples
* agent decision logic
* response handling rules

---

# Repository Structure

```
agent-skill-layerproof
│
├─ SKILL.md
├─ marketplace.json
├─ README.md
│
└─ skills
   ├─ workspaces
   │   └─ SKILL.md
   ├─ projects
   │   └─ SKILL.md
   ├─ public-files
   │   └─ SKILL.md
   ├─ project-files
   │   └─ SKILL.md
   ├─ slide-decks
   │   └─ SKILL.md
   ├─ slides
   │   └─ SKILL.md
   ├─ themes
   │   └─ SKILL.md
   ├─ tones
   │   └─ SKILL.md
   ├─ exports
   │   └─ SKILL.md
   ├─ social-campaigns
   │   └─ SKILL.md
   └─ jobs
       └─ SKILL.md
```

---