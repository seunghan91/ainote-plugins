---
name: ainote-handoff
description: Save and restore session handoff notes for cross-device or cross-session continuation using the ainote MCP tools (handoff_save, handoff_list, handoff_get). Use when the conversation is approaching a context limit, when work needs to continue in a new session, or when picking up work on a different machine.
---

# ainote Session Handoffs

ainote stores session handoffs as timestamped notes in the user's ainote vault so a future session (same device or a different one) can resume in minutes instead of re-discovering context from scratch.

## Onboarding (do this first if unauthenticated)

If a tool call returns an authentication error, get an MCP key before continuing:

- Call `signup_and_get_key` (new user) or `login_and_get_key` (existing user) — both work without an `Authorization` header.
- Or run `npx @ainote/mcp login` on the user's machine (RFC 8628 device-flow login).

Either path returns a key. Store it as `AINOTE_API_KEY` and use it in the `Authorization: McpKey <key>` header for every other call.

## Saving a handoff — `handoff_save`

Call this before ending a session that has unfinished work, is near the context limit, or needs to hand off to another device.

Arguments:
- `project` (string, required) — short project slug, e.g. `ainote`, `logi`.
- `topic` (string, required) — short topic slug, lowercase-hyphen, e.g. `phase-d-port`, `oauth-fix`.
- `content` (string, required) — the full handoff text.
- `time` (string, optional) — `HHMM` in 24h KST, to disambiguate multiple handoffs saved the same day (appended to the slug).

The handoff is stored at `handoffs/{project}-{topic}-{YYYY-MM-DD}.txt` (or with the time suffix) in the user's primary vault.

### What `content` must contain

Write it as a self-contained brief — the next session has none of the current conversation. Cover:
- **Current state** — what is done, what is in progress.
- **Files touched** — exact paths, so the next session can jump straight in.
- **Build/test status** — does it currently pass? What was last run?
- **Decisions made** — anything non-obvious that shaped the approach.
- **Next steps** — the concrete next action, not a vague direction.
- **Known pitfalls** — anything that already tripped up this session, so it isn't rediscovered the hard way.

### Large bodies

If `content` is roughly 10KB or larger, some edge WAFs can false-positive block it as an injection attempt. To bypass this, base64-encode the text and pass it as `content_b64` instead of `content` — the server decodes it before storing.

## Listing handoffs — `handoff_list`

Arguments (all optional):
- `project` — filter to handoffs for one project.
- `status` — filter by frontmatter status (`in_progress`, `paused`, `completed`, `blocked`). Only applies to handoffs that carry v2 frontmatter.

Returns entries most-recent-first. Handoffs older than 7 days are automatically excluded — a daily server-side job purges them, so this tool is not meant for long-term archival, only short-lived continuation notes.

## Retrieving a handoff — `handoff_get`

Arguments:
- `project` (required) — must match the slug used at save time.
- `topic` (required) — must match the slug used at save time.
- `date` (optional) — if omitted, returns the most recent match.
- `time` (optional) — `HHMM`, needed only to disambiguate same-day multiple saves.

Also subject to the 7-day TTL — older handoffs are not returned.

## Typical flow

1. Session nears its context limit or the user asks to hand off / continue elsewhere.
2. Call `handoff_save` with a complete, self-contained brief as described above.
3. In the new session (same device or another one), call `handoff_list` (optionally filtered by `project`) to find the right entry, then `handoff_get` to fetch the full text and resume from it.
