---
name: agent-plugins-bootstrap
description: >-
  Clone and manage tingaidehua/agent-plugins meta-repo and on-demand plugin-*
  checkouts under ~/workspace/agent-plugins. Use when setting up plugins on a
  new machine, installing into DeepSeek Harness via junction, or deciding which
  plugin repos to download.
---

# agent-plugins bootstrap

## Default

1. Ensure meta: `git clone https://github.com/tingaidehua/agent-plugins.git ~/workspace/agent-plugins`
2. **Do not** clone any `plugin-*` until the user names them.
3. Catalog / deps: `docs/PLUGINS.md`. Procedure: `docs/INSTALL.md`. Rules: `AGENTS.md`.

## When user names plugins

```bash
cd ~/workspace/agent-plugins
git clone https://github.com/tingaidehua/<name>.git <name>
```

Clone dependencies first (e.g. `plugin-cdp-cli` before `plugin-cdp-immersivetranslate`).

## Harness install

Quit Desktop, then `dsh plugin --profile desktop add ~/workspace/agent-plugins/<name>` (junction / link). Never copy trees for day-to-day edits.

## Privacy

Meta and plugins are public. No secrets, PII, or private IPs in commits.
