[English](README.md) | [한국어](README.ko.md)

# Claude Loop

A wrapper script that automatically retries when Claude Code hits usage limits.

## What is this?

`claude-loop` is a **standalone shell script**. It doesn't run inside Claude Code — it **wraps Claude Code and runs from your terminal**.

```
Normal usage:         claude "prompt"           → stops when usage limit hit
With claude-loop:     claude-loop "prompt"     → waits and auto-resumes
```

## Installation

### Option 1: Via Claude Code Skill (Recommended)

```bash
# Inside Claude Code
/plugin marketplace add baerae-zkap/claude-loop-plugin
/plugin install claude-loop
/claude-loop:install
```

What the skill does:
1. Creates `~/.local/bin/claude-loop` script
2. Adds `~/.local/bin` to PATH

### Option 2: Manual Installation

```bash
# Download script
mkdir -p ~/.local/bin
curl -o ~/.local/bin/claude-loop https://raw.githubusercontent.com/baerae-zkap/claude-loop-plugin/main/scripts/claude-loop.sh
chmod +x ~/.local/bin/claude-loop

# Add to PATH (~/.zshrc or ~/.bashrc)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## Usage

**Important: Run from terminal, outside of Claude Code.**

```bash
# From terminal
claude-loop "implement user authentication"
```

### How it works

```
1. claude-loop runs claude
2. Work in progress...
3. Usage limit detected
4. Wait 5 minutes (countdown displayed)
5. Auto-resume previous session
6. Continue working...
7. Repeat until complete
```

### Environment Variables

```bash
# Change retry delay (default 300 seconds = 5 minutes)
CLAUDE_LOOP_DELAY=180 claude-loop "fix errors"

# Max retry count (default 100)
CLAUDE_LOOP_MAX_RETRIES=50 claude-loop "big task"

# Debug mode (show JSON response)
CLAUDE_LOOP_DEBUG=1 claude-loop "test"
```

## Plugin vs Script

| Component | Description |
|-----------|-------------|
| **Plugin/Skill** | Installer helper. Run `/claude-loop:install` inside Claude Code to install the script |
| **claude-loop script** | The actual tool. Use `claude-loop "prompt"` from terminal |

```
┌─────────────────────────────────────────────────┐
│  Claude Code                                    │
│  └─ /claude-loop:install  → installs script    │
└─────────────────────────────────────────────────┘
                    ↓ install
┌─────────────────────────────────────────────────┐
│  Terminal (outside Claude Code)                 │
│  └─ claude-loop "prompt" → runs Claude w/retry │
└─────────────────────────────────────────────────┘
```

## License

MIT
