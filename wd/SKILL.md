---
name: wd
description: "Post a work-done announcement to a Slack channel and create/update a Trello card, then log time to Clockify. Use after completing any task that produces PRs, fixes, or deployable changes. Supports --skip-time, --skip-post, --time <duration>, and --range-time for bulk backfill."
---

# Work Done (/wd)

> **Setup required before use.** See `README.md` for all prerequisite IDs and keys.

## Fixed IDs — configure once, never look up again

Replace all `<PLACEHOLDER>` values with your own before using this skill.

| Resource | Placeholder |
|---|---|
| Trello sprint board ID | `<TRELLO_BOARD_ID>` |
| Trello "In Progress" list ID | `<TRELLO_IN_PROGRESS_LIST_ID>` |
| Your Trello member ID | `<YOUR_TRELLO_MEMBER_ID>` |
| Slack announce channel ID | `<SLACK_CHANNEL_ID>` |
| Primary CC Slack user ID | `<SLACK_CC_USER_ID>` |

## Clockify Fixed IDs — configure once, never look up again

| Resource | Placeholder |
|---|---|
| Clockify API Key | `<CLOCKIFY_API_KEY>` |
| Clockify Workspace ID | `<CLOCKIFY_WORKSPACE_ID>` |
| Project: web.admin (or equivalent) | `<PROJECT_ID_WEB_ADMIN>` |
| Project: app.partners (or equivalent) | `<PROJECT_ID_APP_PARTNERS>` |
| Project: web.customer (or equivalent) | `<PROJECT_ID_WEB_CUSTOMER>` |
| Project: web.partner (or equivalent) | `<PROJECT_ID_WEB_PARTNER>` |
| Project: app.customer (or equivalent) | `<PROJECT_ID_APP_CUSTOMER>` |
| Project: AI / agents | `<PROJECT_ID_AI>` |
| Project: Miscellaneous / fallback | `<PROJECT_ID_MISC>` |

### Project detection rules (first match wins)

Map keywords found in repo names or the task title to a Clockify project ID. Adapt to your project naming.

| Keyword in repos/title | Maps to |
|---|---|
| `web.admin` / `helpdesk` / `bff` | `<PROJECT_ID_WEB_ADMIN>` |
| `app.partner` | `<PROJECT_ID_APP_PARTNERS>` |
| `web.customer` | `<PROJECT_ID_WEB_CUSTOMER>` |
| `web.partner` | `<PROJECT_ID_WEB_PARTNER>` |
| `app.customer` | `<PROJECT_ID_APP_CUSTOMER>` |
| `carowl` / `ai` / `agent` / `leaderboard` | `<PROJECT_ID_AI>` |
| anything else | `<PROJECT_ID_MISC>` |

---

## Parameters

| Flag | Alias | Behavior |
|---|---|---|
| `--skip-time` | `--st` | Run normal Slack/Trello flow. Skip Clockify entirely. |
| `--skip-post` | `--sp` | Skip Slack + Trello. Only log time to Clockify. |
| `--time <duration>` | `--t <duration>` | Pre-supply duration (e.g. `--time 3h`, `--t 1.5h`, `--t 45m`). Skips the interactive duration question. |
| `--range-time [range]` | `--rt [range]` | Bulk backfill mode. Scans Slack channel for the given range and batch-logs to Clockify. No Slack/Trello posting. Range defaults to "this month". |

---

## Git workflow (only if PRs not yet open)

Per repo: create `feature/<slug>` from `develop` → stage named files → commit (conventional, ≤72 char header) → push → find existing integration branch (`git branch -r | grep integration`, pick highest suffix) → open 2 PRs: `--base develop` and `--base integration/<branch>`.

---

## Normal flow (no flags, or `--st`, or `--sp`)

1. **Gather from context** — title, 1–2 line summary, PRs grouped by repo (base + URL), existing Trello card URL if any, bullet list of changes, extra CC names if specified.

2. **Trello card** _(skip if `--sp`)_ — if card exists in context: use its ID (from URL slug). If not: `create_card` → `idList: <TRELLO_IN_PROGRESS_LIST_ID>`, `idMembers: <YOUR_TRELLO_MEMBER_ID>`. Save card ID + URL.

3. **Trello attachments** _(skip if `--sp`)_ — one `add_attachment` per PR not already attached. Name: `<repo> → <base-branch>`.

4. **Slack main message** _(skip if `--sp`)_ → channel `<SLACK_CHANNEL_ID>`:
   `<emoji> **<Title>**\n\n<1–2 line summary>\n\n<Trello URL>`
   Emoji: `:shield:` security · `:rocket:` features · `:white_check_mark:` fixes · `:tools:` infra
   Save returned `message_ts`.

5. **Slack thread — per repo** _(skip if `--sp`)_:
   `:white_check_mark: **<repo>** — <one-line summary>\n- → \`<branch>\`: <PR URL>`

6. **Slack thread — what changed** _(skip if `--sp`)_:
   `**What changed:**\n- <change>\n- <change>` — one line per change, no descriptions.

7. **Slack thread — CC** _(skip if `--sp`)_ (always last): `cc <@SLACK_CC_USER_ID>` + any extras on the same line.

8. **Clockify time entry** _(skip if `--st`)_ — if `--time`/`--t` was passed, use that duration directly. Otherwise ask: `"Clockify: how long did this take? (e.g. 2h, 1.5h, 45m — or 'skip')"`. If not skipped:
   - Detect `projectId` from repo/title context using the project detection rules above. Default to `<PROJECT_ID_MISC>`.
   - Compute `end` = current UTC datetime in ISO 8601 (`date -u +"%Y-%m-%dT%H:%M:%SZ"`). Compute `start` = `end` minus duration.
   - Run via Bash:
     ```bash
     curl -s -X POST \
       -H "X-Api-Key: <CLOCKIFY_API_KEY>" \
       -H "Content-Type: application/json" \
       -d "{\"start\":\"<start>\",\"end\":\"<end>\",\"description\":\"<title>\",\"projectId\":\"<projectId>\",\"billable\":true}" \
       "https://api.clockify.me/api/v1/workspaces/<CLOCKIFY_WORKSPACE_ID>/time-entries"
     ```
   - Verify response contains `"id"` field. Report project name + duration logged.

9. Return Trello card URL + Slack message link + Clockify entry confirmation (or "skipped").

---

## `--range-time` / `--rt` flow (bulk backfill)

Use case: forgot to log time for a period. Scans your Slack announcements and batch-logs them to Clockify.

### Range parsing (step 1)

Parse the optional string after `--rt`. If absent, default to `"this month"`.

| Input | `oldest` | `latest` |
|---|---|---|
| _(none)_ / `"this month"` | First second of current month (UTC) | Now |
| `"this week"` | Last Monday 00:00:00 UTC | Now |
| `"last week"` | Monday of previous week 00:00:00 UTC | Last Sunday 23:59:59 UTC |
| `"last two weeks"` / `"last 2 weeks"` | 14 days ago 00:00:00 UTC | Now |
| `"last N weeks"` / `"last N days"` | N×7 days (or N days) ago 00:00:00 UTC | Now |
| `"last month"` | First second of previous calendar month UTC | Last second of previous calendar month UTC |
| `"YYYY-MM-DD to YYYY-MM-DD"` | Given start date 00:00:00 UTC | Given end date 23:59:59 UTC |

Compute timestamps via Bash `date` commands. Always confirm resolved range before fetching: `"Scanning: 2026-05-01 → 2026-05-24"`.

### Steps

1. **Compute range** — parse range string → compute `oldest` + `latest` UTC Unix timestamps. Print resolved dates for confirmation.

2. **Fetch all channel messages** — read `<SLACK_CHANNEL_ID>` with `oldest` set, paginate until exhausted. Collect only top-level messages from your user ID that look like work announcements: has a Trello link, a bold title (`*text*`), or a work emoji at start (`:rocket:`, `:white_check_mark:`, `:tools:`, `:shield:`, `:ladybug:`). Skip casual messages. If a single message contains multiple entries each with their own Trello link, split into individual items.

3. **Present numbered list**:
   ```
   #1  [YYYY-MM-DD] <title> — Project: <detected-project-name>
   #2  [YYYY-MM-DD] <title> — Project: <detected-project-name>
   ```

4. **Ask for durations in one shot**:
   `"Enter durations comma-separated in order. Use 'skip' to skip. Example: 2h, 1.5h, skip, 3h"`

5. **Log each non-skipped item** — for each with a duration:
   - Detect `projectId` from title/content using project detection rules.
   - Use the message's actual date for `start` (09:00:00 UTC). Chain same-day entries (second starts where first ends). Compute `end` = `start` + duration.
   - POST to Clockify with `"billable": true` (same curl as normal flow).

6. **Report summary**:
   ```
   Logged N entries to Clockify:
   ✓ [YYYY-MM-DD] <title> — 2h → web.admin
   - [YYYY-MM-DD] <title> — skipped
   ```

---

## Rules

- Use `thread_ts` for all thread replies (steps 5–7).
- Do not post if no PRs or deliverables — ask what to announce instead.
- Primary Slack user is always CC'd. Never look up IDs at runtime — use the configured values above.
- `--rt` never posts to Slack or Trello — read-only on those systems.
- All Clockify entries are always `"billable": true`.
