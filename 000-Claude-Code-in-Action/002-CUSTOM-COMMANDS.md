# Custom Commands in Claude Code

Custom commands (also called **skills**) let you create reusable slash commands that Claude executes with your project context. Invoke them with `/command-name`.

---

## Storage Locations

| Scope | Path | Shared via |
|---|---|---|
| **Personal** | `~/.claude/skills/<name>/SKILL.md` | Only you, across all projects |
| **Project** | `.claude/skills/<name>/SKILL.md` | Git — shared with the team |
| **Legacy** | `.claude/commands/<name>.md` | Git — still works, not recommended |

Personal skills take precedence over project skills when names conflict.

---

## File Structure

```
.claude/
└── skills/
    └── my-command/
        ├── SKILL.md          # required — main instructions
        ├── reference.md      # optional — docs Claude can read
        └── templates/
            └── template.md   # optional
```

### SKILL.md format

```markdown
---
name: my-command
description: What this does and when Claude should use it
---

Your instructions here...
```

The `description` field is critical — Claude uses it to decide when to auto-invoke the skill.

---

## Invoking Commands

```
/my-command
/my-command some arguments here
```

Type `/` to see all available commands. Claude can also invoke skills automatically when your prompt matches the description.

---

## Arguments

Use `$ARGUMENTS` to pass input to the command:

```markdown
---
name: fix-issue
description: Fix a GitHub issue by number
---

Fix GitHub issue #$ARGUMENTS following our coding standards.
Fetch the issue with `gh issue view $ARGUMENTS`, implement the fix, and write tests.
```

Usage: `/fix-issue 123`

Access individual arguments by position with `$0`, `$1`, `$2` (0-indexed):

```markdown
---
name: migrate-component
description: Migrate a component between frameworks
---

Migrate the $0 component from $1 to $2. Preserve all existing behavior and tests.
```

Usage: `/migrate-component SearchBar React Vue`

### Available variables

| Variable | Value |
|---|---|
| `$ARGUMENTS` | Everything passed after the command name |
| `$0`, `$1`, `$2` | Individual arguments by position |
| `${CLAUDE_SESSION_ID}` | Current session ID |
| `${CLAUDE_SKILL_DIR}` | Directory containing this SKILL.md |

---

## Controlling Invocation

```yaml
---
name: deploy
description: Deploy to production
disable-model-invocation: true   # only YOU can trigger this
---
```

```yaml
---
name: legacy-api-context
description: Reference for deprecated API patterns
user-invocable: false            # only CLAUDE can use this, hidden from menu
---
```

| Setting | You invoke `/name`? | Claude auto-invokes? |
|---|---|---|
| (default) | Yes | Yes |
| `disable-model-invocation: true` | Yes | No |
| `user-invocable: false` | No | Yes |

Use `disable-model-invocation: true` for destructive commands like deploys that should never run automatically.

---

## Frontmatter Reference

```yaml
---
name: my-command
description: What this skill does and when to use it
argument-hint: "[issue-number]"       # shown in autocomplete
disable-model-invocation: true        # prevent auto-invocation
user-invocable: false                 # hide from / menu
allowed-tools: Read, Grep, Bash       # restrict tool access
model: claude-opus-4-6                # model override
context: fork                         # run in isolated subagent
agent: Explore                        # subagent type (with context: fork)
---
```

---

## Running in Isolated Context

Add `context: fork` to run the skill as an isolated subagent. The subagent handles the task and returns a summary — it doesn't pollute your main conversation context.

```yaml
---
name: deep-research
description: Research a topic thoroughly without cluttering the main context
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob
---

Research $ARGUMENTS:
1. Find relevant files with Glob and Grep
2. Read and analyze the code
3. Return a concise summary of findings
```

---

## Practical Examples

### Commit with convention

```markdown
---
name: commit
description: Create a conventional commit message
---

Stage all changes and create a commit following conventional commits format.
Run `git diff --staged` first to understand what changed, then write a concise
commit message (type: subject) and commit.
```

### Review before PR

```markdown
---
name: pr-review
description: Review code quality and security before opening a PR
context: fork
agent: Explore
---

Review the staged changes for:
- Code quality and readability
- Security vulnerabilities
- Missing tests
- Breaking changes

Run `git diff main` to see the full diff. Summarize findings.
```

### Fix a GitHub issue

```markdown
---
name: fix
description: Fix a GitHub issue by number
disable-model-invocation: true
argument-hint: "[issue-number]"
---

Fix GitHub issue #$ARGUMENTS:
1. `gh issue view $ARGUMENTS` — read requirements
2. Find and edit the relevant files
3. Write or update tests
4. Commit referencing the issue number
```

---

## Context Behavior

- **Skill descriptions** are always loaded into context so Claude knows what's available
- **Full skill content** is only loaded when the skill is actually invoked
- **CLAUDE.md** is reloaded fresh after every compaction — skills benefit from this too
- If you have many skills, run `/context` to check for warnings about context usage

### CLAUDE.md vs Skills

| Use CLAUDE.md for... | Use a Skill for... |
|---|---|
| Permanent rules and conventions | Repeatable multi-step workflows |
| Architecture notes | Deployments, migrations, releases |
| Standards that always apply | Tasks you trigger on demand |

---

## Quick Reference

```
.claude/skills/<name>/SKILL.md     project-level skill
~/.claude/skills/<name>/SKILL.md   personal skill (all projects)

/my-command                        invoke with no args
/my-command some args              invoke with $ARGUMENTS
/my-command arg0 arg1 arg2         invoke with $0, $1, $2
```