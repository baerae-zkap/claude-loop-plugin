---
name: install-claude-loop
description: Install claude-loop script for automatic usage limit retry
---

# Install Claude Loop

Installs the `claude-loop` command - a wrapper that automatically retries when Claude Code hits usage limits.

## Features

- Automatic session resume when usage limited
- Configurable retry delay and max retries
- JSON output parsing for accurate limit detection
- Countdown timer during wait periods

## Installation

Run all commands below in sequence:

### 1. Create directory and script

```bash
mkdir -p ~/.local/bin && cat > ~/.local/bin/claude-loop << 'EOF'
#!/bin/bash
# claude-loop - Rate limit auto-retry wrapper for Claude Code

set -e

RETRY_DELAY=${CLAUDE_LOOP_DELAY:-300}
MAX_RETRIES=${CLAUDE_LOOP_MAX_RETRIES:-100}
DEBUG=${CLAUDE_LOOP_DEBUG:-0}

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() { echo -e "${BLUE}[claude-loop]${NC} $1"; }
warn() { echo -e "${YELLOW}[claude-loop]${NC} $1"; }
error() { echo -e "${RED}[claude-loop]${NC} $1"; }
success() { echo -e "${GREEN}[claude-loop]${NC} $1"; }

get_project_key() { echo "$(pwd)" | tr '/' '-'; }

get_session_index() {
  local project_key=$(get_project_key)
  echo "$HOME/.claude/projects/$project_key/sessions-index.json"
}

get_latest_session() {
  local session_index=$(get_session_index)
  [[ -f "$session_index" ]] && jq -r '.entries | sort_by(.modified) | last | .sessionId' "$session_index" 2>/dev/null
}

check_limit_status() {
  local json="$1"
  local is_error=$(echo "$json" | jq -r '.is_error // false' 2>/dev/null)
  local subtype=$(echo "$json" | jq -r '.subtype // ""' 2>/dev/null)

  if [[ "$is_error" == "true" ]]; then
    case "$subtype" in
      *rate_limit*|*usage_limit*|*quota*|*exceeded*) echo "usage_limit"; return 0 ;;
      *) echo "error:$subtype"; return 0 ;;
    esac
  fi

  local result_text=$(echo "$json" | jq -r '.result // ""' 2>/dev/null)
  echo "$result_text" | grep -qiE "usage.*limit|rate.*limit|hit.*limit|quota.*exceeded" && { echo "usage_limit"; return 0; }
  echo "ok"
}

run_claude() {
  local prompt="$1" session_id="$2"
  if [[ -n "$session_id" ]]; then
    log "Resuming session: ${session_id:0:8}..."
    echo "$prompt" | claude --print --output-format json --resume "$session_id" 2>&1
  else
    log "Starting new session"
    echo "$prompt" | claude --print --output-format json 2>&1
  fi
}

extract_result() {
  local result=$(echo "$1" | jq -r '.result // empty' 2>/dev/null)
  [[ -n "$result" ]] && echo "$result" || echo "$1"
}

main() {
  local initial_prompt="$1" retry_count=0 prompt="$initial_prompt" session_id=""

  log "Claude Loop started (retry delay: ${RETRY_DELAY}s, max: ${MAX_RETRIES})"

  while [[ $retry_count -lt $MAX_RETRIES ]]; do
    [[ $retry_count -gt 0 ]] && { session_id=$(get_latest_session); prompt="Continue from where we left off."; }

    json_output=$(run_claude "$prompt" "$session_id")
    exit_code=$?

    [[ "$DEBUG" == "1" ]] && { echo -e "${BLUE}━━━ JSON Response ━━━${NC}"; echo "$json_output" | jq '.' 2>/dev/null || echo "$json_output"; }

    result_text=$(extract_result "$json_output")
    [[ "$DEBUG" != "1" ]] && echo "$result_text"

    status=$(check_limit_status "$json_output")

    case "$status" in
      usage_limit)
        ((retry_count++))
        warn "Usage limit detected (${retry_count}/${MAX_RETRIES})"
        warn "Retrying in ${RETRY_DELAY}s..."
        for ((i=RETRY_DELAY; i>0; i--)); do
          printf "\r${YELLOW}[claude-loop]${NC} Time remaining: %3ds" $i
          sleep 1
        done
        echo ""; continue ;;
      ok) [[ $exit_code -eq 0 ]] && { success "Completed"; exit 0; } ;;
      error:*) error "Error: ${status#error:}"; exit 1 ;;
    esac

    [[ $exit_code -ne 0 ]] && { error "Error (exit code: $exit_code)"; exit $exit_code; }
    success "Completed"; exit 0
  done

  error "Max retries exceeded"; exit 1
}

if [[ $# -eq 0 ]]; then
  cat << 'HELP'
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Claude Loop - Rate Limit Auto-Retry
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Usage:
  claude-loop "your prompt"    Non-interactive

Environment Variables:
  CLAUDE_LOOP_DELAY=300        Retry delay (seconds)
  CLAUDE_LOOP_MAX_RETRIES=100  Max retry count
  CLAUDE_LOOP_DEBUG=1          Show JSON output
HELP
  exit 0
fi

main "$1"
EOF
chmod +x ~/.local/bin/claude-loop
```

### 2. Add to PATH

```bash
SHELL_CONFIG="$HOME/.zshrc"
[[ "$SHELL" == *"bash"* ]] && SHELL_CONFIG="$HOME/.bashrc"
grep -q 'export PATH="$HOME/.local/bin:$PATH"' "$SHELL_CONFIG" 2>/dev/null || {
  echo '' >> "$SHELL_CONFIG"
  echo '# claude-loop' >> "$SHELL_CONFIG"
  echo 'export PATH="$HOME/.local/bin:$PATH"' >> "$SHELL_CONFIG"
}
echo "Done! Run: source $SHELL_CONFIG"
```

## Usage

```bash
source ~/.zshrc  # or ~/.bashrc
claude-loop "implement feature X"
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CLAUDE_LOOP_DELAY` | 300 | Seconds to wait before retry |
| `CLAUDE_LOOP_MAX_RETRIES` | 100 | Maximum retry attempts |
| `CLAUDE_LOOP_DEBUG` | 0 | Set to 1 for JSON output |
