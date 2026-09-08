---
name: loadster-setup
description: Connect Claude Code to Loadster's MCP server and verify the connection. Use when Loadster tools are missing, a Loadster tool returns 401 or unauthorized, or the user asks to set up, connect, or reconnect Loadster.
---

# Setting up the Loadster MCP connection

The Loadster plugin registers one MCP server, `loadster`, at `https://api.loadster.com/mcp`. It authenticates with
OAuth, so the user has to approve the connection once in their browser. You cannot complete that approval for them,
but you can get them there and confirm the result.

## Check whether Loadster is connected

Call `list_projects`. Three outcomes:

- **It returns projects.** The connection works. Tell the user which team's projects came back, because access is
  scoped to the team they connected in.
- **The tool is missing.** The plugin's MCP server hasn't started or isn't authenticated. Follow the steps below.
- **It fails with 401 or "unauthorized".** The OAuth connection was revoked, or the user is using a token that was
  revoked or mistyped. Follow the steps below.

## Connect with OAuth

Ask the user to:

1. Run `/mcp` in Claude Code.
2. Select **loadster** from the list and choose **Authenticate**.
3. Approve the connection in the browser window that opens. If they belong to more than one Loadster team, the
   team they pick there is the one the agent will work in.
4. Come back and confirm.

Then call `list_projects` again to verify.

## Connect with an MCP token instead

Only for headless use, CI, or clients where OAuth isn't possible. Tell the user to create a token in the Loadster
dashboard under **Settings → AI Agents → MCP Tokens**, copy it immediately (it's shown once), and register the server
outside the plugin:

```bash
claude mcp add --transport http loadster https://api.loadster.com/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

Never ask the user to paste a token into the conversation, and never write a token into a file that will be
committed.

## Troubleshooting

- **Stale or missing tools after a change:** ask the user to run `/mcp` and reconnect, or restart Claude Code.
- **Wrong team's projects:** the user has to disconnect and reconnect, choosing the right team during OAuth.
- **Still stuck:** point them at https://loadster.com/manual/ai-agents/ or help@loadster.com.
