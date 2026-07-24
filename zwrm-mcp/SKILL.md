---
name: zwrm-mcp
description: |
  ZWRM's MCP gateway: register upstream MCP servers, compose virtual servers from curated tool selections, connect them to agents or local MCP clients, and manage per-user OAuth connections. Use this skill when the user wants to give an agent external tools, register an MCP server, create a virtual MCP server, connect an OAuth-backed integration (GitHub, Microsoft 365, …), or point Claude Code at ZWRM tools. Triggers on "MCP", "connector", "virtual server", "upstream", "tool access", "connect my account", or "OAuth integration".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm mcp

The MCP gateway sits between agents and external MCP servers. **Upstreams** are the org's registered external servers; **virtual servers** are curated tool selections composed from upstreams; agents and local clients attach to virtual servers, never to upstreams directly.

## Register an upstream

```bash
# Bearer/header auth
zwrm mcp upstream add linear https://mcp.linear.app/mcp --bearer-token "lin_..."

# Per-user OAuth: each member connects their own account
zwrm mcp upstream add ms365 https://gateway.example/ms365 --oauth

# Service-account OAuth (client_credentials)
zwrm mcp upstream add crm https://crm.example/mcp \
  --oauth-client-id ... --oauth-client-secret ... --oauth-token-url https://auth.example/token

zwrm mcp upstream sync linear      # refresh its tool catalog
zwrm mcp upstream tools linear
```

## Compose a virtual server

```bash
zwrm mcp server create dev-tools \
  --upstream "linear:list_issues,create_issue" \
  --upstream github \
  --description "Issue tracking + code review tools"
zwrm mcp server get dev-tools
```

## Attach to an agent

```bash
zwrm agent connectors attach my-agent dev-tools
zwrm agent connectors attach my-agent prod-tools --escalate   # human approves each use in autonomous runs
zwrm agent connectors verify my-agent dev-tools               # live initialize → tools/list test
zwrm agent connectors list my-agent
```

## Per-user OAuth connections

```bash
zwrm mcp connect ms365                     # connect YOUR account (opens browser)
zwrm mcp connect ms365 --agent my-agent    # authorize the AGENT's own identity
zwrm mcp upstream grants ms365             # who's connected
zwrm mcp disconnect ms365
```

## Use from a local MCP client

```bash
zwrm mcp install dev-tools                 # print Claude Code config (--client/--all for others)
zwrm mcp                                   # stdio MCP server exposing agent runs themselves
```

## Command reference

```text
zwrm mcp — Run an MCP server exposing autonomous agent runs (stdio)

zwrm mcp connect <upstream> — Connect your account (or an agent's) to a per-user OAuth upstream
      --agent string   Authorize this agent (id or instance) instead of yourself

zwrm mcp disconnect <upstream> — Disconnect your account (or an agent's) from a per-user OAuth upstream
      --agent string   Disconnect this agent (id or instance) instead of yourself

zwrm mcp install <server> — Print client config to connect to a virtual server
      --all             Print config for every supported client
      --client string   Target client (default claude-code)

zwrm mcp server — Manage virtual MCP servers (curated tool compositions)

zwrm mcp server create <name> — Create a virtual server from upstream tool selections
      --description string     Description (served as MCP instructions)
      --upstream stringArray   Upstream selection "name" or "name:tool1,tool2" (repeatable)

zwrm mcp server get <name> — Show a virtual server and its exposed tools

zwrm mcp server list — List virtual servers

zwrm mcp server rm <name> — Remove a virtual server

zwrm mcp upstream — Manage the org's registered upstream MCP servers

zwrm mcp upstream add <name> <url> — Register an upstream MCP server
      --allow-private                Allow private/internal addresses (host-admin only)
      --bearer-token string          Bearer token sent to the upstream
      --header stringArray           Custom auth header "Name: value" (repeatable)
      --oauth                        Per-user OAuth sign-in (each member connects their own account)
      --oauth-client-id string       OAuth client ID (client_credentials flow, or a pre-registered app with --oauth)
      --oauth-client-secret string   OAuth client secret (client_credentials flow, or a pre-registered app with --oauth)
      --oauth-issuer string          Pin the authorization server for --oauth (e.g. https://login.microsoftonline.com/<tenant>/v2.0); requires --oauth-client-id
      --oauth-scope string           Optional space-separated OAuth scopes
      --oauth-token-url string       OAuth token endpoint URL (client_credentials flow)
      --transport string             Upstream transport: streamable-http (default) or sse

zwrm mcp upstream grants <name> — List connected accounts on a per-user OAuth upstream

zwrm mcp upstream list — List registered upstreams

zwrm mcp upstream rm <name> — Remove an upstream
      --force   Remove even when virtual servers reference this upstream

zwrm mcp upstream sync <name> — Re-sync an upstream's tool catalog

zwrm mcp upstream tools <name> — List an upstream's synced tool catalog
```

## Tips

- **Curate tools per virtual server** (`name:tool1,tool2`) — smaller tool sets make agents faster and safer than exposing whole upstreams.
- **`--escalate` on attach** pauses that connector's tool calls for human approval during autonomous runs — use it for anything that writes to production systems.
- **`verify` before debugging an agent** — it tests the full path (gateway → upstream → auth) in one shot.
- On a per-user OAuth upstream, tool calls run under whoever's grant applies (the caller's, or the agent's own via `--agent` connect).

## See also

- [zwrm-agent](../zwrm-agent/SKILL.md) — the agents these connectors serve
