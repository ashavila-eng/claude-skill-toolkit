---
name: daily-recap
description: >
  Fires the remote daily recap trigger, which queries Jira, Confluence, and
  sends a standup summary to ash.avila on Slack. Use this when the user asks
  for a daily recap, standup summary, or wants to know what was done yesterday.
  Trigger on phrases like "recap yesterday", "daily recap", "standup summary",
  "run the daily recap", or "fire the recap".
---

Fire the remote daily recap trigger using the RemoteTrigger tool.

## Step 1 — Load the tool

Use ToolSearch with query "select:RemoteTrigger" to load the RemoteTrigger tool
schema before calling it.

## Step 2 — Run the trigger

Call RemoteTrigger with:
- action: "run"
- trigger_id: "trig_01DX2KtSWngvtGfy9MwK6gPh"

## Step 3 — Confirm

Tell the user the trigger is running. The recap will arrive as a Slack DM to
ash.avila once the remote agent finishes (usually within a minute or two).
