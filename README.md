# molecule-hitl — Human-in-the-Loop Gates

Plugin for LangGraph and Claude Code. Gates irreversible or public actions behind
a human approval request, preventing unattended agents from deploying, deleting,
or sending messages without review.

Wraps the `@requires_approval` decorator and `pause_task` / `resume_task` tools
from `builtin_tools/hitl.py`, which are present in every runtime image. This plugin
is the opt-in policy layer that tells an agent *when* to call them.

## What it provides

**Skill:** `hitl-gates` — tells an agent which actions need a gate and how to
configure the decorator or explicit pause/resume form.

**Gated action classes:**
- Deployment (fly deploy, docker push, kubectl apply, Vercel)
- Irreversible filesystem (`rm -rf`, `git push --force`, `DROP TABLE`)
- Public / external messages (GitHub issues/PRs, Slack, email, social media)
- Production mutations (migrations, secret rotation, cache invalidation)
- Cross-workspace destructive actions (deleting other agents' memories, workspaces)

## Usage

See the `hitl-gates` skill for decorator and explicit pause/resume usage patterns.

### Install via org template (org.yaml)
```yaml
plugins:
  - molecule-hitl
```

### Configure in workspace config.yaml
```yaml
hitl:
  channels:
    - type: dashboard          # always on — uses the platform approval API
    - type: slack
      webhook_url: ${SLACK_HITL_WEBHOOK}
  default_timeout: 300
  bypass_roles: [operator]     # roles that skip the gate entirely
```

## License
Business Source License 1.1 — © Molecule AI.
