# Agent instructions for this repository

This repo holds **Markdown knowledge-base entries** distilled from Launch support conversations.

## How to create or update an entry

1. Paste or attach the **raw support thread** (Slack export, screenshot text, forwarded email chain).
2. Ask the assistant to produce (or refine) an MD file using the **support pattern** (see `.cursor/rules/support-pattern.mdc`).

Rules live here:

- `.cursor/rules/support-pattern.mdc` (applied in this workspace)

Reference **`@.cursor/rules/support-pattern.mdc`** or **`@AGENTS.md`** when starting a chat so extraction stays in context.

## Folder structure

Entries are organized into **feature folders** (see the rule's "File naming & location" section for the full list and what belongs where):

- `custom-domains-ssl/`
- `edge-functions-rewrites/`
- `deployments-builds/`
- `caching-cdn/`
- `logs-monitoring/`
- `networking-connectivity/`
- `integrations/`

Save new entries into the folder matching the **primary Launch feature** in the query. Create a new kebab-case feature folder if none fit — don't leave entries at repo root. Meta files (`AGENTS.md`, `CLAUDE.md`, `Brief understanding of launch.md`) stay at root.
