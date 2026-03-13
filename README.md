# agentctl

A lightweight CLI for managing AI agent tasks and state.

`agentctl` provides a simple task queue, state management system, and event logging for autonomous agents. It uses SQLite for storage, requires no external dependencies beyond Python 3.8+, and is designed to be embedded in agent workflows.

## Features

- **Task Management**: Create, list, update, and delete tasks
- **Status Tracking**: Track task states (pending, in_progress, completed, blocked)
- **Priority Levels**: Assign low/medium/high priority to tasks
- **State Storage**: Key-value state storage for agent persistence
- **Checkpoints**: Export/restore full state to portable JSON - survive context resets, migrate between machines
- **Purge**: Clean wipe of tasks, events, state, or everything
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

# List all tasks
agentctl list

# Update task status
agentctl update 1 -s in_progress

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
# Add tasks with different priorities
agentctl add "Deploy to production" -p high
agentctl add "Update documentation" -p low -d "Add examples to README"

# List pending tasks only
agentctl list -s pending

# Show task details
agentctl show 1

# Update task title and priority
agentctl update 1 -t "Deploy v2.0 to production" -p high

# Delete a task
agentctl delete 1
```

### State Management

```bash
# Store state values
agentctl state set last_run "2026-02-27T15:00:00"
agentctl state set current_project "agentctl"

# Retrieve state values
agentctl state get last_run
# Output: 2026-02-27T15:00:00

# Use in scripts
LAST_RUN=$(agentctl state get last_run)
echo "Last run: $LAST_RUN"
```

### Event Logging

```bash
# Log a plain event
agentctl log "Agent started"

# Log with a tag (useful for filtering later)
agentctl log "Fetched 100 records" --tag sync

# Log with structured JSON data
agentctl log "Rate limit hit" --tag error --data '{"code":429,"retry_after":60}'

# List all events
agentctl logs

# Filter by tag
agentctl logs --tag error

# Show events from the last hour
agentctl logs --since 1h

# Show events from the last 7 days, including data payloads
agentctl logs --since 7d --data
```

Duration formats for `--since`: `30m` (minutes), `1h` (hours), `7d` (days).

### Reports

```bash
# Default: last 24 hours
agentctl report

# Custom window
agentctl report --since 7d
agentctl report --since 1h
```

A report shows:
- Task totals (all time) and tasks completed within the window
- Event counts by tag within the window
- The 10 most recent events

### Integration in Agent Workflows

```bash
#!/bin/bash
# Example: Daily agent workflow with event logging

# Check if already running
if [ "$(agentctl state get daily_run_status)" = "running" ]; then
    agentctl log "Skipped: daily run already in progress" --tag cron
    exit 0
fi

agentctl state set daily_run_status "running"
agentctl log "Daily run started" --tag cron

# Process pending tasks
for task_id in $(agentctl list -s pending --limit 10 | tail -n +3 | awk '{print $1}'); do
    agentctl update "$task_id" -s in_progress

    if process_task "$task_id"; then
        agentctl update "$task_id" -s completed
        agentctl log "Task $task_id completed" --tag cron
    else
        agentctl update "$task_id" -s blocked
        agentctl log "Task $task_id blocked" --tag error
    fi
done

agentctl state set daily_run_status "idle"
agentctl state set last_run "$(date -Iseconds)"
agentctl log "Daily run finished" --tag cron

# Print a summary
agentctl report --since 1h
```

## Commands

| Command | Description |
|---------|-------------|
| `add` | Create a new task |
| `list` | List tasks with optional filters |
| `show` | Display detailed task information |
| `update` | Modify task properties |
| `delete` | Remove a task |
| `status` | Show task statistics |
| `state set` | Store a state value |
| `state get` | Retrieve a state value |
| `log` | Log a structured event |
| `logs` | List recent events with optional filters |
| `report` | Generate a summary report of tasks and events |
| `checkpoint` | Export full state to a portable JSON file |
| `restore` | Restore state from a checkpoint file |
| `purge` | Delete all data (or specific tables) |

## Options

| Flag | Description |
|------|-------------|
| `-d, --description` | Task description |
| `-p, --priority` | Priority: low, medium, high |
| `-s, --status` | Status: pending, in_progress, completed, blocked |
| `-m, --metadata` | JSON metadata string |
| `-n, --limit` | Limit number of results |
| `--db` | Custom database path |
| `--tag` | Tag for event log/filter |
| `--data` | JSON data payload (log) or show payloads flag (logs) |
| `--since` | Duration window: 30m, 1h, 24h, 7d |
| `-o, --output` | Checkpoint output file path |
| `--merge` | Merge restored data instead of replacing |
| `-y, --yes` | Skip purge confirmation |

### Checkpoints & Recovery

```bash
# Save a checkpoint before risky operations
agentctl checkpoint -o backup.json

# Restore from checkpoint (replaces current data)
agentctl restore backup.json

# Merge checkpoint into existing data
agentctl restore backup.json --merge

# Wipe everything
agentctl purge -y

# Wipe only events
agentctl purge events -y
```

Checkpoints are portable JSON - move them between machines, back them up, diff them in git.
Useful for agent continuity across context resets or when migrating between environments.

## Database

By default, data is stored in `~/.agentctl/tasks.db`. Override with `--db` flag:

```bash
agentctl --db /path/to/custom.db status
```

The database has three tables: `tasks`, `state`, and `events`.

## Requirements

- Python 3.8+
- No external dependencies

## License

MIT License - see [LICENSE](LICENSE) file.

## Contributing

Contributions welcome! Please open an issue or PR on GitHub.
