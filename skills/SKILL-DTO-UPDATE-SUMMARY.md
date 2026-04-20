# Skill / API sync summary (2026-04)

## What changed

Skills in `agent-skill-layerproof` were aligned with the **public API** in `service-poc/service` (Micronaut controllers under `app/api-application/.../publicapi/`).

### New or expanded skill modules

| Skill | Controller(s) | Notes |
|-------|----------------|-------|
| **tones** | `PublicApiToneController` | New skill: `/api/v2/tones` — preset CRUD, apply, save-as-preset, duplicate. |
| **social-campaigns** | `PublicApiSocialCampaignController` | Rewritten: variations, captions, theme, exports, citations/history; removed obsolete “platform post” endpoints not present on controller. |
| **workspaces** | `PublicApiWorkspaceController` | Added deleted list, restore, permanent delete, list files. |
| **projects** | `PublicApiProjectController` | Added visibility, recent, deleted list, restore, permanent delete. |
| **project-files** | `PublicApiProjectFileController` | Added AI files, subdirectories, resolve-assets, resolve-paths-to-ids, preview-url. |
| **exports** | `PublicApiExportController` | Added **video** export; status types use snake_case in docs (`export_id`, `download_url`). |
| **themes** | `PublicThemeController` | Added save, update, delete, regenerate, unapply, `by-user-id`; corrected save payload (requires `visual_style_description` + `preview_s3_key`). |
| **slide-decks** | `PublicApiSlideDeckController` | Outline section fields (`url_references`, `reference_image_paths`, `slide_intent`); removed invalid `outline_type` on generate; expanded endpoint table (citations, TTS, import, layout). |
| **slides** | `PublicApiSlideController` | `SaveKonvaNodesRequest` matches code (no `flattened_image_path`). |
| **jobs** | `PublicApiJobController` | `JobStatusResponse`: `output` is a map; added `live_object_id`. |

### Pack manifest

- Root `SKILL.md` and `marketplace.json` version **1.1.0**; includes **tones** and **social-campaigns** in the pack list.

### Convention

- Request/response types use **snake_case** field names where the JSON API uses snake_case (some clients may send camelCase; server accepts per Jackson/Micronaut config).
- After changing a public controller or DTO, update the matching `skills/<domain>/SKILL.md`.
