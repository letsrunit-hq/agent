![letsrunit](https://cdn.jsdelivr.net/gh/letsrunit-hq/letsrunit@main/docs/.gitbook/assets/logo-light.svg)

# AI Agent Integration

Skills and plugin for AI coding agents. Gives your agent a real Chromium browser it can drive with Gherkin steps — and produces `.feature` files that keep running in CI after the session ends.

## Claude Code

Install the plugin once to get both the MCP server and the skill:

```
/plugin marketplace add letsrunit-hq/agent
/plugin install letsrunit@letsrunit
```

On the next conversation, Claude has the full step library, the locator syntax, and knows how to handle failures on its own.

## Cursor

Add `.cursor/mcp.json` to your project root:

```json
{
  "mcpServers": {
    "letsrunit": {
      "command": "npx",
      "args": ["-y", "@letsrunit/mcp-server@latest"]
    }
  }
}
```

Then open **Settings → Rules**, click **Add Rule → Remote Rule (GitHub)**, and enter:

```
https://github.com/letsrunit-hq/agent
```

## Codex CLI

Install the MCP server

```bash
codex mcp add letsrunit -- npx -y @letsrunit/mcp-server@latest
```

Install the skill:

```bash
npx skills add letsrunit-hq/agent
```

## Other agents

Add the MCP server in your agent's config format:

```json
{
  "mcpServers": {
    "letsrunit": {
      "command": "npx",
      "args": ["-y", "@letsrunit/mcp-server@latest"]
    }
  }
}
```

Install the skill using

```bash
npx skills add letsrunit-hq/agent
```

## AGENTS.md

Add the following to your `AGENTS.md` or `CLAUDE.md` to instruct the agent to verify its work with letsrunit and commit passing tests as regression tests:

```markdown
## Browser testing

Use letsrunit to verify any task that changes UI behaviour.

A task is not complete until the relevant flow has been tested in a real browser and passes.

When a scenario passes:

- Save it as a `.feature` file under `/features`
- Include it in the commit
- These files run in CI as regression tests for future changes

Only keep scenarios that cover flows that could plausibly break from an unrelated change. Skip trivial or redundant ones.
```

