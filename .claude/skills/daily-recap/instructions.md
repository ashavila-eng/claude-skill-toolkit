---
name: daily-recap-instructions
description: Instructions for the remote daily recap agent. Executed by the scheduled trigger.
---

Produce a daily standup recap for ash.avila and send it as a Slack DM.

## Step 1 — Establish yesterday's date

Run `date` in Bash to get today's date. Yesterday = today minus 1 calendar day.
If today is Monday, use last Friday instead (skip the weekend).
Store the result as YESTERDAY in YYYY-MM-DD format for all queries below.

## Step 2 — Resolve the Jira account ID

Using the Atlassian MCP, call `lookupJiraAccountId` with the username
`ash.avila@get-carrot.com` to get the exact Jira account ID. Store this as
ACCOUNT_ID — use it in all Jira and Confluence queries below.

## Step 3 — Jira activity

Using the Atlassian MCP, search Jira with this JQL (substitute real values):

  updated >= 'YESTERDAY' AND (assignee = "ACCOUNT_ID" OR comment ~ "ash.avila@get-carrot.com")

Limit to 20 results. For each issue record: key, summary, current status, and
whether ash.avila is the assignee or just a commenter.

## Step 4 — Confluence updates

Using the Atlassian MCP, search Confluence with this CQL (substitute real values):

  contributor = "ACCOUNT_ID" AND lastModified >= 'YESTERDAY'

Limit to 10 results. For each page note: title, space name, and what
meaningfully changed. Skip pages where the only change was an automated macro
refresh or Jira embed update.

## Step 5 — Write the recap

Synthesize everything into this format:

---
### Daily Recap — [Weekday, Month DD]

**What shipped / merged**
- [Jira tickets moved to Done / Closed / Resolved]

**What was in progress**
- [Jira tickets moved to In Progress, or updated with substantive work]

**Reviews / collaboration**
- [Jira issues commented on where ash.avila is not the assignee]

**Docs updated**
- [Confluence pages meaningfully edited — page title (Space name), what changed]
---

Rules:
- Lead with outcomes: "Shipped X" not "Worked on X"
- Omit any section that has nothing to report
- If there is truly nothing across all sources, say so plainly in one line
- Keep each bullet to one line
- Skip noise: automated status syncs, minor formatting edits

## Step 6 — Send the recap via Slack DM

Using the Slack MCP, search for the user with display name or username
`ash.avila` and send the recap as a direct message. Do not post to any
channel — DM only.
