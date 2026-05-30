# wd — Work Done Skill

Post a work-done announcement to Slack, create/update a Trello card, and log time to Clockify — all from one command.

## Install

```bash
npx skills add carofi-auto/agent-skills --skill wd
```

## Prerequisites

You need to fill in all `<PLACEHOLDER>` values in `SKILL.md` before first use. Here is every key/ID you need and where to find it:

### Trello

| Placeholder | How to get it |
|---|---|
| `<TRELLO_BOARD_ID>` | Open your board → URL slug: `trello.com/b/<BOARD_ID>/...` |
| `<TRELLO_IN_PROGRESS_LIST_ID>` | Board API: `GET https://api.trello.com/1/boards/<BOARD_ID>/lists?key=&token=` → find your "In Progress" list `id` |
| `<YOUR_TRELLO_MEMBER_ID>` | `GET https://api.trello.com/1/members/me?key=&token=` → `id` field |

### Slack

| Placeholder | How to get it |
|---|---|
| `<SLACK_CHANNEL_ID>` | Open channel in Slack → right-click → "Copy link" → last segment of URL, or channel details pane |
| `<SLACK_CC_USER_ID>` | Click the user's profile → "Copy member ID" |

Your agent must have Slack MCP access with read/write permissions to the target channel.

### Clockify

| Placeholder | How to get it |
|---|---|
| `<CLOCKIFY_API_KEY>` | clockify.me → Profile Settings → API → **API key** |
| `<CLOCKIFY_WORKSPACE_ID>` | `GET https://api.clockify.me/api/v1/workspaces` with your API key → `id` of your workspace |
| `<PROJECT_ID_*>` | `GET https://api.clockify.me/api/v1/workspaces/<WORKSPACE_ID>/projects` → `id` per project |

Quick fetch for all Clockify project IDs:

```bash
curl -s -H "X-Api-Key: <YOUR_API_KEY>" \
  "https://api.clockify.me/api/v1/workspaces/<YOUR_WORKSPACE_ID>/projects?page-size=50" \
  | python3 -c "import json,sys; [print(p['id'], '—', p['name']) for p in json.load(sys.stdin)]"
```

---

## Flags & Shortcuts

| Flag | Alias | What it does |
|---|---|---|
| _(none)_ | | Full flow: git PRs + Slack announcement + Trello card + Clockify time entry |
| `--time-only` | `--to` | Clockify only. Skip everything else. |
| `--pr-only` | `--pro` | Git workflow only. Skip Slack, Trello, Clockify. |
| `--post-only` | `--po` | Slack + Trello only. Skip git and Clockify. |
| `--time <duration>` | `--t <duration>` | Pass duration directly (e.g. `--time 3h`, `--t 1.5h`, `--t 45m`). No interactive question. |
| `--range-time [range]` | `--rt [range]` | Bulk backfill: scan Slack for your announcements in the given period and batch-log to Clockify. No posting. |

### `--rt` range examples

```
/wd --rt                         → this month (default)
/wd --rt "this week"             → Monday → now
/wd --rt "last week"             → prev Mon → prev Sun
/wd --rt "last two weeks"        → 14 days ago → now
/wd --rt "last month"            → full previous calendar month
/wd --rt "2026-05-01 to 2026-05-15"  → explicit range
```

### Common combos

```
/wd --t 2h                       → full flow, 2h logged, no question
/wd --po                         → Slack + Trello only, no Clockify
/wd --to --t 3h                  → Clockify only, 3h, no Slack/Trello
/wd --pro                        → git PRs only, no Slack/Trello/Clockify
/wd --rt "last week"             → backfill last week to Clockify
```

---

## What the full flow does

1. Posts a formatted announcement to your Slack channel
2. Creates (or links) a Trello card in your sprint board
3. Attaches PRs to the Trello card
4. Posts per-repo PR links + change summary in thread
5. CCs your configured Slack user
6. Logs a billable time entry to Clockify (auto-detects project from repo/title)
