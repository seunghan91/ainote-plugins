---
name: ainote-devdocs
description: Centrally manage team dev docs — CLAUDE.md, .cursorrules, memory files, and other AI-config or project documentation — across devices using ainote MCP tools (list_dev_docs, get_dev_doc, create_dev_doc, update_dev_doc, pull_dev_docs, delete_dev_doc, list_dev_categories). Use when the user wants to sync CLAUDE.md or Cursor rules to another machine, back up memory files, or centrally distribute team dev documentation.
---

# ainote Dev Docs

Dev docs are ainote's mechanism for multi-device sync of files that normally live outside git — `CLAUDE.md`, `.cursorrules`, `.windsurfrules`, memory files, local env notes, and project planning docs. This skill covers `list_dev_docs`, `get_dev_doc`, `create_dev_doc`, `update_dev_doc`, `pull_dev_docs`, `delete_dev_doc`, and `list_dev_categories`.

## Onboarding (do this first if unauthenticated)

If a tool call returns an authentication error, get an MCP key before continuing:

- Call `signup_and_get_key` (new user) or `login_and_get_key` (existing user) — both work without an `Authorization` header.
- Or run `npx @ainote/mcp login` on the user's machine (RFC 8628 device-flow login).

Either path returns a key. Store it as `AINOTE_API_KEY` and use it in the `Authorization: McpKey <key>` header for every other call.

## Categories

Dev docs live under subcategories of `dev/`:
- `memory` — Claude/AI memory files.
- `claude` — `CLAUDE.md` and Claude-specific configs.
- `cursor` — `.cursorrules` files.
- `env` — environment notes and config references (never actual secret values).
- `docs` — general project documentation (default if `category` is omitted on create).

Call `list_dev_categories` (no arguments) to see all subcategories in use, with document counts, before filtering `list_dev_docs` by `category`.

## The multi-device pattern: `local_path` + `pull_dev_docs`

This is the core use case — centrally distributing `CLAUDE.md` / Cursor rules / memory files to every machine a team or a single user works from:

1. On the source machine, save the file with `create_dev_doc`, setting `local_path` to the absolute path where it lives on disk (e.g. `~/.claude/CLAUDE.md` or a project's `~/.claude/projects/<project-slug>/memory/MEMORY.md` — `~` is expanded).
2. On any other machine, call `pull_dev_docs` (optionally filtered by `category`). It fetches every dev doc that has a `local_path` set, creates missing parent directories, and writes each file to that path on the current machine. Run it once after registering ainote MCP on a new device, or any time to re-sync.

Only docs with `local_path` set participate in `pull_dev_docs` — plain notes without it are stored but not auto-restored to disk.

## Creating a doc — `create_dev_doc`

Required: `title` (used as filename, e.g. `ainote-memory.md`), `content` (full file content). Optional: `category` (see above, default `docs`), `content_type` (`markdown`/`json`/`yaml`/`text`, auto-detected from the title's extension if omitted), `local_path` (see pattern above), `memory_type` (`state`/`event`/`preference` — only set this for memory-category docs; see below).

### `memory_type` semantics

- `state` — the latest value replaces the past (this is what memory-search "latest state" mode reads).
- `event` — an immutable, accumulating log.
- `preference` — a user preference record.
- Omit entirely for plain documents that aren't memory entries.

## Updating a doc — `update_dev_doc`

Identify the target by `title` (+ `category` to disambiguate if the same title exists in multiple subcategories) or by `id`. `content` is required. `mode` controls how it's applied: `replace` (default), `append`, or `prepend`. Can also update `local_path` or `memory_type` in the same call.

## Reading — `get_dev_doc` and `list_dev_docs`

- `get_dev_doc` — fetch one doc's full content by `title` (+ optional `category` to disambiguate) or `id`. Set `include_versions: true` to also get version history.
- `list_dev_docs` — browse/search. Filter by `category`, `search` (title keyword), or `content_type`. Omit all filters to list everything under `dev/`.

## Deleting — `delete_dev_doc`

Soft-delete by `title` (+ `category` to disambiguate) or `id`. Reversible from trash.
