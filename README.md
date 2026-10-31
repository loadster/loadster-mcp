# Loadster MCP

[Loadster](https://loadster.com) is a cloud load testing and monitoring platform. Its built-in
[Model Context Protocol](https://modelcontextprotocol.io/) (MCP) server lets AI agents like Claude, ChatGPT, Codex,
Cursor, and VS Code write and play load test scripts, build load test scenarios, set up monitors, and analyze
results in your Loadster account.

This repository is the public home for connecting to that server: setup snippets for each client, the Claude Code
plugin, and the metadata behind Loadster's listings in MCP directories. The server itself is hosted by Loadster at
`https://api.loadster.com/mcp`, and its source is not published here. Loadster is also an official plugin in the
[ChatGPT and Codex plugin directory](https://chatgpt.com/plugins/plugin_asdk_app_6aa2109570b0819194170aae3449f6fb).

The maintained, full-length guide is the
[AI Assistants & MCP chapter of the Loadster manual](https://loadster.com/manual/mcp/).
This README is the short version.

## The Loadster MCP endpoint

| | |
| --- | --- |
| URL | `https://api.loadster.com/mcp` |
| Transport | Streamable HTTP |
| Authentication | OAuth 2.1 (preferred), or a personal access token in an `Authorization: Bearer` header |
| Registry name | `com.loadster/loadster-mcp` |

**OAuth** is the easiest way to connect. Clients that support MCP OAuth send you to Loadster to approve the
connection in your browser, and you can review or revoke connected agents on the **API & MCP** page in your Loadster
settings. The approval page leads with the web address Loadster can verify for the app, so check that it's one you
expect before you approve. You also choose an access level there: **Read only** (selected to start with), **Read and
edit**, or **Read, edit and execute**. To change it later, revoke the connection and connect again. Changing or
resetting your Loadster password disconnects every OAuth-connected agent, so reconnect each one afterward. Personal
access tokens keep working.

**Personal access tokens** are for clients that can't do OAuth, and for CI jobs and scripted agents. The same kind of
token also works for calling the [Loadster REST API](https://loadster.com/manual/rest-api/) directly. Create one in
the Loadster dashboard under **Settings → API & MCP → Personal Access Tokens**, choose its access level, copy it right
away (it's only shown once), and send it as `Authorization: Bearer YOUR_TOKEN`. A token acts as you within the team
where you created it, so treat it like a password.

## Connecting Claude Code to Loadster

The quickest way is the plugin from this repository, which adds the Loadster MCP server plus skills for a
sensible load testing workflow: `setup` to connect, `setup-script` for a first script, `setup-scenario` for a
first load test scenario, `setup-monitor` for a first synthetic monitor, `load-test` for a full test, and
`results` for reading what happened.

```
/plugin marketplace add loadster/loadster-mcp
/plugin install loadster@loadster
```

Then run `/mcp`, pick **loadster**, and approve the OAuth connection in your browser. Ask Claude something like
_"List my Loadster projects"_ to confirm it can reach your account.

Without the plugin, add the server directly and Claude Code will walk you through OAuth:

```bash
claude mcp add --transport http loadster https://api.loadster.com/mcp
```

Or with a personal access token, for headless use:

```bash
claude mcp add --transport http loadster https://api.loadster.com/mcp --header "Authorization: Bearer YOUR_TOKEN"
```

The equivalent project-level `.mcp.json`:

```json
{
  "mcpServers": {
    "loadster": {
      "type": "http",
      "url": "https://api.loadster.com/mcp"
    }
  }
}
```

## Connecting Claude Desktop and claude.ai to Loadster

Claude Desktop and claude.ai connect to remote MCP servers as custom connectors, which use OAuth.

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Enter a name and the URL `https://api.loadster.com/mcp`.
3. Choose **Connect** and approve the Loadster OAuth connection when your browser opens.
4. In a conversation, open the tools menu and turn on the Loadster connector.

## Connecting ChatGPT and Codex to Loadster

Loadster is an official plugin in the ChatGPT and Codex plugin directory, with the `setup`, `load-test`, and
`results` skills. The new `setup-script`, `setup-scenario`, and `setup-monitor` skills reach it when the directory
listing is updated.

- **ChatGPT:** open the [Loadster plugin](https://chatgpt.com/plugins/plugin_asdk_app_6aa2109570b0819194170aae3449f6fb),
  choose **Install plugin**, then **Connect** and approve the Loadster OAuth connection. Start a conversation with
  **@Loadster**. On a Business or Enterprise workspace, an admin may need to make it available under
  **Workspace settings → Plugins** first.
- **Codex:** install Loadster from the **Plugins** tab in the Codex desktop app, or run `/plugins` in the Codex CLI,
  then start a new session.

The Codex IDE extension doesn't support plugins, so it connects through Codex's MCP configuration, which the desktop
app and CLI share:

```bash
codex mcp add loadster --url https://api.loadster.com/mcp
codex mcp login loadster
```

Or with a personal access token in `~/.codex/config.toml`:

```toml
[mcp_servers.loadster]
url = "https://api.loadster.com/mcp"
bearer_token_env_var = "LOADSTER_MCP_TOKEN"
```

If the plugin isn't available on your ChatGPT account, you can still add Loadster as a custom connection in
**Developer mode** (**Settings → Security and login**). Open **Plugins**, add a connection with the URL
`https://api.loadster.com/mcp`, then add it to a new conversation from the tools menu and approve the OAuth
connection.

## Connecting Cursor to Loadster

[![Install in Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=loadster&config=eyJ1cmwiOiJodHRwczovL2FwaS5sb2Fkc3Rlci5jb20vbWNwIn0=)

Or add it to `.cursor/mcp.json` in your project (or `~/.cursor/mcp.json` for all projects) and approve the OAuth
connection when Cursor first connects:

```json
{
  "mcpServers": {
    "loadster": {
      "url": "https://api.loadster.com/mcp"
    }
  }
}
```

To use a personal access token instead, add `"headers": { "Authorization": "Bearer ${env:LOADSTER_MCP_TOKEN}" }` and set
`LOADSTER_MCP_TOKEN` in the environment before starting Cursor.

## Connecting VS Code to Loadster

[![Install in VS Code](https://img.shields.io/badge/VS_Code-Install_Loadster_MCP-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://vscode.dev/redirect/mcp/install?name=loadster&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.loadster.com%2Fmcp%22%7D)
[![Install in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Install_Loadster_MCP-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=loadster&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.loadster.com%2Fmcp%22%7D)

Or run **MCP: Add Server** from the Command Palette, or add it to `.vscode/mcp.json`:

```json
{
  "servers": {
    "loadster": {
      "type": "http",
      "url": "https://api.loadster.com/mcp"
    }
  }
}
```

To use a personal access token, declare a `promptString` input with `"password": true` and reference it in an
`Authorization` header. The [manual](https://loadster.com/manual/mcp/#connecting-vs-code-to-loadster) has the
complete example.

## Connecting other MCP clients to Loadster

Any client that supports remote servers over Streamable HTTP can connect the same way. For clients that only
support local (stdio) servers, the [mcp-remote](https://www.npmjs.com/package/mcp-remote) bridge usually works.
Leave out the `--header` arguments to have it run the OAuth flow instead of using a token:

```json
{
  "mcpServers": {
    "loadster": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://api.loadster.com/mcp", "--header", "Authorization: Bearer YOUR_TOKEN"]
    }
  }
}
```

## What an agent can do with Loadster

The server exposes most of the Loadster dashboard. Clients fetch the live tool list when they connect, so this table
is a map of the surface rather than the source of truth. The [MCP tool reference](https://loadster.com/manual/mcp-tools/)
in the Loadster manual is generated from the tool definitions themselves, with each tool's parameters and whether it
is a Read, Write, Sensitive write, or Execute tool.

| Group | Tools |
| --- | --- |
| Projects and documentation | `list_projects`, `create_project`, `get_documentation`, `get_scripting_api`, `get_example`, `list_command_types`, `get_command_schema`, `get_variable_schema` |
| Scripts | `list_scripts`, `get_script`, `create_script`, `update_script`, `duplicate_script`, `delete_script`, `import_script`, `validate_script`, `play_script`, `get_play_status`, `stop_script`, `get_step_detail`, `get_screenshot`, `list_script_revisions`, `get_script_revision`, `restore_script_revision`, `list_script_assets`, `get_script_asset`, `put_script_asset`, `delete_script_asset` |
| Datasets | `list_datasets`, `get_dataset`, `create_dataset`, `update_dataset`, `append_dataset_rows`, `delete_dataset` |
| Scenarios and engines | `list_scenarios`, `get_scenario`, `create_scenario`, `update_scenario`, `delete_scenario`, `list_engines` |
| Load tests and reports | `launch_load_test`, `stop_load_test`, `get_load_test_status`, `list_load_tests`, `get_load_test_report`, `get_load_test_url_timings`, `update_load_test_notes` |
| Monitoring | `list_monitoring_locations`, `list_monitors`, `get_monitor`, `create_monitor`, `update_monitor`, `enable_monitor`, `disable_monitor`, `delete_monitor`, `list_monitor_cycles`, `get_monitor_cycle_detail`, `get_monitoring_summary`, `list_incidents`, `get_incident` |
| Feedback | `submit_feedback` |

Scripts, scenarios, datasets, and monitors can carry tags. The list tools report each item's tag names, and the
create and update tools take a `tags` list. Tags are shared across your team, so agents reuse the names you already
have and create a new tag only when you ask for one.

Updating a script, scenario, monitor, or dataset changes only the fields the agent sends and keeps the rest. A list
that's sent, such as a scenario's bot groups or a dataset's rows, replaces the stored list rather than merging into
it. The load test tools take `loadTestId` and scenario bot groups take `engineId`; the older names `testId` and
`loadEngineId` still work in 2.0 and will be removed in the next major version.

The tools an agent sees depend on the access level you grant. **Read only** sees the tools that read, **Read and
edit** adds the tools that create, change, delete, and play scripts, and **Read, edit and execute** adds
`launch_load_test`, `stop_load_test`, and `enable_monitor`. Launching a load test and enabling a monitor send real
traffic and burn Fuel unless your plan covers them, so the server tells the agent to get your go-ahead before either.
Before a launch the agent gives you your balance and, when the test would burn Fuel, the Fuel estimate range the
dashboard shows. A launch with no estimate runs on your plan and burns no Fuel, and one with an estimate runs
entirely on Fuel. At
the other two levels the agent gets everything ready and hands off so you push the launch button in the dashboard.

Some actions are always left to humans. An agent cannot manage notification policies or maintenance windows, or
administer your team, billing, or Fuel.

Every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`), so
clients that honor them can prompt before writes and deletes.

## Safety practices for AI load testing

- **The agent acts in your account.** It edits the same scripts, scenarios, datasets, and monitors your team sees.
  Script changes create revisions you can restore, but it's still a good idea to review proposed changes and keep
  experiments in their own project.
- **Pages under test can talk to your agent.** Response bodies, page HTML, console output, logs, screenshots, and
  monitor messages come from the site you're testing, and anyone who can put text there can put instructions there.
  Loadster marks that content as untrusted data and tells agents not to follow instructions inside it, but agents can
  still be misled, so a read-only connection is safer when an agent only needs to analyze results.
- **Only test what's yours.** Point scripts only at systems you own or are authorized to test. Playing a script runs
  a single bot, but it's your responsibility either way.
- **Scope your access.** Use one token per agent or machine, name them clearly, and revoke any you no longer use.
  Grant the lowest access level the agent needs. If it only needs to read, keep **Read only** (selected to start with)
  when you approve the OAuth connection, or choose **Read only** when you create a token. It can then use only the
  tools that read, and can't change anything or play scripts. Choose **Read, edit and execute** only for an agent you
  want launching load tests or enabling monitors. Review OAuth connections on the **API & MCP** page in your
  Loadster settings. See [How Your Assistant Authenticates](https://loadster.com/manual/mcp/#how-your-assistant-authenticates).

## Support

- Setup help and questions: [help@loadster.com](mailto:help@loadster.com)
- Security issues: see [SECURITY.md](SECURITY.md)
- Problems with the contents of this repository (the README, plugin, or directory metadata): open an issue here

## What's in this repository

| Path | Purpose |
| --- | --- |
| `server.json` | Loadster's entry in the [official MCP Registry](https://registry.modelcontextprotocol.io/) |
| `glama.json` | Maintainer metadata for the [Glama](https://glama.ai/mcp/servers) listing |
| `.claude-plugin/`, `.mcp.json`, `skills/` | The Claude Code plugin, its marketplace manifest, and the `setup`, `setup-script`, `setup-scenario`, `setup-monitor`, `load-test`, and `results` skills |
| `.github/workflows/` | Validation on every push, and registry publishing on release |

## License

The contents of this repository are released under the [MIT License](LICENSE). The Loadster service itself is
governed by [Loadster's terms of service](https://loadster.com/legal/terms-of-service/).
