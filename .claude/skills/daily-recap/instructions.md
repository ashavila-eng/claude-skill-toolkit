---
name: daily-recap
description: >
  Recaps the previous day's work by pulling together git commits, GitHub PRs,
  Jira activity, and Confluence page updates from yesterday. Use this when the
  user asks for a daily standup summary, wants to know what was done yesterday,
  or needs a recap before a standup meeting. Trigger on phrases like "recap
  yesterday", "what did I do yesterday", "daily recap", "standup summary", or
  "what did we ship yesterday".
---

Produce a concise, narrative recap of yesterday's engineering work. Pull from
every available signal — git, GitHub, Jira, Confluence — and synthesize them
into a human-readable summary the user can paste into a standup or send to a
manager.

## Step 1 — Establish "yesterday"

Today's date is available in context as `currentDate`. Yesterday = today minus
one calendar day. Format it as `YYYY-MM-DD` for use in queries.

If today is a Monday, yesterday is the previous Friday (skip the weekend).

## Step 2 — Git commits

In the current working directory, run:

```
git log --oneline --all --since="YESTERDAY 00:00" --until="YESTERDAY 23:59" \
  --author="$(git config user.email)" --format="%h %s (%D)"
```

If the cwd is not a git repo, skip silently.

Also check if the user has a `~/workspace` or `~/code` or `~/dev` directory —
if it exists, iterate one level deep and run the same `git log` command in any
subdirectory that is a git repo. Collect all results.

Deduplicate commits by hash. Group by repo name (the directory name).

## Step 3 — GitHub PRs

Run these `gh` commands (silently skip if `gh` is not authenticated):

```bash
# PRs opened yesterday
gh search prs --author=@me --created="YESTERDAY" --json title,url,state,repository

# PRs merged yesterday
gh search prs --author=@me --merged="YESTERDAY" --json title,url,state,repository

# PRs reviewed yesterday (comments/reviews left)
gh search prs --reviewed-by=@me --updated="YESTERDAY" --json title,url,state,repository,author
```

Deduplicate by URL. Tag each with its status: Opened, Merged, or Reviewed.

## Step 4 — Jira issues (if available)

If the Atlassian MCP tools are available, run:
```
searchJiraIssuesUsingJql: "updated >= 'YESTERDAY' AND updated <= 'YESTERDAY 23:59' AND (assignee = currentUser() OR comment ~ currentUser())"
```

Limit to 20 results. For each issue, note: key, summary, status.

If Atlassian MCP is not available, skip this step silently.

## Step 5 — Confluence updates (if available)

If the Atlassian MCP tools are available, search for Confluence pages edited
yesterday:

```
searchConfluenceUsingCql: "contributor = currentUser() AND lastModified >= 'YESTERDAY' AND lastModified <= 'YESTERDAY 23:59'"
```

Limit to 10 results. For each page, fetch it with `getConfluencePage` and note:
- Page title and space name
- What section(s) changed (scan the version history comment if present, or
  compare headings against the page body to infer what was touched)
- Whether it was a minor edit (typo fix, formatting) or a meaningful update
  (new content, status change, implementation notes added)

Skip pages where the only change was an automated update (e.g. status macros,
Jira issue embeds refreshing). Focus on pages the user actively wrote.

If Atlassian MCP is not available, skip this step silently.

## Step 6 — Synthesize the recap

Combine all signals into a structured summary. Use this format:

---

### Daily Recap — [YESTERDAY formatted as "Monday, April 21"]

**What shipped / merged**
- [List PRs merged or commits that look like features/fixes — be specific about what changed]

**What was in progress**
- [PRs opened but not yet merged, Jira tickets moved to In Progress]

**Reviews / collaboration**
- [PRs reviewed for others, comments left on Jira tickets]

**Docs updated**
- [Confluence pages meaningfully edited — page title, space, and what changed]

**Key commits** _(if not already covered above)_
- `repo-name`: Short description of what changed

---

Rules for the narrative:
- Lead with outcomes, not activity ("Shipped X" not "Worked on X")
- Group related commits under one bullet if they form a logical unit
- Skip noise: merge commits, dependency bumps, lint-only changes, version bumps
- If a commit message is cryptic, infer meaning from the diff summary if available
- Keep each bullet to one line
- If there's nothing from a section (e.g. no PRs), omit that section entirely
- If there's truly nothing to report across all sources, say so plainly

## Step 6 — Surface blockers and follow-ups

After the main recap, add a short **Blockers / Notes** section only if you
spot signals of incomplete work:
- PRs open for more than 1 day with no review
- Jira tickets still In Progress with no recent commit activity
- Commits referencing "WIP", "TODO", "FIXME", or "broken"

Keep this section to 3 items max. If nothing notable, omit it entirely.
