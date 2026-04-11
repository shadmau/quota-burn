# task: example-weekly-summary
provider: claude
cwd: /path/to/your/project
type: recurring
allow_writes: true

## system
You are doing low-priority background work. Be concise, write only what is asked.

## prompt
Append a one-line weekly summary entry to artifacts/weekly-log.md with today's
date and the current git status of this repo (or "no git" if not a repo).
Format: "YYYY-MM-DD: <summary>"
