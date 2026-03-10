---

type: skill-pack
name: layerproof-api
description: Skill pack for interacting with the Layerproof API including workspaces, projects, slide decks, files, themes, exports, and jobs.
author: compilet-dev
version: 1.0.0

includes:

* skills/workspaces/SKILL.md
* skills/projects/SKILL.md
* skills/public-files/SKILL.md
* skills/project-files/SKILL.md
* skills/slide-decks/SKILL.md
* skills/slides/SKILL.md
* skills/themes/SKILL.md
* skills/exports/SKILL.md
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

Before performing any API operations, ensure these environment variables are set:

```bash
echo 'export LAYERPROOF_BASE_URL=https://api.layerproof.com' >> ~/.zshrc
echo 'export LAYERPROOF_API_KEY=your-api-key-here' >> ~/.zshrc
source ~/.zshrc
```

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
| Exports       | `skills/exports/SKILL.md`       | Export projects as PNG or PPTX      |
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

Example workflow for generating a presentation:

1. Create project
2. Generate outline
3. Poll job status
4. Batch generate slides
5. Poll job status
6. Retrieve slide deck
7. Export presentation

The agent should automatically select the correct skill module for each step.

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
* skills/exports/SKILL.md
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
   ├─ exports
   │   └─ SKILL.md
   └─ jobs
       └─ SKILL.md
```

---