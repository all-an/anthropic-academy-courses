# Thinking Prompts for Claude Code

Words and sentences you can use to activate deeper reasoning and improve Claude's suggestions.

---

## Core Thinking Keywords

- `think` — prompts a basic reasoning step before answering
- `think hard` — triggers more deliberate analysis
- `think harder` — pushes for deeper consideration of trade-offs
- `think carefully` — encourages methodical, step-by-step reasoning
- `think step by step` — forces explicit reasoning chain before output
- `think deeply` — activates thorough exploration of the problem space
- `think thoroughly` — ensures no corner cases are overlooked
- `ultrathink` — maximum reasoning effort; best for complex architecture, bugs, or decisions

---

## Sentence Templates

### Before starting a task
- "Think step by step before writing any code."
- "Think hard about the best approach before suggesting anything."
- "Think carefully about edge cases before implementing."
- "Ultrathink this problem — I need the best possible solution."
- "Think deeply about the trade-offs between these two approaches."

### For debugging
- "Think hard about what could be causing this bug."
- "Think step by step through the execution path to find the issue."
- "Ultrathink this bug — I've been stuck on it for hours."
- "Think carefully before suggesting a fix — explain your reasoning first."

### For architecture and design
- "Think deeply about the architecture before writing any code."
- "Ultrathink this design — we need it to scale."
- "Think hard about potential failure modes in this design."
- "Think thoroughly about security implications before proceeding."

### For code review
- "Think carefully about this code — what could go wrong?"
- "Think hard about whether this is the simplest solution."
- "Think step by step through this function and identify any issues."

### For refactoring
- "Think deeply about how to refactor this without breaking existing behavior."
- "Think hard before refactoring — what are the risks?"
- "Think step by step through the dependencies before making changes."

---

## Intensity Scale

| Keyword | Effort Level | Best Used For |
|---|---|---|
| `think` | Low | Simple clarifications |
| `think carefully` | Medium | Standard coding tasks |
| `think hard` | High | Complex bugs, tricky logic |
| `think step by step` | High | Multi-step problems, algorithms |
| `think deeply` | Very High | Architecture, design decisions |
| `ultrathink` | Maximum | Critical decisions, hard bugs, system design |

---

## Tips

- Combine keywords for emphasis: *"Think hard and step by step about..."*
- Use `ultrathink` sparingly — reserve it for genuinely complex problems.
- Adding *"explain your reasoning before writing code"* alongside any keyword further improves output quality.
- Asking Claude to *"consider alternatives before deciding"* alongside `think hard` helps surface better options.
