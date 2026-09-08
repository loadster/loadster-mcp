# Security

This repository holds documentation, a Claude Code plugin, and directory metadata. It contains no server code. The
Loadster MCP server runs at `https://api.loadster.com/mcp` as part of the Loadster service.

## Reporting a vulnerability

Please report security issues in the Loadster MCP server, its OAuth flow, or anything else in the Loadster service
to [security@loadster.com](mailto:security@loadster.com), following Loadster's
[vulnerability disclosure policy](https://loadster.com/legal/vulnerability-disclosure-policy/). Please don't open a
public GitHub issue for security reports.

Issues with the contents of this repository (for example a setup snippet that would leak a token, or a skill that
encourages an unsafe practice) can go to the same address.

## Tokens and connections

- MCP tokens act as the user who created them, within that team. Treat them like passwords, use one per agent or
  machine, and revoke unused tokens under **Settings → AI Agents** in the Loadster dashboard.
- OAuth connections can be reviewed and revoked on the same page.
- Never commit a token to a repository. The snippets here use placeholders, environment variables, or editor-managed
  secret inputs for that reason.
