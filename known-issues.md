# Known Issues

> Living document for molecule-ai-plugin-molecule-hitl.

---

## 1. Approval state is lost on runtime restart

**Severity:** Medium
**Affected versions:** All stable
**Status:** Known; tracked in `#hitl-14`

Approval state is held in the runtime's in-memory store. If the workspace process
is restarted (OOM kill, deploy, manual restart), all pending approvals are dropped.
Agents that were waiting for approval silently resume or re-trigger depending on
the calling code's retry logic.

**Workaround:** Set `hitl.timeout_seconds` conservatively (≤ 60 s) for critical
operations. For long-running approvals, implement idempotency keys in the
upstream tool so a re-trigger after restart is safe.

---

## 2. RBAC role change does not retroactively bypass pending approvals

**Severity:** Low
**Affected versions:** All stable
**Status:** Known; tracked in `#hitl-19`

When a user's role is upgraded to `hitl_bypass` while a pending approval exists,
the in-flight approval is not automatically bypassed — the human must still respond.
The bypass check runs only at task-creation time.

**Workaround:** If urgent: cancel the pending task and re-invoke the tool with
the now-bypassed user. Automated re-check is tracked in `#hitl-19`.

---

## 3. Slack notifications silently skipped when webhook URL is absent

**Severity:** Low
**Affected versions:** All stable
**Status:** Known; tracked in `#hitl-22`

The runtime logs no error when `slack.webhook_url` is absent; it simply omits the
Slack notification and falls back to dashboard. Operators may not notice that
Slack alerts are not being sent.

**Workaround:** Verify `SLACK_WEBHOOK_URL` is set in your environment before relying
on Slack notifications. A startup warning log is planned for a future release.

---

## 4. `resume_task` requires the exact task ID — no partial match or retry

**Severity:** Low
**Affected versions:** `< 1.2.0`
**Status:** Known; tracked in `#hitl-31`

`resume_task` accepts only an exact task ID. If the agent has lost the task ID
(e.g., it was printed to logs but the agent did not capture it), there is no
mechanism to list and resume pending tasks. All pending tasks for a workspace
are invisible to the agent without the exact ID.

**Workaround:** Log or store the task ID returned by `pause_task` before any
potentially-blocking operation. A `list_pending_approvals` tool is planned.

---

## 5. Email channel does not support HTML bodies, only plain text

**Severity:** Low (usability)
**Affected versions:** All stable
**Status:** Known; tracked in `#hitl-40`

Approval emails are sent as `text/plain`. Long approval descriptions may wrap
poorly in email clients that render plain text with proportional fonts.

**Workaround:** Keep approval descriptions short (< 200 chars). Rich-text email
support is tracked in the email channel spec.
