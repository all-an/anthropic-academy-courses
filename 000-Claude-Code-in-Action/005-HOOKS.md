# Hooks in Claude Code

Hooks let you run shell commands automatically in response to Claude Code events — before a tool runs, after it runs, when a session starts, and more. They are defined in `settings.json` and executed by the harness, not by Claude.

---

## Where Hooks Live

Hooks go inside your Claude Code settings file:

| Scope | Path |
|---|---|
| **User (all projects)** | `~/.claude/settings.json` |
| **Project** | `.claude/settings.json` |
| **Project local (gitignored)** | `.claude/settings.local.json` |

Project-level hooks override user-level hooks when names conflict.

---

## Hook Anatomy

```json
{
  "hooks": {
    "<event>": [
      {
        "matcher": "<tool-name or pattern>",
        "hooks": [
          {
            "type": "command",
            "command": "your shell command here"
          }
        ]
      }
    ]
  }
}
```

- **`<event>`** — lifecycle event that triggers the hook (see table below).
- **`matcher`** — optional glob/string that filters which tool calls trigger this hook.
- **`command`** — the shell command to run. Receives context via environment variables and/or stdin (JSON).

---

## Events

| Event | When it fires |
|---|---|
| `PreToolUse` | Before Claude calls any tool |
| `PostToolUse` | After a tool call completes |
| `Notification` | When Claude sends a background notification |
| `Stop` | When Claude finishes a turn |
| `SubagentStop` | When a subagent finishes |

---

## Hook Exit Codes

The exit code of your command controls what happens next:

| Exit code | Effect |
|---|---|
| `0` | Success — Claude continues normally |
| `2` | **Block** — Claude is told the action was blocked; it can adjust |
| anything else | Error — logged, Claude continues |

Use exit code `2` to prevent Claude from proceeding with a specific tool call.

---

## Step-by-Step: Your First Hook

### 1. Open (or create) your settings file

```bash
# user-level (applies to all projects)
code ~/.claude/settings.json

# project-level
code .claude/settings.json
```

### 2. Add the `hooks` key

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Bash tool finished'"
          }
        ]
      }
    ]
  }
}
```

### 3. Save and start a new Claude Code session

Hooks are loaded at session start — changes take effect on the next session.

---

## Useful Hooks

### Auto-format after file edits

Run your formatter whenever Claude writes or edits a file.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### Run tests after every code change

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit|NotebookEdit",
        "hooks": [
          {
            "type": "command",
            "command": "npm test --passWithNoTests 2>&1 | tail -5"
          }
        ]
      }
    ]
  }
}
```

### Desktop notification when Claude finishes

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude finished\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

> On Linux replace with: `notify-send "Claude Code" "Claude finished"`

### Log every tool Claude calls

Useful for auditing or debugging what Claude does in a session.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo \"$(date -u +%FT%T) TOOL: $CLAUDE_TOOL_NAME\" >> ~/.claude/tool-audit.log"
          }
        ]
      }
    ]
  }
}
```

---

## Preventing Claude from Reading or Grepping Specific Files

Use a `PreToolUse` hook with exit code `2` to block `Read` and `Grep` calls on sensitive files.

### Block a specific file

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"\nimport json, sys, os\ninput_data = json.load(sys.stdin)\npath = input_data.get('file_path', '')\nblocked = ['.env', '.env.local', 'secrets.json', 'credentials.json']\nif any(os.path.basename(path) == b for b in blocked):\n    print(f'Blocked: reading {path} is not allowed.', file=sys.stderr)\n    sys.exit(2)\n\""
          }
        ]
      }
    ]
  }
}
```

### Block Read and Grep on entire directories or patterns

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read|Grep",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"\nimport json, sys\ndata = json.load(sys.stdin)\n# Read passes file_path; Grep passes path\npath = data.get('file_path') or data.get('path', '')\nblocked_patterns = ['secrets/', '.env', 'private/', 'certs/']\nif any(p in path for p in blocked_patterns):\n    print(f'Blocked: access to {path} is restricted.', file=sys.stderr)\n    sys.exit(2)\n\""
          }
        ]
      }
    ]
  }
}
```

### Using a dedicated script (recommended for complex rules)

For maintainability, move the logic to a standalone script.

**`.claude/hooks/guard-sensitive-files.py`**

```python
#!/usr/bin/env python3
import json
import sys

BLOCKED_PATHS = [
    ".env",
    ".env.local",
    ".env.production",
    "secrets.json",
    "credentials.json",
]

BLOCKED_DIRS = [
    "secrets/",
    "private/",
    ".ssh/",
    "certs/",
]

data = json.load(sys.stdin)
path = data.get("file_path") or data.get("path", "")

for name in BLOCKED_PATHS:
    if path.endswith(name):
        print(f"Blocked: {path} is a protected file.", file=sys.stderr)
        sys.exit(2)

for directory in BLOCKED_DIRS:
    if directory in path:
        print(f"Blocked: {path} is inside a protected directory.", file=sys.stderr)
        sys.exit(2)
```

**`.claude/settings.json`**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read|Grep|Glob",
        "hooks": [
          {
            "type": "command",
            "command": "python3 .claude/hooks/guard-sensitive-files.py"
          }
        ]
      }
    ]
  }
}
```

Make it executable:

```bash
chmod +x .claude/hooks/guard-sensitive-files.py
```

---

## Hook Environment Variables

These variables are available inside every hook command:

| Variable | Value |
|---|---|
| `CLAUDE_TOOL_NAME` | Name of the tool being called (e.g. `Read`, `Bash`) |
| `CLAUDE_TOOL_INPUT_FILE_PATH` | File path argument (Write/Edit/Read tools) |
| `CLAUDE_SESSION_ID` | Current session ID |

The full tool input is also piped as JSON on **stdin** — parse it with `jq` or Python for richer logic.

---

## Configuring Hooks via the `/update-config` Skill

Instead of editing JSON by hand, use the built-in skill:

```
/update-config add a PostToolUse hook that runs prettier after Write and Edit
/update-config block Claude from reading .env files
```

The skill edits `settings.json` for you and explains what it changed.

---

## Quick Reference

```
~/.claude/settings.json          user-level hooks (all projects)
.claude/settings.json            project hooks (committed)
.claude/settings.local.json      project hooks (gitignored)

PreToolUse                       fires before the tool runs
PostToolUse                      fires after the tool runs
Stop                             fires when Claude's turn ends

exit 0    → allow
exit 2    → block (Claude is notified)
```
