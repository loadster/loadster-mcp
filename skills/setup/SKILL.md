---
name: setup
description: Connect the current client to Loadster's MCP server and verify the connection. Use when Loadster tools are missing, a Loadster tool returns 401 or unauthorized, or the user asks to set up, connect, or reconnect Loadster.
---

# Setting up the Loadster MCP connection

The Loadster MCP server is `loadster`, at `https://api.loadster.com/mcp`. It authenticates with OAuth, so the user
has to approve the connection once in their browser. You cannot complete that approval for them, but you can get
them there and confirm the result. The steps depend on the client. Use only the section for the client they are
actually using. For a client that is not listed here, follow that client's section in the repository README rather
than inventing steps.

## Check whether Loadster is connected

Call `list_projects`. Three outcomes:

- **It returns projects.** The connection works. Tell the user which team's projects came back, because access is
  scoped to the team they connected in. Then give them the getting-started overview below.
- **The tool is missing.** The server hasn't started or isn't authenticated. Follow the section for their client.
- **It fails with 401 or "unauthorized".** The OAuth connection was revoked, or the user is using a token that was
  revoked or mistyped. Follow the section for their client.

After they approve a connection, call `list_projects` again to verify.

## Claude Code

Ask the user to:

1. Run `/mcp` in Claude Code.
2. Select **loadster** from the list and choose **Authenticate**.
3. Approve the connection in the browser window that opens. If they belong to more than one Loadster team, the
   team they pick there is the one the agent will work in.
4. Come back and confirm.

Without the plugin, add the server directly and Claude Code will walk them through OAuth:

```bash
claude mcp add --transport http loadster https://api.loadster.com/mcp
```

Or with an MCP token, for headless use:

```bash
claude mcp add --transport http loadster https://api.loadster.com/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

## ChatGPT

Ask the user to reconnect through **Plugins**, then **Loadster**, then **Connect**, and to approve the Loadster
OAuth connection in the browser. If Loadster is not installed yet, they install the plugin first (on a Business or
Enterprise workspace an admin may need to allow it), then Connect. If the plugin is not available on their account,
the Developer mode fallback is the one in the repository README.

## Codex

Ask the user to run `/plugins` and install or open Loadster, approve the OAuth connection, and start a new session.
If Loadster is already configured as an MCP server, including in the Codex IDE extension (which has no plugins),
ask them to run `codex mcp login loadster`.

## Cursor

Ask the user to approve the OAuth connection when Cursor prompts. The repository README has an install link. The
same server in `.cursor/mcp.json`, or in `~/.cursor/mcp.json` for every project, is:

```json
{
  "mcpServers": {
    "loadster": {
      "url": "https://api.loadster.com/mcp"
    }
  }
}
```

To use a token instead, add the README's `Authorization` header (`Bearer ${env:LOADSTER_MCP_TOKEN}`) and set
`LOADSTER_MCP_TOKEN` before starting Cursor. Never ask them to paste the token into the chat.

## MCP tokens

Only for headless use, CI, or a client where OAuth isn't possible. Tell the user to create a token in the Loadster
dashboard under **Settings → AI Agents → MCP Tokens** and copy it immediately (it's shown once). The README has
the snippet for each client. Never ask the user to paste a token into the conversation, and never write a token
into a file that will be committed.

## Troubleshooting

- **Stale or missing tools after a change:** repeat the section for their client. In Claude Code that is `/mcp`
  or a restart. In ChatGPT, start a new conversation after reconnecting. In Codex, start a new session.
- **Wrong team's projects:** the user has to disconnect and reconnect, choosing the right team during OAuth.
- **Still stuck:** point them at https://loadster.com/manual/ai-agents/ or help@loadster.com.

## Rules

- Never launch a cloud load test, enable a monitor, or spend Fuel without explicit human confirmation in-product,
  and never claim that you can.
- Prefer an existing project. Call `list_projects`, and call `create_project` only when no project exists or the
  user explicitly asks for one.
- Ask the user for target URLs, how to handle credentials, and regions. Never invent customer facts.

## Getting started

When `list_projects` succeeds, point the user at the skill for what they want to do first:

- **`setup-script`** creates or imports a first Protocol, Browser Bot, or Playwright script, then validates it,
  plays it, and checks the play status.
- **`setup-scenario`** wires the bot groups for a first load test scenario. The shape is the script, the bots,
  the ramp, the duration, and the region. It reports the Fuel estimate and the launch blockers.
- **`setup-monitor`** attaches a script to a synthetic monitor definition.

`load-test` is the longer path that builds a script and a scenario together. `results` is for a finished load
test or a monitor incident. All three first-run skills follow the rules above.
