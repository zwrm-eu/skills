---
name: zwrm-agent
description: |
  Coding agents on ZWRM: interactive agent workspaces, autonomous agent runs, cron schedules, inbound triggers, agent memory, and skills. Use this skill when the user wants a cloud dev VM with Claude Code or Codex, wants to start or continue an autonomous agent run, schedule recurring agent work, route external events to an agent, or curate an agent's memory/skills/instructions. Triggers on "agent", "agent run", "run Claude on", "schedule a run", "cron agent", "trigger", "webhook to agent", "agent memory", or "coding agent VM".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm agent

Agents are named, org-scoped identities with defaults (model, effort, VM size, instructions), their own secrets, memory, skills, and MCP connectors. Use them two ways:

- **Interactive workspace**: `zwrm agent claude` attaches you to a persistent dev VM running Claude Code (or `zwrm agent pi` / `zwrm agent codex` for those harnesses).
- **Autonomous runs**: `zwrm agent run` executes a prompt headlessly on a fresh VM and reports the result.

## Interactive workspaces

```bash
zwrm agent claude                    # default instance; creates on first use
zwrm agent claude my-project         # named instance = separate volume per project
zwrm agent list
zwrm agent resize my-project --volume 20GB
zwrm agent destroy my-project        # stop the VM, KEEP the volume
zwrm agent delete my-project         # remove VM + volume + record (-f to skip confirm)
```

## Autonomous runs

```bash
zwrm agent run --agent my-project --prompt "Fix the failing CI on main"
zwrm agent run list --status running
zwrm agent run status <run-id>              # status + transcript
zwrm agent run continue <run-id> --prompt "Now add tests for that fix"
zwrm agent run cancel <run-id>
```

- Run VMs are ephemeral; the agent's identity (memory, secrets, connectors) persists across runs.
- `--session my-key` keeps a keyed workspace between runs so context and checkout survive; empty means a fresh workspace each run.
- Per-run overrides: `--model`, `--effort`, `--size`, `--timeout`.

## Agent configuration

```bash
zwrm agent update my-project --model opus --effort high \
  --instructions "Prefer small PRs. Never force-push." \
  --size performance-2x
zwrm agent budget my-project --daily-usd 20 --max-runs 50
zwrm agent logs my-project

# Secrets: pipe the value in, never type it into the command
printf '%s' "$GITHUB_TOKEN" | zwrm agent secrets set GITHUB_TOKEN --stdin --instance my-project
```

Never write a literal token into a `zwrm` command. If you don't have the value, ask the user
to export it (then reference `"$VAR"`) or to run the `--stdin` form themselves — see
[zwrm-secrets](../zwrm-secrets/SKILL.md).

`--allowed-scopes` on `update` constrains what the agent's boot token may do against the platform API (e.g. `apps:read,deploy,logs:read`).

## Memory and skills

```bash
zwrm agent memory list my-project
zwrm agent memory save my-project deploy-rules --description "Release process" --file notes.md
zwrm skills list                                  # org skill library
zwrm agent skills enable my-project <skill>       # live-syncs running workspaces
```

## Schedules (cron → runs)

```bash
zwrm schedules create nightly-triage --agent my-project \
  --cron "0 6 * * *" --prompt "Triage new GitHub issues; comment on each"
zwrm schedules list
zwrm schedules firings <schedule-id>
zwrm schedules pause <schedule-id>
```

## Triggers (external events → runs)

Triggers give you a webhook URL; each delivery starts a run:

```bash
zwrm triggers create sentry-alerts --default-agent my-project \
  --instruction "Investigate this Sentry alert and open an issue if real"
zwrm triggers deliveries <trigger-id>
```

`--match` gates deliveries, `--routes` picks agents per payload, `--session-key-path` keys workspace continuity on a payload field, `--secret` imports a provider-minted signing secret (Linear, Shopify, Sentry).

## Command reference

```text
zwrm agent [type] [instance] — Manage agents and their workspaces
      --cmd string        Override default command (e.g., "aider")
      --instance string   Named instance (e.g., work, personal) (default "default")
      --size string       VM size preset (default: the agent's stored size; performance-2x for a new agent)
      --template string   Agent template (registry ID or github.com/user/repo)

zwrm agent budget [instance] — Show or set an agent's daily spend/run caps
      --daily-usd float   Daily spend cap in USD (0 = unlimited)
      --max-runs int      Daily run-count cap (0 = unlimited)

zwrm agent connectors — Manage an agent's MCP connectors (gateway virtual servers)

zwrm agent connectors attach <agent> <server> — Attach a virtual server to an agent
      --disabled   Attach without enabling (no tools until re-enabled)
      --escalate   Pause this connector's tools for a human during autonomous runs

zwrm agent connectors copy <agent> --from <source-agent> — Copy another agent's connectors (with their enabled/escalate flags)
      --from string   Source agent (id or name) to copy connectors from

zwrm agent connectors detach <agent> <server> — Detach a virtual server from an agent

zwrm agent connectors list <agent> — List the virtual servers attached to an agent

zwrm agent connectors verify <agent> <server> — Live-test a connector (initialize → tools/list against its upstream)

zwrm agent delete [instance] — Delete an agent entirely (VM, volume, and record)
  -f, --force   Skip confirmation prompt

zwrm agent destroy [instance] — Destroy a running agent VM (preserves volume)

zwrm agent list — List all agent instances across all organizations

zwrm agent logs [instance] — Show activity logs for an agent
      --limit int   Maximum number of log entries (default 50)

zwrm agent memory — Curate an agent's durable memory entries

zwrm agent memory delete <agent> <slug> — Delete a memory entry

zwrm agent memory list <agent> — List the agent's memory entries

zwrm agent memory save <agent> <slug> — Create or replace a memory entry (body from --body, --file, or stdin)
      --body string          Entry body (markdown)
      --description string   One-line summary shown in the recall index (required)
      --file string          Read the body from a file
      --kind string          Optional category: user | project | reference | feedback

zwrm agent memory show <agent> <slug> — Print one memory entry's body

zwrm agent resize [instance] — Resize an agent's persistent volume
      --volume string   New volume size (e.g., 20GB, 500MB)

zwrm agent run — Start an agent run (a prompt on one of your agents)
      --agent string       Run on this agent (instance name or agent ID) (required)
      --effort string      Reasoning effort: low|medium|high|xhigh|max (default: agent default)
      --model string       Model, harness-specific: opus/sonnet for claude, catalog aliases for pi and codex (default: agent default)
      --prompt string      Task prompt for the agent (required)
      --session string     Session continuity key: same key = continue that workspace's context; empty = fresh workspace per run
      --size string        VM size preset for this run's VM, e.g. performance-2x (default: agent default)
      --timeout duration   Hard run timeout, e.g. 30m (0 = server default)

zwrm agent run cancel <run-id> — Cancel an agent run

zwrm agent run continue <run-id> — Continue a finished run's conversation with a follow-up run
      --effort string      Reasoning effort (default: the run being continued)
      --model string       Model, harness-specific: opus/sonnet for claude, catalog aliases for pi and codex (default: the run being continued)
      --prompt string      Follow-up prompt (required)
      --size string        VM size preset (default: the run being continued)
      --timeout duration   Hard run timeout, e.g. 30m (0 = server default)

zwrm agent run list — List agent runs
      --status string   Filter by status (queued, running, completed, failed, ...)

zwrm agent run status <run-id> — Show an agent run's status and result

zwrm agent secrets — Manage secrets for an agent instance

zwrm agent secrets add --from-file <path> — Add secrets from a file
      --from-file string   Path to .env file
      --instance string    Agent instance name (default "default")

zwrm agent secrets list — List secrets for an agent
      --instance string   Agent instance name (default "default")

zwrm agent secrets set <name> [value] — Set a secret for an agent
      --instance string   Agent instance name (default "default")
      --stdin             Read the secret value from standard input instead of argv

zwrm agent secrets unset <name> — Remove a secret from an agent
      --instance string   Agent instance name (default "default")

zwrm agent skills — Manage an agent's enabled skills

zwrm agent skills disable <agent> <skill> — Toggle a skill for an agent (live-syncs running workspaces)

zwrm agent skills enable <agent> <skill> — Toggle a skill for an agent (live-syncs running workspaces)

zwrm agent skills list <agent> — List the agent's skill states

zwrm agent update <instance-or-id> — Update an agent's defaults (size, model, effort, instructions, budget, TTL, scopes)
      --allowed-scopes strings   Boot-token capabilities (comma-separated resource:action, e.g. apps:read,deploy,secrets:read,secrets:write,postgres:read,postgres:write,volumes:read,volumes:write,routes:write,logs:read,skills:read,skills:write,memories:read,memories:write,schedules:read,schedules:write,agents:read,triggers:read,triggers:write,mcp:read,mcp:write,connectors:read,connectors:write,ssh,scale,destroy,identity); empty resets to default
      --daily-budget float       Daily spend cap in USD (0 = unlimited)
      --effort string            Default reasoning effort: low|medium|high|xhigh|max
      --instructions string      Per-agent system-prompt append
      --max-runs-per-day int     Daily run cap (0 = unlimited)
      --model string             Default model (harness-specific: opus/sonnet for claude, catalog aliases for pi and codex)
      --size string              VM size preset (e.g. performance-2x)
      --workspace-ttl duration   Keyed-workspace TTL, e.g. 720h (0 = never expire)

zwrm schedules — Manage scheduled agent runs (cron → agent runs)

zwrm schedules create <name> — Create a scheduled run
      --agent string           Agent to run (id or name) — required
      --cron string            Standard 5-field cron in UTC (e.g. '0 9 * * 1-5') or @hourly/@daily/@weekly/@monthly — required
      --metadata stringArray   Tag the schedule with key=value (repeatable), e.g. --metadata creator=alice
      --paused                 Create the schedule disabled
      --prompt string          The prompt for each run — required
      --session-key string     Workspace continuity key (blank = fresh workspace per run)

zwrm schedules delete <schedule-id> — Delete a schedule (and its firing log)

zwrm schedules firings <schedule-id> — Show a schedule's firing log (every due firing, any outcome)
      --limit int   Max firings to show (default 50)

zwrm schedules list — List scheduled runs
      --agent string           Only schedules for this agent (id or name)
      --metadata stringArray   Only schedules tagged with this key=value (repeatable, ANDed)

zwrm schedules pause <schedule-id> — Pause or resume a schedule (resume never catch-up-fires)

zwrm schedules resume <schedule-id> — Pause or resume a schedule (resume never catch-up-fires)

zwrm triggers — Manage inbound triggers (external events → agent runs)

zwrm triggers create <name> — Create an inbound trigger
      --default-agent string      Catch-all agent (id or name) when no route matches
      --instruction string        The agent's task for each delivery
      --match string              Match-condition gate as JSON: [{path,op,value?}]
      --message-template string   Render {dot.path} tokens over the body (blank = raw body)
      --no-secret                 Create a slug-gated trigger with no secret
      --rate-per-day int          Daily delivery cap (0 = unlimited)
      --routes string             Routing rules as JSON: [{agent_id,when?}]
      --secret string             Import a provider-minted signing secret (Linear, Shopify, Sentry) instead of generating one
      --session-key-path string   Dot-path whose value keys session continuity (blank = one-shot)

zwrm triggers delete <trigger-id> — Delete a trigger (and its delivery log)

zwrm triggers deliveries <trigger-id> — Show a trigger's delivery log
      --limit int   Max deliveries to show (default 50)

zwrm triggers list — List inbound triggers

zwrm triggers pause <trigger-id> — Pause or resume a trigger

zwrm triggers resume <trigger-id> — Pause or resume a trigger

zwrm skills — Manage the skill library (validated bundles agents can enable)

zwrm skills create <slug> — Create a skill (upload a bundle to activate it)
      --description string   When-to-use description shown to agents and humans
      --name string          Display name (default: the slug)

zwrm skills delete <skill> — Delete a skill (removes it from agents and live workspaces)

zwrm skills list — List the skill library

zwrm skills upload <skill> <bundle.zip> — Upload a bundle as the skill's next version
```

## Tips

- **Workspace vs run**: `destroy` keeps the volume (reattach later); `delete` is permanent.
- **Session keys** are the continuity primitive — same `--session` (or schedule `--session-key`, or trigger `--session-key-path` value) = same workspace.
- **Budgets are guardrails** for schedules and triggers — set them before wiring an agent to external events.
- MCP connectors for agents (`zwrm agent connectors`) are covered in [zwrm-mcp](../zwrm-mcp/SKILL.md).

## See also

- [zwrm-mcp](../zwrm-mcp/SKILL.md) — give agents external tools
- [zwrm-secrets](../zwrm-secrets/SKILL.md) — agent secret namespace
- [zwrm-sandbox](../zwrm-sandbox/SKILL.md) — plain code execution without an agent identity
