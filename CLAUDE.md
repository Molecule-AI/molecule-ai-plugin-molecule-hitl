# molecule-ai-plugin-molecule-hitl

Human-in-the-loop (HITL) approval gate plugin for the Molecule AI platform.
Wraps `builtin_tools/hitl.py` with an `@requires_approval` decorator, provides
`pause_task` / `resume_task` tools, and routes approval requests across multiple
notification channels.

## Purpose

`molecule-hitl` lets humans review, approve, or reject agent actions before they
execute. It is the platform's primary safety mechanism for high-stakes operations
(destructive commands, external API calls, credential usage, etc.).

## Key Conventions

| Topic | Convention |
|---|---|
| **Decorator** | `@requires_approval(action_name, severity)` — marks a tool/function as requiring human sign-off |
| **Tool names** | `pause_task`, `resume_task` — registered in the runtime's tool registry |
| **Severity levels** | `low`, `medium`, `high`, `critical` — maps to approval timeout and escalation path |
| **Notification channels** | Dashboard (in-app), Slack, email — configured via `config.yaml → hitl:` |
| **RBAC bypass** | Users with `hitl_bypass` role can auto-approve without human intervention |
| **Runtimes** | Supports `langgraph`, `claude_code`, `deepagents` |

## Project Structure

```
molecule-ai-plugin-molecule-hitl/
├── skills/
│   └── hitl-gates/           # Skill manifest + prompt fragments
│       └── skill.yaml
├── adapters/                 # Runtime-specific adapter shims
│   └── <runtime>/
├── hooks/
│   └── <runtime>-hitl-hook.ts   # Pre-execution hook registration
├── rules/
│   └── hitl-gates.md
├── runbooks/
│   └── local-dev-setup.md
└── plugin.yaml
```

## Dev Setup

```bash
git clone https://github.com/Molecule-AI/molecule-ai-plugin-molecule-hitl
cd molecule-ai-plugin-molecule-hitl

# Validate plugin.yaml
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"

# Validate skill manifests
for f in skills/*/skill.yaml; do
  python3 -c "import yaml; yaml.safe_load(open('$f'))"
done
```

### Configuring Notification Channels

In your `config.yaml`:

```yaml
hitl:
  default_severity: medium          # fallback severity if not annotated
  timeout_seconds: 300             # approval window before auto-reject
  channels:
    dashboard: true
    slack:
      webhook_url: "${SLACK_WEBHOOK_URL}"
    email:
      smtp_host: "${SMTP_HOST}"
      smtp_port: 587
      from: "molecule-hitl@yourdomain.com"
  bypass_roles:
    - hitl_bypass
    - admin
```

## Testing

```bash
# Unit tests (mock approval flow)
pytest tests/ -v

# Integration: requires a running platform + a workspace with this plugin loaded
# See runbooks/local-dev-setup.md
```

## Release Process

1. Update `plugin.yaml` version
2. Update `skills/hitl-gates/skill.yaml` if prompt fragments changed
3. Tag: `git tag vX.Y.Z && git push --tags`
4. Update org templates that pin this plugin

## Rules

See `rules/hitl-gates.md`.

## Known Gotchas

- `pause_task` pauses the **agent execution**, not the underlying process. If the
  agent holds a database connection or file lock, releasing without resuming
  can leave resources in an inconsistent state.
- Slack notifications require `SLACK_WEBHOOK_URL` to be set; if absent, Slack
  channel is silently skipped and dashboard is used as fallback.
- RBAC bypass is evaluated at task-creation time; changing a user's role
  mid-approval does not retroactively bypass the pending approval.
- Approval state is stored in-memory in the runtime; a runtime restart wipes all
  pending approvals.
