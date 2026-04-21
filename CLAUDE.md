# molecule-hitl — Molecule AI Plugin

Human-in-the-loop gates for any async callable. Wraps `builtin_tools/hitl.py`: `@requires_approval` decorator, `pause_task`/`resume_task` tools, multi-channel notification.

## Overview

HITL plugin activates the decorator pattern for specific roles. Supports `langgraph`, `claude_code`, and `deepagents` runtimes.

## Build and Test

```bash
python3 .molecule-ci/scripts/validate-plugin.py
pip install -r .molecule-ci/scripts/requirements.txt
```

## Project Structure

```
plugin.yaml             # Plugin manifest
SKILL.md                # agentskills.io spec
.molecule-ci/
  scripts/
    validate-plugin.py  # plugin.yaml validator
    requirements.txt
runbooks/
  local-dev-setup.md
```

## Plugin Manifest

```yaml
name: molecule-hitl
version: 1.0.0
description: Human-in-the-loop gates for any async callable
runtimes: [langgraph, claude_code, deepagents]
skills: [hitl-gates]
```

## Pre-commit Checklist

```bash
python3 .molecule-ci/scripts/validate-plugin.py && \
python3 -c "
import re, sys
with open('plugin.yaml') as f:
    content = f.read()
patterns = [r'sk.ant', r'ghp.', r'AKIA[A-Z0-9]']
if any(re.search(p, content) for p in patterns):
    print('FAIL: possible credentials found')
    sys.exit(1)
print('No credentials: OK')
" && echo "All checks passed"
```

## Release

Version bumps in plugin.yaml → tag → platform registry publishes.