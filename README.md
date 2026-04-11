# quota-burn

Automatically runs low-priority backlog tasks when your AI quota is about to expire.

Monitors the weekly usage windows for Claude and Codex. When the window is closing (< 60 min to reset) and you've been idle for 10+ minutes, it runs queued tasks back-to-back — burning quota that would otherwise be wasted.

## How it works

```
cron (every 10 min)
  └── quota-burn run-once
        ├── fetch rate limits from API  (Claude: Anthropic API headers, Codex: JSONL from CLI)
        ├── check idle (JSONL timestamps)
        ├── policy: reset < 60 min AND idle > 10 min?
        └── run up to max_tasks_per_run tasks sequentially
```

Reset times come directly from the provider APIs — rolling windows, matches exactly what you see in the web UI.

## Setup

**1. Install**
```bash
git clone <repo>
chmod +x quota-burn
```

**2. Configure** (`config.toml`)
```toml
[policy]
reset_threshold_minutes = 60   # trigger window before reset
idle_threshold_minutes  = 10   # skip if you've been active recently
max_tasks_per_run       = 3    # tasks per cron invocation

[providers.claude]
binary        = "claude"
refresh_model = "claude-haiku-4-5-20251001"

[providers.codex]
binary        = "codex"
refresh_model = "gpt-5.4-mini"
```

**3. Add tasks** (`backlog/my-task.md`)
```markdown
# task: add-docstrings
provider: claude
cwd: /path/to/your/project
type: once
model: claude-haiku-4-5-20251001

## system
Do low-priority background work only. Do not change logic.

## prompt
Add missing docstrings to all public functions in src/auth.py.
```

**4. Set up cron**
```bash
crontab -e
# add:
*/10 * * * * /path/to/quota-burn run-once >> ~/.quota-burn/cron.log 2>&1
```

## Task types

| Type | Behavior |
|---|---|
| `once` | Runs once, then done. Reset with `quota-burn reset <name>` |
| `recurring` | Runs every eligible window |
| `loop` | Runs up to `max_passes` times; each pass gets iteration context injected |

## Commands

```bash
quota-burn check          # show provider status + eligible tasks
quota-burn run-once       # run up to max_tasks_per_run tasks now
quota-burn done           # list completed tasks and loop progress
quota-burn reset <name>   # re-queue a once/loop task
```

## Provider support

**Claude** — reads idle from `~/.claude/projects/**/*.jsonl`, fetches reset time directly from `api.anthropic.com` rate limit headers. Requires Claude CLI (`claude`).

**Codex** — reads idle and reset time from `~/.codex/sessions/**/*.jsonl` (written by the Codex CLI after each call). If data is stale (Cursor extension users), fires a minimal refresh ping. Requires Codex CLI (`codex`).

## Idle detection

Idle is checked per provider. A hard cooldown also prevents back-to-back runs if quota-burn itself was active recently (its own JSONL writes would otherwise reset the idle clock).

```
effective_idle = provider idle AND time since our last task > threshold
```

## File structure

```
quota-burn          # single Python script, no dependencies
config.toml         # policy + provider settings
backlog/            # task files (.md)
state.json          # runtime state (gitignored)
artifacts/          # output files written by tasks (gitignored if desired)
```

## Requirements

- Python 3.8+
- `claude` CLI (for Claude tasks)
- `codex` CLI (for Codex tasks)
- Linux/macOS (uses `fcntl` for locking)
