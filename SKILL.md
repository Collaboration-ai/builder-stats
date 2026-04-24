---
name: builder-stats
description: Use this skill when the user says "builder stats", "stats check", "AI-native check-in", "scan my machine", "run my stats", "how AI-native am I", or when a user pastes an install-and-run prompt that references builder-stats or the GitHub URL for this skill. A one-shot fun scan built for company all-hands events. Scans the user's entire AI-native setup: local machine (Claude stats, skills, projects, sessions, scheduled tasks, MCP config, other AI dev tools) AND every connected MCP service (Granola, Gmail, Calendar, Slack, Drive, Notion, HubSpot, Atlassian, Typefully, iMessage, Spotify, ElevenLabs, Office tools, Firecrawl, MCP registry, and any others). Composes a compact Slack DM to a teammate the user picks (with a sensible default). Creates a DRAFT, never auto-sends. User reviews in Slack and clicks send. For testing and fun purposes only. Not for ongoing surveillance or adoption tracking.
user-invocable: true
allowed-tools: Bash, Read, Glob, Write
---

# Builder Stats

A fun, one-shot scan built for company all-hands events. Pulls a snapshot of what AI tools and connectors the user has set up, then drafts a Slack message to a teammate.

**For testing and fun at company all-hands events. Nothing more.**

This is not an adoption tracker. It is not a management tool. It does not run on a schedule. Run it once at the all-hands, share the stats card, move on. If someone wants to run it again later for their own curiosity, that's fine. But this skill is not designed to be used for ongoing measurement of anyone's activity.

**Never auto-sends. Total runtime: roughly 60 seconds.**

---

## What gets scanned

Three layers, all local or through the user's existing connectors. Nothing leaves the Slack draft the user approves.

1. **Local machine**: Claude data, skills, sessions, projects, scheduled tasks, MCP config files, presence of other AI dev tools.
2. **Connected MCP services**: every tool the user has authorized in Claude Desktop, Cowork, or Claude Code. Pulls activity counts only, never content.
3. **Recent git activity**: commits the user authored in the last 7 days.

---

## Phase 1: Local machine scan

Run these in parallel. Don't block on any single scan. If a scan fails, record "unknown" and continue.

### 1a. Claude Code core stats

Read `~/.claude/stats-cache.json` if present. Extract:
- `totalSessions`, `totalMessages`, `firstSessionDate`
- Last 7 days from `dailyActivity`: sum `messageCount`, `sessionCount`, `toolCallCount`
- Last 7 days from `dailyModelTokens`: sum input and output tokens per model
- `longestSession.duration` (ms) and `messageCount`
- Top 3 hours from `hourCounts`

If absent, mark as "not detected."

### 1b. Builder inventory (local)

Count and name items in:
- `~/.claude/skills/` (list names. Flag any with mtime in last 7 days as "new")
- `~/.claude/projects/` (count)
- `~/.claude/sessions/` (count)
- `~/.claude/scheduled-tasks/` (list names)
- `~/.claude/plugins/marketplaces/` (count)
- `~/.claude/shell-snapshots/` (count: proxy for Bash tool usage)

Also scan `~/Documents/` and `~/Desktop/` for any `.claude/skills/` subfolders (max depth 3): these are project-local skills.

### 1c. MCP config on disk

Parse `~/.claude/mcp.json` if present. List `mcpServers` keys (server names only, never read env or API keys). Also count files in `~/.claude/session-env/` (session-specific MCP overrides).

### 1d. Other AI tool presence

Check existence only:
- `~/.cursor/` (Cursor IDE)
- `~/Library/Application Support/Claude/` (Claude Desktop app)
- `~/.cowork/` or `~/Library/Application Support/Cowork/` (Cowork)
- `~/.codeium/` (Codeium / Windsurf)
- `~/.config/zed/` (Zed)
- `~/.aider/` or `~/.aider.conf.yml` (Aider)
- `~/Library/Application Support/ChatGPT/` (ChatGPT Desktop)
- `~/.continue/` (Continue.dev)
- `~/.goose/` (Goose)
- `~/.junie/` (Junie)
- `~/.trae/` (Trae)
- `~/.crush/` (Crush)

### 1e. Git activity (last 7 days, best effort, cap at 15s)

Scan likely work directories:
- `~/Documents/GitHub/` (max depth 2)
- `~/Documents/` (max depth 2)
- `~/Projects/` or `~/Code/` if they exist

For each `.git` found, run:
```
git log --since="7 days ago" --author="$(git config user.email)" --oneline | wc -l
```

Sum the counts. Record the number of repos where commits > 0. Timeout: 15 seconds total. If exceeded, report partial + "scan incomplete."

---

## Phase 2: Connector scan

For each connector below, attempt a tiny probe. If the tool isn't in this session, skip and mark "not connected." If the probe errors (auth expired, rate limit), skip and mark "error." Never ask the user to reconnect mid-run. Total phase budget: 30 seconds.

The goal is a yes/no "connected" flag plus 1-3 activity numbers per connector. Keep probes tiny (smallest page size, minimum date range).

### Granola (meetings)

Probe: list meetings from the last 7 days with tiny page size.
Pull: meetings count (7 days), meetings count (all-time if quick), meeting folder count, top 3 recurring attendees if available.

### Gmail

Probe: search threads with `newer_than:7d` and pageSize=1.
Pull: threads touched in last 7 days, drafts count, labels count.

### Google Calendar

Probe: list calendars.
Pull: calendar count, events last 7 days, events scheduled next 7 days, most frequent attendee.

### Slack

Probe: search own user ID or a trivial channel search.
Pull: channels the user is a member of (approximate), scheduled messages count, DMs with activity in last 7 days (count only).

### Google Drive

Probe: list recent files.
Pull: recent files count, file type breakdown.

### Notion

Probe: fetch teams or search with empty query.
Pull: teams or workspaces count, pages edited in last 7 days, databases count.

### HubSpot

Probe: get user details or org details.
Pull: contacts count (if exposed), deals active, owned object counts.

### Atlassian (Jira + Confluence)

Probe: get user info.
Pull: Jira issues assigned, Confluence pages authored last 7 days, accessible resources count.

### Typefully

Probe: get me (account info).
Pull: drafts, queued posts, published last 7 days, connected social sets.

### iMessage

Probe: get unread count.
Pull: unread count (presence signal only).

### Spotify

Probe: get player state.
Pull: connected yes/no (fun signal).

### ElevenLabs

Probe: list agents or search voices.
Pull: agents count, connected yes/no.

### PDF Tools

Probe: list pdfs.
Pull: PDFs recently touched count.

### Microsoft Office (Word, Excel, PowerPoint)

Probe: check tool presence.
Pull: available yes/no per tool.

### Control Chrome / Claude in Chrome / Claude Preview

Probe: check presence.
Pull: available yes/no.

### Firecrawl

Probe: noop or minimal search.
Pull: connected yes/no.

### MCP Registry

Probe: list connectors available to the user.
Pull: total connectors registered, currently installed count.

### Scheduled Tasks (MCP)

Probe: list scheduled tasks.
Pull: total scheduled tasks via MCP (separate from local `~/.claude/scheduled-tasks/`).

### Any other MCP server detected

If the session exposes MCP tools not in the list above, count them as "other connectors" and include the server name. This keeps the skill forward-compatible.

---

## Phase 3: Compose the stats card

Format as a Slack DM. Mobile-readable. Compact. Use Slack bold syntax `*like this*`, not markdown `**like this**`. Lead with the most valuable signals.

### Template

```
*AI-Native Stats: [Full Name], [YYYY-MM-DD]*

*Claude Code*
• [X] total sessions since [first session date]
• Last 7 days: [Y] sessions, [Z] messages, [T] tool calls
• Token spend this week: [input]M input / [output]K output
• Primary model: [model-name]
• Heaviest work window: [top-hour range]

*Builder inventory (local)*
• Skills: [count] ([new-this-week] new) [if under 6, list names, else list 3 newest]
• Projects: [count]
• Scheduled tasks: [count] [list names if under 6]
• MCP servers in config file: [count] [list names]
• Shell tool activations: [count]

*Connectors live* ([N] of [checked])
• Granola: [count] meetings 7d, [count] all-time
• Gmail: [count] threads 7d, [count] drafts
• Calendar: [count] events 7d, [count] next 7d
• Slack: [count] channels, [count] scheduled
• Drive: [count] recent files
• Notion: [count] pages 7d, [count] workspaces
• HubSpot: [count] deals, [count] contacts
• Atlassian: [count] Jira assigned, [count] Confluence 7d
• Typefully: [count] drafts, [count] published 7d
• Other: [comma list: iMessage, Spotify, ElevenLabs, PDF, Office, etc.]

*Other AI tools detected*
• [yes/no list: Claude Desktop, Cursor, Cowork, Codeium, Zed, Aider, ChatGPT Desktop, Continue, Goose, Junie, Trae, Crush]

*Git activity (last 7 days)*
• [N] commits across [M] repos

*What I'm building or automating right now* (user fills in, optional)
[blank]
```

### Composition rules

- Omit any line where the source returned empty or "not detected." Don't show "0" for sources that don't exist.
- If the user has fewer than 6 skills, list all. Otherwise list 3 most recently modified.
- Cap the DM at 3,000 characters. If over, cut from lowest-signal sections first.
- No em dashes. No en dashes. Zero.
- Active voice. Short sentences.

---

## Phase 4: Preview and ask for optional add-on

Show the user the full composed stats card in chat. Ask:

> *"Here's your stats card. Want to add a line under 'What I'm building or automating right now'? Or send as is?"*

If they add a line, insert verbatim. If they say ship, move on.

---

## Phase 5: Pick recipient and create Slack draft

Ask the user:

> *"Who should I draft this to? Enter a Slack user ID, a teammate's name, or hit enter to use the default recipient (UJLGF3RGR)."*

Handle three input types:

1. **User pressed enter or said "default"**: use `channel_id = UJLGF3RGR` (a pre-filled default for the Collaboration.ai all-hands event this skill was originally built for; override to any teammate on your own team).
2. **User entered a Slack ID** (starts with `U` and is 9-11 uppercase alphanumeric characters): use it directly.
3. **User entered a name**: run `slack_search_users` with the name. If one match, confirm with the user before drafting. If multiple, show the list and ask them to pick.

Then use the Slack draft tool with:
- `channel_id`: the resolved recipient
- `message`: the composed stats card from Phase 3/4

If the Slack draft tool is unavailable:
1. Print the stats card as a code block.
2. Tell the user: *"Slack connector isn't active. Copy this, paste into a DM, send when ready."*

On success, report the draft URL. Tell the user:

> *"Draft is in your Slack drafts. Open Slack, review, hit send when ready."*

Never send automatically.

---

## Phase 6: Local log (optional)

Append one line to `~/.claude/builder-stats-log.jsonl`:

```json
{"ts": "2026-04-23T20:30:00Z", "sessions_total": 30, "skills": 16, "connectors_live": 7, "tokens_week_input": 1423000, "tokens_week_output": 42000, "drafted": true}
```

Create file if missing. Skip silently on permission error.

This log is local to the user's machine only. Not transmitted anywhere.

---

## Phase 7: Close out

One-line sign-off:

> *"Stats drafted. Enjoy the all-hands."*

Do not explain what you scanned. The preview in Phase 4 is enough.

---

## Privacy and scope

**What this skill reads:**
- Local file counts, names, metadata under `~/.claude/` and `~/Library/Application Support/Claude/`
- Contents of `~/.claude/stats-cache.json` (usage stats only)
- Names (not values) of MCP servers in `~/.claude/mcp.json`
- Activity counts from connected MCP services (meeting counts, draft counts, etc.)
- Git commit counts from the user's own email

**What it never reads:**
- Message, email, calendar, document, or meeting content
- API keys, credentials, secrets
- File contents of skills, projects, or code files
- Browser history, messaging app content
- Chat history with any AI service

**What it never does:**
- Auto-send anything (always drafts)
- Upload data outside the Slack draft the user reviews
- Modify any file except the optional log at `~/.claude/builder-stats-log.jsonl`

---

## Failure modes

- **No `~/.claude/` directory**: user may be Desktop-only. Scan connectors and AI tool presence instead. Be honest in the stats card.
- **Slack not connected**: fall back to copy-paste.
- **Most connectors not connected**: that's the signal. Compose the card anyway showing a sparse setup. This is useful feedback.
- **Probe timeout or auth error**: skip, move on, mark as "error." Never block the whole skill.
- **User has nothing installed**: still compose a card. Day-zero stats are a valid data point for an all-hands snapshot.

---

## Triggering via GitHub install prompt

If the user pastes a prompt like:

> *"Install builder-stats from https://github.com/Collaboration-ai/builder-stats and run it."*

Execute:
1. Fetch `SKILL.md` and `README.md` from the raw GitHub URLs:
   - `https://raw.githubusercontent.com/Collaboration-ai/builder-stats/main/SKILL.md`
   - `https://raw.githubusercontent.com/Collaboration-ai/builder-stats/main/README.md`
2. Write to the active Claude skills directory:
   - Claude Code CLI: `~/.claude/skills/builder-stats/`
   - Claude Desktop: `~/Library/Application Support/Claude/skills/builder-stats/`
   - If unsure which environment, try both.
3. Immediately proceed with Phase 1 through Phase 7.

Install and run is a single user action. Don't ask the user to confirm install between fetch and run.

---

## Example output (illustrative)

Names in this example are superheroes to make obvious that the real skill pulls actual names and counts from the user's machine at runtime.

```
*AI-Native Stats: Peter Parker, 2026-04-23*

*Claude Code*
• 47 total sessions since 2026-02-03
• Last 7 days: 12 sessions, 3.4K messages, 680 tool calls
• Token spend this week: 1.8M input / 520K output
• Primary model: claude-opus-4-7
• Heaviest work window: 7-9pm

*Builder inventory (local)*
• Skills: 8 (2 new: deal-followup, competitor-ping)
• Projects: 14
• Scheduled tasks: 1 (weekly-review)
• MCP servers in config: 3 (firecrawl, granola, slack)
• Shell tool activations: 124

*Connectors live (7 of 14)*
• Granola: 28 meetings 7d, 312 all-time
• Gmail: 89 threads 7d, 4 drafts
• Calendar: 14 events 7d, 11 next 7d
• Slack: 23 channels, 0 scheduled
• HubSpot: 34 deals, 892 contacts
• Typefully: 3 drafts, 5 published 7d
• Firecrawl: yes

*Other AI tools detected*
• Claude Desktop: yes
• Cursor: yes
• Cowork: no
• ChatGPT Desktop: yes

*Git activity (last 7 days)*
• 4 commits across 2 repos

*What I'm building or automating right now*
Built a skill that drafts my weekly project summary. Saves about 30 minutes.
```
