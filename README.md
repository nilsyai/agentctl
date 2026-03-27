# agentctl

A lightweight CLI for managing AI agent tasks and state.

`agentctl` provides a simple task queue, state management system, and event logging for autonomous agents. It uses SQLite for storage, requires no external dependencies beyond Python 3.8+, and is designed to be embedded in agent workflows.

## Features

- **Task Management**: Create, list, update, and delete tasks
- **Status Tracking**: Track task states (pending, in_progress, completed, blocked)
- **Priority Levels**: Assign low/medium/high priority to tasks
- **Task Tagging**: Tag tasks with comma-separated labels and filter or summarise by tag
- **Due Dates**: Set deadlines with relative (+1h, +2d) or absolute (YYYY-MM-DD) formats
- **Overdue Detection**: Filter overdue tasks; visual indicators (🔴/🟡) in list output
- **Per-task Notes**: Append timestamped progress notes without overwriting descriptions
- **State Storage**: Key-value state storage for agent persistence
- **Checkpoints**: Export/restore full state to portable JSON - survive context resets, migrate between machines
- **Purge**: Clean wipe of tasks, events, state, notes, or everything
- **Event Logging**: Log structured events with tags and JSON data payloads
- **Reports**: Generate time-window summaries of tasks and events
- **Zero Dependencies**: Uses only Python standard library + SQLite
- **Simple CLI**: Intuitive commands with emoji-enhanced output

## Installation

```bash
# Clone the repository
git clone https://github.com/nilsyai/agentctl.git
cd agentctl

# Make executable
chmod +x agentctl

# Optional: Add to PATH
sudo ln -s $(pwd)/agentctl /usr/local/bin/agentctl
```

Or use directly without installing:

```bash
./agentctl --help
```

## Quick Start

```bash
# Add a task
agentctl add "Review pull requests" -d "Check and review pending PRs" -p high

# Add a task with tags
agentctl add "Deploy v2.0" -p high --tags "deploy,urgent"

# Add a task with a due date (relative)
agentctl add "Fix login bug" -p high --due +4h

# Add a task with a due date (absolute)
agentctl add "Quarterly review" -p medium --due 2025-06-30

# List all tasks
agentctl list

# List only overdue tasks
agentctl list --overdue

# Filter tasks by tag
agentctl list --tag deploy

# Show all tags with counts
agentctl tags

# Update task status
agentctl update 1 -s in_progress

# Set a due date on an existing task
agentctl update 1 --due +2d

# Remove a due date
agentctl update 1 --clear-due

# Append a progress note to a task
agentctl note 1 "Completed auth refactor, testing next"

# Show task details (includes notes)
agentctl show 1

# Mark as completed
agentctl update 1 -s completed

# Log an event
agentctl log "Processed 42 records" --tag sync --data '{"count":42}'

# Show recent events
agentctl logs --since 1h

# Generate a report for the last 24 hours
agentctl report
```

## Usage Examples

### Task Management

```bash
# Add tasks with full options
agentctl add "Deploy API" -d "Deploy to production" -p high --tags "deploy,prod" --due +2d

# List high-priority tasks
agentctl list -s pending

# Show full task details with notes
agentctl show 3
```

### Due Dates

Due dates support three input formats:

```bash
# Relative (from now)
agentctl add "Quick fix" --due +30m    # 30 minutes
agentctl add "Code review" --due +4h   # 4 hours
agentctl add "Sprint task" --due +3d   # 3 days

# Date only (treated as end of day 23:59:59)
agentctl add "Quarterly report" --due 2025-06-30

# Date + time
agentctl add "Team standup prep" --due 2025-06-30T09:00
```

In list output, due dates show with visual indicators:
- 🔴 = overdue (past due date)
- 🟡 = due within 24 hours
- Plain date = more than 24 hours away

### Per-task Notes

Notes let agents (or humans) append progress updates without overwriting the task description:

```bash
# Append notes as work progresses
agentctl note 5 "Started investigation - issue in auth middleware"
agentctl note 5 "Root cause found: token expiry not handled"
agentctl note 5 "Fix deployed to staging, waiting for QA sign-off"

# All notes appear in show output
agentctl show 5
```

### Checkpoint & Restore

```bash
# Export everything (tasks, state, events, notes) to a JSON file
agentctl checkpoint -o my-backup.json

# Restore from checkpoint (replaces existing data)
agentctl restore my-backup.json

# Merge checkpoint into existing data
agentctl restore my-backup.json --merge
```

### State Management

```bash
# Store key-value state for the agent
agentctl state set last_run "2025-06-01T14:00:00"
agentctl state set current_phase "ingestion"

# Retrieve state
agentctl state get last_run
```

### Reports

```bash
# Last 24 hours (default)
agentctl report

# Last 7 days
agentctl report --since 7d

# Last hour
agentctl report --since 1h
```

## Commands Reference

| Command | Description |
|---------|-------------|
| `add` | Add a new task (`--due`, `--tags`, `-p`, `-d`) |
| `list` | List tasks (`--overdue`, `--tag`, `-s`, `-n`) |
| `show <id>` | Show task details + notes |
| `update <id>` | Update task fields (`--due`, `--clear-due`, `--tags`, `-s`, `-p`) |
| `note <id> <text>` | Append a timestamped note to a task |
| `delete <id>` | Delete a task (and its notes) |
| `status` | Show task statistics (incl. overdue count) |
| `tags` | List all tags with counts |
| `state set/get` | Key-value state storage |
| `log` | Log a structured event |
| `logs` | List recent events |
| `report` | Time-windowed summary of tasks + events |
| `checkpoint` | Export all data to JSON |
| `restore` | Restore from checkpoint JSON |
| `purge` | Delete data (tasks/events/state/notes/all) |

## Database

By default, agentctl stores data at `~/.agentctl/tasks.db`. Override with `--db`:

```bash
agentctl --db /path/to/custom.db list
```

`agentctl` automatically migrates existing databases when new columns are added - no manual migration needed.

## Changelog

### v0.5.0
- **Due dates**: `--due` flag on `add`/`update` with relative (+1h/+2d/+30m) and absolute (YYYY-MM-DD/YYYY-MM-DDTHH:MM) formats
- **Overdue filter**: `agentctl list --overdue` shows tasks past their due date
- **Visual due indicators**: 🔴 (overdue) and 🟡 (due within 24h) in list output
- **Note command**: `agentctl note <id> <text>` appends timestamped progress notes
- **Notes in show**: Notes displayed in `agentctl show` output
- **Notes in checkpoint/restore**: Notes included in full-state export
- **Overdue section in report**: Report now lists overdue tasks separately
- **Overdue count in status**: `agentctl status` shows overdue count when non-zero
- **`--clear-due` flag**: Remove a due date from an existing task
- Auto-migration: `due_at` column and `task_notes` table added to existing DBs on startup

### v0.4.0
- Tags: `--tags` flag on `add`/`update`, `agentctl tags` command, `--tag` filter on `list`

### v0.3.0
- Checkpoint and restore: full-state export/import to portable JSON

### v0.2.0
- Event logging: `log`, `logs` commands with tag and time filters

### v0.1.0
- Initial release: task management, state storage, reports
