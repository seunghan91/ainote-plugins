---
name: ainote-tasks
description: Create, list, update, and delete tasks and categories in ainote via MCP tools (list_tasks, create_task, update_task, delete_task, list_categories). Use when the user wants to manage a to-do list, add a reminder, check what's due, or organize tasks into categories.
---

# ainote Tasks

ainote's task tools cover full CRUD plus rich filtering. This skill covers `list_tasks`, `create_task`, `update_task`, `delete_task`, and `list_categories`.

## Onboarding (do this first if unauthenticated)

If a tool call returns an authentication error, get an MCP key before continuing:

- Call `signup_and_get_key` (new user) or `login_and_get_key` (existing user) — both work without an `Authorization` header.
- Or run `npx @ainote/mcp login` on the user's machine (RFC 8628 device-flow login).

Either path returns a key. Store it as `AINOTE_API_KEY` and use it in the `Authorization: McpKey <key>` header for every other call.

## Date convention — read before setting dates

ainote distinguishes two kinds of date value; passing the wrong shape corrupts due dates across timezones. See the project's `DATETIME_CONVENTION.md` for the full rationale — the short version:

- **CalendarDate** — `"YYYY-MM-DD"`, no time zone. Use for an all-day due date.
- **Instant** — full ISO 8601 with offset, e.g. `"2026-01-28T15:00:00+09:00"`. Use when the task has a specific time.

`create_task`/`update_task` accept `due_date` as either an ISO date or a full ISO datetime, plus a separate `due_time` (`"HH:MM"`) field for the time-of-day when the task isn't all-day. Set `is_all_day: true` to explicitly suppress time-of-day rendering. Don't hand-construct date strings — pass through what the user gave, in one of the two shapes above.

## Listing tasks — `list_tasks`

All arguments optional. Useful filters: `status` (`pending`/`completed`), `is_important`, `search` (keyword in content/notes), `location`, `category_id`, `overdue`, `due_today`, `has_notification`, and date-range pairs (`due_date_start`/`due_date_end`, `completed_date_start`/`completed_date_end`, `created_date_start`/`created_date_end`). Sort with `sort_by` (`due_date`/`created_at`/`completed_at`/`updated_at`/`is_important`) and `sort_order` (`asc`/`desc`). Paginate with `limit` (default 25, max 500) and `offset`.

## Creating a task — `create_task`

Only `content` is required. Notable optional fields:
- `due_date` / `due_time` / `is_all_day` / `start_date` — see date convention above.
- `category_id` — assign to a category (get valid IDs from `list_categories`).
- `location`, `location_lat`, `location_lng`, `travel_time` — location-aware tasks.
- `has_notification` + `reminder_timing` (minutes before, default 30) or `reminder_timings` (array, for multiple reminders — overrides `reminder_timing`).
- `repeat_rule` — `daily`/`weekly`/`monthly`/`yearly` or an RRULE string.
- `is_important` — boolean flag, default false.

## Updating a task — `update_task`

Only `id` is required; every other field is optional and only changes what's passed. To clear a field, pass the string `"null"` (not JSON null) for date/text fields — e.g. `due_date: "null"` clears the due date, `completed_at: "null"` marks a task incomplete. `reminder_timings: []` clears reminders.

## Deleting a task — `delete_task`

Soft-delete by `id`. Reversible for 30 days (a daily cleanup job purges trash at 2am KST). Returns 404 if the task doesn't exist or isn't owned by the authenticated user — don't retry blindly on that error.

## Listing categories — `list_categories`

No arguments. Read-only — returns `id`/`name`/`color`/`icon`/`task_count` tuples. Use the returned `id` values as `category_id` in `create_task`/`update_task`; there is no create/update/delete tool for categories via MCP.
