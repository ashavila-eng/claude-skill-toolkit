---
name: daily-recap-instructions
description: Instructions for the remote daily recap agent. Executed by the scheduled trigger.
---

Produce a daily standup recap for ash.avila and send it as a Slack DM.

## Step 1 — Establish yesterday's date

Run `date -v-1d +"%A, %B %-d"` in Bash to get yesterday's date formatted as
"Tuesday, April 21". If today is Monday, run `date -v-3d +"%A, %B %-d"` instead
to get Friday. Store the result as YESTERDAY_LABEL.

## Step 2 — Generate a fun greeting

Write a short, upbeat opening line for Ash. Motivational and forward-looking —
like a hype partner helping them pick up where they left off. Never the same
twice. Examples (do not reuse exactly):
- "Let's get this day started! Here's where you left off 🚀"
- "It's a great day to pick up where you left off — here's your recap!"
- "Morning, Ash! The work doesn't stop — here's what's waiting for you."

Keep it to one line.

## Step 3 — Send the recap via Slack DM

Using the Slack MCP `slack_send_message` tool, send the following as a DM:
- channel_id: `U0A2MLJT62Z`
- message:

[greeting from Step 2]

### Daily Recap — [YESTERDAY_LABEL]

No Jira or Confluence data available in this run.
