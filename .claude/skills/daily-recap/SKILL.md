---
name: daily-recap
description: >
  Fires the remote daily recap trigger, which queries Jira, Confluence, and
  sends a standup summary to ash.avila on Slack. Use this when the user asks
  for a daily recap, standup summary, or wants to know what was done yesterday.
  Trigger on phrases like "recap yesterday", "daily recap", "standup summary",
  "run the daily recap", or "fire the recap".
---

Gather local Claude Code session stats, then fire the remote recap trigger
with that data included.

## Step 1 — Compute yesterday's date

Run this in Bash:
```bash
python3 -c "
from datetime import datetime, timedelta
today = datetime.now()
yesterday = today - timedelta(days=1)
if today.weekday() == 0:  # Monday
    yesterday = today - timedelta(days=3)  # Friday
print(yesterday.strftime('%Y-%m-%d'))
"
```

Store the result as YESTERDAY.

## Step 2 — Gather Claude Code session stats

Run this Bash script to parse session data from `~/.claude/projects/`:

```bash
python3 -c "
import os, json, glob
from datetime import datetime, timedelta

today = datetime.now()
yesterday = today - timedelta(days=1)
if today.weekday() == 0:
    yesterday = today - timedelta(days=3)
yesterday_str = yesterday.strftime('%Y-%m-%d')

projects_dir = os.path.expanduser('~/.claude/projects')
project_stats = {}

for project_dir in os.listdir(projects_dir):
    full_dir = os.path.join(projects_dir, project_dir)
    if not os.path.isdir(full_dir):
        continue
    for f in glob.glob(os.path.join(full_dir, '*.jsonl')):
        mod_date = datetime.fromtimestamp(os.path.getmtime(f)).strftime('%Y-%m-%d')
        if mod_date != yesterday_str:
            continue
        timestamps = []
        with open(f) as fh:
            for line in fh:
                try:
                    entry = json.loads(line)
                    if 'timestamp' in entry:
                        timestamps.append(entry['timestamp'])
                except:
                    pass
        if len(timestamps) >= 2:
            first = datetime.fromisoformat(timestamps[0].replace('Z', '+00:00'))
            last = datetime.fromisoformat(timestamps[-1].replace('Z', '+00:00'))
            duration_mins = max(1, int((last - first).total_seconds() / 60))
            name = project_dir
            name = name.replace('-Users-ash-avila-workspace-', '')
            name = name.replace('-Users-ash-avila-', '')
            parts = name.split('--claude-worktrees-')
            if len(parts) == 2:
                name = parts[0].replace('-', '/') + ' (worktree)'
            else:
                name = name.replace('-', '/')
            project_stats[name] = project_stats.get(name, 0) + duration_mins

if project_stats:
    lines = ['Claude Code time yesterday:']
    for proj, mins in sorted(project_stats.items(), key=lambda x: -x[1]):
        h, m = divmod(mins, 60)
        dur = f'{h}h {m}m' if h > 0 else f'{m}m'
        lines.append(f'- {proj}: {dur}')
    print('\n'.join(lines))
else:
    print('No Claude Code sessions found for yesterday.')
"
```

Store the full output as CLAUDE_STATS.

## Step 3 — Load RemoteTrigger

Use ToolSearch with query "select:RemoteTrigger" to load the RemoteTrigger
tool schema.

## Step 4 — Fire the trigger with Claude stats

Call RemoteTrigger with:
- action: "run"
- trigger_id: "trig_01DX2KtSWngvtGfy9MwK6gPh"
- body: `{"claude_stats": "<value of CLAUDE_STATS>"}`

## Step 5 — Confirm

Tell the user the trigger is running. The recap will arrive as a Slack DM to
ash.avila once the remote agent finishes (usually within a minute or two).
