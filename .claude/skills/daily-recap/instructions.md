---
name: daily-recap-instructions
description: Instructions for the remote daily recap agent. Executed by the scheduled trigger.
---

Send a test Slack DM to verify the trigger is working end-to-end.

## Step 1 — Send a Slack DM

Using the Slack MCP `slack_send_message` tool, send the following message:
- channel_id: `U0A2MLJT62Z`
- message: "Daily recap trigger test — if you're seeing this, Slack is wired up correctly!"
