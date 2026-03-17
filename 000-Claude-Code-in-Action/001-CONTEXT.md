# Context Management in Claude Code

Commands, strategies, and tips for managing the context window in Claude Code.

---

## Core Context Commands

### `/compact [instructions]`
Summarizes conversation history to free up context while preserving important details.

```
/compact
/compact Focus on the API changes
/compact Preserve all database migration details
```

- Clears older tool outputs first, then summarizes if needed
- Accepts optional instructions to control what gets preserved
- CLAUDE.md is fully reloaded fresh after compaction
- Early verbose instructions may be lost — put persistent rules in CLAUDE.md

---

### `/clear`
Wipes the entire conversation history and starts a completely fresh context.

```
/clear
```

- Unlike `/compact`, this does not summarize — it resets entirely
- Use between unrelated tasks to prevent context pollution
- Tip: use `/rename` before clearing so you can resume the session later
- If you've corrected Claude more than twice on the same issue, `/clear` and start fresh

---

### `/context`
Visualizes current context usage as a colored grid.

```
/context
```

- Shows which parts of the session are consuming the most tokens
- Highlights MCP server costs, file reads, and memory bloat
- Shows capacity warnings when approaching limits
- Run this regularly during long sessions

---

### `/cost`
Shows token usage statistics for the current session.

```
/cost
```

- Total cost in dollars
- Total API time and wall clock time
- Lines of code added/removed

---

### `/rewind`
Restores conversation and/or code to a previous checkpoint.

```
/rewind
```

**Keyboard shortcut:** `Esc + Esc`

Options when rewinding:
- **Restore conversation only** — keep code changes, discard conversation history
- **Restore code only** — keep conversation, undo code changes
- **Restore both** — full rollback to that checkpoint
- **Summarize from here** — compact messages from that point forward

Every file edit Claude makes creates an automatic checkpoint.

---

### `/rename`
Names the current session for easy retrieval later.

```
/rename "auth-refactor"
/rename "bug-fix-login"
```

Use before `/clear` so you can resume the session with `--resume`.

---

### `/btw`
Asks a quick question without adding it to the conversation history.

```
/btw what does this function return?
```

Appears as a dismissible overlay — zero context cost.

---

## CLI Flags for Context

### `--continue` / `-c`
Resumes the most recent session in the current directory.

```bash
claude --continue
claude -c
```

- Loads previous conversation history
- Does not inherit session-scoped permissions (re-approval required)

---

### `--resume` / `-r`
Resumes a specific session by name or ID.

```bash
claude --resume
claude --resume "auth-refactor"
claude --resume abc123
```

- Without arguments: shows an interactive picker
- Sessions are tied to the working directory

---

### `--fork-session`
Creates a new session ID while preserving the current conversation history.

```bash
claude --continue --fork-session
```

- Original session remains unchanged
- Useful for branching off to try a different approach
- Multiple terminals can work independently without interleaving messages

---

### `--add-dir`
Adds additional directories Claude can access in the session.

```bash
claude --add-dir ../apps ../lib
```

---

## Automatic Compaction

Claude Code compacts context automatically when approaching the window limit:

1. Clears older tool outputs first
2. Summarizes conversation history if still needed
3. Preserves your requests and key code snippets
4. Reloads CLAUDE.md fresh from disk

**What gets lost:** verbose outputs and early detailed instructions.
**What survives:** CLAUDE.md content, key decisions, code snippets.

### Customizing auto-compaction via CLAUDE.md

Add a section to your CLAUDE.md to control what gets prioritized:

```markdown
## Compact Instructions

When compacting, always preserve:
- The full list of modified files
- Any test commands used
- Database migration details
```

---

## Context Cost by Feature

| Feature | When Loaded | Cost | Recommendation |
|---|---|---|---|
| CLAUDE.md | Every session start | Every request | Keep under 500 lines |
| Skills | Description at start, full content on use | Low | Use `disable-model-invocation: true` for manual-only skills |
| MCP servers | Session start | Every request | Disconnect unused servers |
| Subagents | When spawned | Isolated from main | Use for context-heavy research |
| File reads | On demand | Accumulates | Read only what's needed |

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Esc + Esc` | Open rewind menu |
| `Esc` | Stop Claude mid-action (preserves context) |
| `Ctrl + L` | Clear terminal screen (keeps conversation history) |

---

## Context Management Strategies

1. **Use `/clear` between unrelated tasks** — prevents context pollution
2. **Use `/compact` with focus instructions** — direct what gets preserved
3. **Use `/btw` for quick questions** — no context cost
4. **Use `/rewind` to branch off** — try risky approaches safely
5. **Delegate to subagents** — keep main context clean for investigation tasks
6. **Monitor with `/context`** — know what's consuming space
7. **Keep CLAUDE.md concise** — under 500 lines; move reference material to skills
8. **Put persistent rules in CLAUDE.md** — not in conversation history (they'll get compacted away)

---

## Summary Table

| Command / Flag | Purpose |
|---|---|
| `/compact [focus]` | Summarize history, optionally with guidance |
| `/clear` | Full context reset |
| `/context` | Visualize token usage |
| `/cost` | Show session token stats |
| `/rewind` | Restore to previous checkpoint |
| `/rename` | Name session for later retrieval |
| `/btw` | Ask without adding to history |
| `--continue` | Resume most recent session |
| `--resume` | Pick a session to resume |
| `--fork-session` | Branch off with a new session ID |
| `--add-dir` | Add extra directories to context |
