# MCP Servers on Claude Code

MCP (Model Context Protocol) is an open protocol that lets Claude connect to external tools, data sources, and services. Each MCP server exposes a set of tools that Claude can call during a session.

---

## How MCP Servers Work

When a server is connected, its tools are loaded into the session at startup — every request after that carries those tool definitions. This means MCP servers have a **context cost**, so only connect what you actually need.

There are two scopes for configuring servers:

| Scope | Config file | When to use |
|---|---|---|
| **Project** | `.claude/settings.json` (inside repo) | Team-shared tools for this project |
| **User (global)** | `~/.claude/settings.json` | Personal tools available in every project |

---

## Adding a Server via CLI

The fastest way is the `claude mcp add` command:

```bash
# Standard (stdio) server
claude mcp add <name> <command> [args...]

# SSE server (HTTP)
claude mcp add --transport sse <name> <url>
```

Use `-s global` to add to your user config instead of the project config:

```bash
claude mcp add -s global <name> <command>
```

---

## Example 1 — Playwright (Browser Automation)

The Playwright MCP server gives Claude a real browser it can control: navigate pages, click elements, fill forms, take screenshots, and run end-to-end flows.

### Install

No global install needed — `npx` handles it on first run.

### Add to your project

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

This writes to `.claude/settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### Add globally (available in all projects)

```bash
claude mcp add -s global playwright npx @playwright/mcp@latest
```

### What you can do once connected

| Task | Example prompt |
|---|---|
| Open a URL and describe it | "Navigate to localhost:3000 and describe what you see" |
| Screenshot a page | "Take a screenshot of the login page" |
| Fill and submit a form | "Fill the signup form with test data and submit" |
| Run an E2E flow | "Walk through the checkout flow and report any errors" |
| Debug a UI bug | "Click the Submit button and tell me what happens in the network tab" |

### Playwright-specific options

| Flag | Purpose |
|---|---|
| `--headless` | Run browser without a visible window (default in CI) |
| `--browser chromium\|firefox\|webkit` | Choose the browser engine |
| `--viewport-size "1280,720"` | Set browser viewport |
| `--device "iPhone 15"` | Emulate a mobile device |

Example with options:

```bash
claude mcp add playwright npx @playwright/mcp@latest --headless --browser firefox
```

Or in `settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--headless",
        "--browser", "firefox"
      ]
    }
  }
}
```

---

## Managing Servers

```bash
# List all configured servers and their status
claude mcp list

# Show details for one server
claude mcp get playwright

# Remove a server
claude mcp remove playwright

# Remove from global config
claude mcp remove -s global playwright
```

You can also toggle servers inside an active session with `/mcp`.

---

## Passing Environment Variables

For servers that need API keys or tokens, pass them via `env`:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["my-mcp-server"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

Or reference shell variables (they are expanded at startup):

```json
"env": {
  "API_KEY": "${MY_ENV_VAR}"
}
```

---

## Context Cost Reminder

MCP tool definitions are injected into every request for the duration of the session. Keep an eye on usage with `/context` and disconnect servers you are not actively using — especially in long sessions.

---

## Summary

| Command | Purpose |
|---|---|
| `claude mcp add <name> <cmd>` | Add a stdio server to project config |
| `claude mcp add -s global ...` | Add to user-level global config |
| `claude mcp add --transport sse <name> <url>` | Add an SSE/HTTP server |
| `claude mcp list` | List configured servers |
| `claude mcp get <name>` | Show server details |
| `claude mcp remove <name>` | Remove a server |
| `/mcp` (in-session) | Toggle servers without restarting |
