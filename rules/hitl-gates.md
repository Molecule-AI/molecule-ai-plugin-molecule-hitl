# molecule-hitl — HITL Gate Rules

## Overview

Human-in-the-loop gates intercept agent actions that require human review before
execution. This document covers error handling, logging, configuration, and
the release process for this plugin.

---

## Error Handling

### Timeout Errors

If a human does not respond within `hitl.timeout_seconds`, the task is marked
`TIMEOUT` and the agent receives a `HITL_TIMEOUT` error. The calling tool or
workflow should treat this as a failure and surface the timeout to the user.

```
Error code: HITL_TIMEOUT
Message: "Approval request timed out after 300s"
Recovery: retry (re-triggers approval) or cancel
```

### Notification Failures

If all configured notification channels fail (Slack unreachable, SMTP error,
dashboard unavailable), the approval is still created and the runtime waits.
Log a warning:

```
[WARN] hitl: all notification channels failed for approval <id>
[WARN] hitl: approval <id> created but human may not be alerted
```

### Runtime Errors During Approval Processing

If `pause_task` or `resume_task` encounters an internal error (e.g., task store
unavailable), it raises `HITLError` with a descriptive message. Do not retry
automatically — surface the error to the operator.

---

## Logging

All HITL events are written to the audit ledger:

| Event | Fields |
|---|---|
| `approval_created` | `task_id`, `action`, `severity`, `requester`, `channel` |
| `approval_approved` | `task_id`, `approver`, `latency_seconds` |
| `approval_rejected` | `task_id`, `rejecter`, `reason` |
| `approval_timeout` | `task_id`, `waited_seconds` |
| `approval_bypassed` | `task_id`, `bypass_role` |

---

## Configuration Reference

```yaml
hitl:
  default_severity: medium        # low | medium | high | critical
  timeout_seconds: 300           # auto-reject after this many seconds
  channels:
    dashboard: true
    slack:
      webhook_url: "${SLACK_WEBHOOK_URL}"
    email:
      smtp_host: "${SMTP_HOST}"
      smtp_port: 587
      from: "molecule-hitl@yourorg.com"
      to: ["admin@yourorg.com"]   # fallback if no per-user address
  bypass_roles:
    - hitl_bypass
    - admin
```

---

## Release Process

When releasing a new version of molecule-hitl:

1. Update `plugin.yaml` version to match the tag
2. If skill prompts changed, update `skills/hitl-gates/skill.yaml`
3. If hook logic changed, update `hooks/<runtime>-hitl-hook.ts`
4. Run integration tests (requires a running platform):
   ```bash
   pytest tests/ -v
   ```
5. Tag: `git tag vX.Y.Z && git push --tags`
6. Update all org templates that pin this plugin to the new version
7. Validate: restart a dev workspace with the new plugin and trigger an
   approval to confirm notification channels work end-to-end
