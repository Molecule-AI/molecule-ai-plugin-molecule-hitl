# Local Development Setup

This runbook covers setting up a local development environment for
`molecule-hitl`.

---

## Prerequisites

- Python 3.11+
- `gh` CLI authenticated
- Write access to `Molecule-AI/molecule-ai-plugin-molecule-hitl`

---

## Clone & Bootstrap

```bash
git clone https://github.com/Molecule-AI/molecule-ai-plugin-molecule-hitl.git
cd molecule-ai-plugin-molecule-hitl
```

---

## Validating Plugin Structure

```bash
# YAML structure
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"
echo "plugin.yaml OK"
```

---

## Testing HITL Gates Locally

The `builtin_tools/hitl.py` harness wrapper is not in this repo — it is
provided by the Molecule AI platform at runtime. To test:

1. Install the plugin in a test workspace:
   ```bash
   mol workspace plugin install molecule-hitl --workspace <test-wsid>
   ```

2. Configure channels in workspace `config.yaml`:
   ```yaml
   hitl:
     enabled: true
     channels:
       dashboard: true
       slack: false
       email: false
     default_policy: warn
   ```

3. Trigger an approval-required action and verify the notification appears

4. Approve/deny and verify the task resumes or remains paused

---

## Troubleshooting

### Approval request not appearing

- Check `hitl.enabled: true` in workspace `config.yaml`
- Verify the harness provides `builtin_tools/hitl.py`
- Check Slack webhook is set if `channels.slack: true`

### Task hangs after approval

- The `@requires_approval` decorator may not be connected to the harness
- Verify the workspace runtime supports LangChain primitives (required by hitl.py)

---

## Related

- `builtin_tools/hitl.py` — the platform-provided HITL implementation
- `skills/hitl-gates/SKILL.md` — skill documentation
