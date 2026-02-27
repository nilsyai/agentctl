# agentctl

A lightweight CLI for managing AI agent tasks and state.

`agentctl` provides a simple task queue and state management system for autonomous agents. It uses SQLite for storage, requires no external dependencies beyond Python 3.8+, and is designed to be embedded in agent workflows.

## Features

- **Task Management**: Create, list, update, and delete tasks
- **Status Tracking**: Track task states (pending, in_progress, completed, blocked)
- **Priority Levels**: Assign low/medium/high priority to tasks
- **State Storage**: Key-value state storage for agent persistence
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

# Check overall status
agentctl status
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

### Integration in Agent Workflows

```bash
#!/bin/bash
# Example: Daily agent workflow

# Check if already running
if [ "$(agentctl state get daily_run_status)" = "running" ]; then
    echo "Daily run already in progress"
    exit 0
fi

agentctl state set daily_run_status "running"

# Process pending tasks
for task_id in $(agentctl list -s pending --limit 10 | tail -n +3 | awk '{print $1}'); do
    agentctl update "$task_id" -s in_progress
    
    # Do the work...
    if process_task "$task_id"; then
        agentctl update "$task_id" -s completed
    else
        agentctl update "$task_id" -s blocked
    fi
done

agentctl state set daily_run_status "idle"
agentctl state set last_run "$(date -Iseconds)"
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

## Options

| Flag | Description |
|------|-------------|
| `-d, --description` | Task description |
| `-p, --priority` | Priority: low, medium, high |
| `-s, --status` | Status: pending, in_progress, completed, blocked |
| `-m, --metadata` | JSON metadata string |
| `-n, --limit` | Limit number of results |
| `--db` | Custom database path |

## Database

By default, data is stored in `~/.agentctl/tasks.db`. Override with `--db` flag:

```bash
agentctl --db /path/to/custom.db status
```

## Requirements

- Python 3.8+
- No external dependencies

## License

MIT License - see [LICENSE](LICENSE) file.

## Contributing

Contributions welcome! Please open an issue or PR on GitHub.
