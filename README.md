# Builder Stats

A one-shot fun scan of your AI-native setup, built for company all-hands events. Pulls a snapshot of what AI tools and connectors you have configured, then drafts a Slack message to a teammate with your stats card.

**For testing and fun at company all-hands events. Nothing more.**

This is not an adoption tracker. It is not a management tool. It does not run on a schedule. Run it once at the all-hands, share your stats card for a laugh, move on. The code is public so any team can adapt it for their own event.

---

## Install and run (one step)

Open Claude Desktop, Claude Code, or Cowork. Paste this prompt into a new chat:

```
Install the builder-stats skill from
https://github.com/Collaboration-ai/builder-stats

Fetch SKILL.md and README.md from that repo, save them to my Claude
skills folder, then run the skill immediately to:

1. Scan my machine (Claude sessions, skills, projects, tasks, tokens).
2. Scan every MCP connector I have (Granola, Gmail, Calendar, Slack,
   Drive, Notion, HubSpot, Atlassian, Typefully, and any others).
3. Detect other AI tools I'm running (Cursor, Cowork, Claude Desktop, etc).
4. Compose a compact Slack DM to a teammate I pick at runtime.
5. Create the Slack draft. Never auto-send.

Show me the draft preview before creating the Slack draft so I can add
a line about what I'm building or automating right now.
```

That's it. Claude fetches from GitHub, installs the skill, runs the scan, shows you a preview, asks who to send to, then drops a draft into your Slack.

**After the first run**, the skill stays installed. Next time just say `builder stats` or `run my stats`.

---

## What gets scanned

Three layers, all local or through your existing connectors. Nothing leaves the draft you approve.

**Your machine**
- Claude Code usage: sessions, messages, tokens per model, tool calls, heaviest work hours
- Skills installed (count plus names)
- Projects, sessions, scheduled tasks, shell tool activations
- MCP servers configured locally
- Presence of other AI dev tools: Cursor, Windsurf, Codeium, Cowork, Zed, Aider, ChatGPT Desktop, Continue, Goose, Junie, Trae, Crush

**Every connector you have live** (Claude skips any you don't)
- Granola (meetings)
- Gmail (threads, drafts, labels)
- Google Calendar (events, calendars)
- Slack (channels, scheduled messages)
- Google Drive (recent files)
- Notion (pages, workspaces)
- HubSpot (deals, contacts)
- Atlassian / Jira / Confluence (issues, pages)
- Typefully (drafts, queued, published)
- iMessage, Spotify, ElevenLabs, PDF Tools, Word, Excel, PowerPoint, Firecrawl, Control Chrome, MCP Registry
- Any other MCP server you've connected

**Last 7 days of git**
- Commits you authored across your work directories

**What Claude never reads**: message content, email bodies, meeting transcripts, file contents, credentials, browser history, or chat history with any AI.

---

## Recipient

When the skill gets to the Slack step, it asks who to draft the message to. Three options:

1. **Hit enter** to use the default recipient (a Slack user ID baked into the skill for the original Collaboration.ai all-hands).
2. **Type a Slack user ID** (looks like `U1ABC2DEF3`).
3. **Type a teammate's name** and the skill resolves it via Slack search.

If you're running this for your own team's all-hands, swap in a teammate's ID or just type their name.

---

## What the stats card looks like (illustrative)

The example uses superhero names because the real skill pulls actual names and counts from your machine at runtime.

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

*Connectors live (7 of 14)*
• Granola: 28 meetings 7d, 312 all-time
• Gmail: 89 threads 7d, 4 drafts
• Calendar: 14 events 7d, 11 next 7d
• Slack: 23 channels
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

---

## Troubleshooting

**"Claude said it couldn't fetch from GitHub."**
The repo might be unreachable or temporarily blocked. Try the raw URLs directly or download the zip from the GitHub releases page and drop the folder into your Claude skills directory.

**"Slack draft tool isn't available."**
Connect Slack in your Claude settings (Connectors → Slack → Connect). Rerun. If still blocked, the skill falls back to printing the stats card so you can copy-paste.

**"Wrong recipient showed up."**
The skill asks at runtime. Just re-answer when it prompts. Or open the installed SKILL.md and change the default Slack ID.

**"The scan is hanging on git."**
Git scan caps at 15 seconds. If your work directories are huge, it will report partial. That's fine.

**"It sent without asking."**
It shouldn't. The skill uses the Slack draft tool, not the send tool. If you ever see an auto-sent message, file an issue on the repo and we'll patch.

---

## License and intent

MIT. Use it, fork it, adapt it for your own all-hands event. The skill is designed to be a one-shot snapshot for fun, not a surveillance or adoption-tracking tool. Please don't turn it into one.
