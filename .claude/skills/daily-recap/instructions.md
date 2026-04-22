---
name: daily-recap-instructions
description: Instructions for the remote daily recap agent. Executed by the scheduled trigger.
---

Produce a daily standup recap for ash.avila and send it as a Slack DM.

## Step 1 — Establish yesterday's date

Run `date` in Bash to get today's date. Yesterday = today minus 1 calendar day.
If today is Monday, use last Friday instead (skip the weekend).
Store the result as YESTERDAY in YYYY-MM-DD format for all queries below.

## Step 2 — Generate a fun greeting

Write a short, upbeat opening line for Ash. The vibe is: motivational and
forward-looking — like a hype partner helping them pick up where they left off.
Keep it varied, never the same twice. Examples of the right tone (do not reuse
these exactly):
- "Let's get this day started! Here's where you left off 🚀"
- "It's a great day to pick up where you left off — here's your recap!"
- "Morning, Ash! The work doesn't stop — here's what's waiting for you."
- "Another day, another chance to ship. Here's where things stand!"
- "Rise and pick up — here's everything you left on the table yesterday."

Keep it to one line. Place it at the very top of the Slack message.

## Step 3 — Resolve the Jira account ID

Using the Atlassian MCP, call `lookupJiraAccountId` with the username
`ash.avila@get-carrot.com` to get the exact Jira account ID. Store this as
ACCOUNT_ID — use it in all Jira and Confluence queries below.

## Step 4 — Jira activity

Using the Atlassian MCP, search Jira with this JQL (substitute real values):

  updated >= 'YESTERDAY' AND (assignee = "ACCOUNT_ID" OR comment ~ "ash.avila@get-carrot.com")

Limit to 20 results. For each issue record: key, summary, current status, and
whether ash.avila is the assignee or just a commenter.

## Step 5 — Confluence updates

Using the Atlassian MCP, search Confluence with this CQL (substitute real values):

  contributor = "ACCOUNT_ID" AND lastModified >= 'YESTERDAY'

Limit to 10 results. For each page note: title, space name, and what
meaningfully changed. Skip pages where the only change was an automated macro
refresh or Jira embed update.

## Step 6 — Check for Claude Code stats

Look in the current session context for a `claude_stats` value passed via the
run body. If present, include it verbatim in the recap under a "Claude Code"
section. If absent, omit that section.

## Step 7 — Write the recap

Synthesize everything into this format:

---
[fun greeting from Step 2]

### Daily Recap — [Weekday, Month DD]

**What shipped / merged**
- [Jira tickets moved to Done / Closed / Resolved]

**What was in progress**
- [Jira tickets moved to In Progress, or updated with substantive work]

**Reviews / collaboration**
- [Jira issues commented on where ash.avila is not the assignee]

**Docs updated**
- [Confluence pages meaningfully edited — page title (Space name), what changed]

**Claude Code**
- [Time per project from claude_stats, if available]
---

Rules:
- Lead with outcomes: "Shipped X" not "Worked on X"
- Omit any section that has nothing to report
- If there is truly nothing across all sources, say so plainly in one line
- Keep each bullet to one line
- Skip noise: automated status syncs, minor formatting edits

## Step 8 — Send the recap via Slack DM

Using the Slack MCP, search for the user with display name or username
`ash.avila` and send the recap as a direct message. Do not post to any
channel — DM only.
