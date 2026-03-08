# Claude Loop Plugin

A Claude Code plugin that installs `claude-loop` - a wrapper script that automatically retries when hitting rate limits.

## Features

- **Auto-retry on rate limits**: Detects usage limits and waits before resuming
- **Session resume**: Continues from the last session automatically
- **Configurable**: Adjust retry delay and max attempts via environment variables
- **Visual feedback**: Countdown timer and colored status messages

## Installation

### Via Claude Code Plugin Marketplace

```bash
/plugin marketplace add kimjunghyun/claude-loop-plugin
/plugin install claude-loop
```

Then run the skill:
```bash
/install-claude-loop
```

### Manual Installation

1. Clone this repository
2. Copy `skills/install-claude-loop/SKILL.md` to `~/.claude/skills/install-claude-loop/`

## Usage

After installation:

```bash
# Reload shell config
source ~/.zshrc  # or ~/.bashrc

# Run with auto-retry
claude-loop "implement user authentication"
```

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `CLAUDE_LOOP_DELAY` | 300 | Seconds to wait before retry |
| `CLAUDE_LOOP_MAX_RETRIES` | 100 | Maximum retry attempts |
| `CLAUDE_LOOP_DEBUG` | 0 | Set to 1 to see JSON responses |

Example:
```bash
CLAUDE_LOOP_DELAY=180 claude-loop "fix all TypeScript errors"
```

## How It Works

1. Runs Claude Code with your prompt
2. Parses JSON output to detect rate limit errors
3. If limited: waits, then resumes the session
4. Continues until task completes or max retries reached

## License

MIT
