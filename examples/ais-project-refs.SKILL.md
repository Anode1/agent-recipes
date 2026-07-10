---
name: project-refs
description: Recall this project's shared references (environment URLs, version/status endpoints, deploy commands) from the local ais index by keyword. Use when a developer asks for a URL, endpoint, or deploy command, or wants to save one.
---

# Project references (via ais)

This project keeps its shared references in a local ais index (`.ais` in the
repo root); inside the repo `ais` finds it automatically. Always recall the live
value by running `ais` - the index is the source of truth and may change, so do
not hardcode values here.

## Keys in use
- `dev`, `uat` - the environments
- `version` - the version and /api/status endpoints
- `release`, `deploy` - how to deploy and publish

Confirm the current set anytime with `ais --keys`.

## Common recalls
    ais dev                  # dev base URL (plus its version, status, deploy)
    ais uat                  # uat base URL
    ais dev version          # dev version and /api/status
    ais release deploy       # the deploy and publish commands
    ais -o dev uat           # anything in either environment

## Save a reference
    ais -v "<a url, path, note, or command>" <keys>

Keep values short: store a URL or a path, not a document.

## Notes
- Everything is local and private; nothing leaves the machine.
- Prefer the existing keys so the index stays consistent; check `ais --keys`
  before inventing a new one.
