# molecule-hitl — Local Dev Setup

## Prerequisites

- `git`
- Python 3.11+
- A local or staging Molecule platform
- A workspace with the `claude_code` runtime

## Clone and Validate

```bash
git clone https://github.com/Molecule-AI/molecule-ai-plugin-molecule-hitl
cd molecule-ai-plugin-molecule-hitl

# Validate plugin.yaml
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"

# Validate skill manifest
python3 -c "import yaml; yaml.safe_load(open('skills/hitl-gates/skill.yaml'))"
```

## Set Up a Dev Workspace

Add the plugin to your org template (`org.yaml`):

```yaml
plugins:
  - molecule-hitl
```

Configure HITL settings in `config.yaml`:

```yaml
hitl:
  default_severity: medium
  timeout_seconds: 60          # short timeout for faster testing
  channels:
    dashboard: true
    slack: {}                  # skip Slack in dev
    email:
      smtp_host: localhost
      smtp_port: 1025           # Mailhog / MailHog dev SMTP
      from: "hitl-dev@localhost"
      to: ["dev@localhost"]
  bypass_roles:
    - admin
```

## Triggering a Test Approval

From a Claude Code session in a workspace with this plugin:

```python
# Pseudocode — adapt to your runtime's HITL tool call
result = agent.call_tool(
    "requires_approval",
    action="delete_production_table",
    severity="critical",
    reason="Deleting prod_events table",
)
# This should pause and wait for approval
```

## Checking Approval State

In the platform dashboard, navigate to the workspace and look for pending
approvals. Approve or reject to observe the agent resume or fail.

## Via Platform API

```bash
# List pending approvals (once API is available)
curl -H "Authorization: Bearer $MOL_API_KEY" \
  "$MOL_PLATFORM_URL/hitl/approvals?workspace_id=$WORKSPACE_ID&status=pending"

# Approve
curl -X POST -H "Authorization: Bearer $MOL_API_KEY" \
  "$MOL_PLATFORM_URL/hitl/approvals/<id>/approve"
```

## Common Issues

| Symptom | Cause | Fix |
|---|---|---|
| No dashboard approval visible | Plugin not loaded in workspace | Restart workspace with plugin in org template |
| Timeout fires instantly | `timeout_seconds: 0` in config | Set to a positive value |
| Slack not sending | `SLACK_WEBHOOK_URL` not set | Remove `slack:` from channels or set the env var |
| All channels silent | SMTP/Mailhog not running | Start Mailhog: `docker run -p 8025:8025 -p 1025:1025 mailhog/mailhog` |
