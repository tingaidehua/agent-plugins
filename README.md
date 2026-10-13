# agent-plugins

Governance meta-repo for **public** DeepSeek Harness plugins.

- This repo holds **rules + catalog + install docs only**.
- Each `plugin-*` is a **separate GitHub public repo**.
- **Default: clone meta only — do not clone any `plugin-*` until the user names one.**

## Quick start (any machine)

```bash
# 1) Meta only (no plugins yet)
mkdir -p ~/workspace
git clone https://github.com/tingaidehua/agent-plugins.git ~/workspace/agent-plugins
cd ~/workspace/agent-plugins

# 2) On demand — only when the user asks for a specific plugin
git clone https://github.com/tingaidehua/plugin-cdp-cli.git plugin-cdp-cli
git clone https://github.com/tingaidehua/plugin-cdp-immersivetranslate.git plugin-cdp-immersivetranslate
```

Windows PowerShell equivalent: use `$HOME\workspace\agent-plugins` (or `C:\workspace\agent-plugins` if that is already your convention).

## Install into DeepSeek Harness = junction / link

Never copy plugin trees into the profile. Always **junction** (Windows) or `dsh plugin … add <path>` so edits in `~/workspace/agent-plugins/plugin-*` take effect live.

```powershell
# Quit DeepSeek Harness first
$root = "$HOME\workspace\agent-plugins"   # or C:\workspace\agent-plugins
$dsh  = "$env:LOCALAPPDATA\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd"

& $dsh plugin --profile desktop add "$root\plugin-cdp-cli"
& $dsh plugin --profile desktop add "$root\plugin-cdp-immersivetranslate"
# …add only the plugins you cloned
& $dsh plugin --profile desktop list
```

## Docs map

| File | Purpose |
|------|---------|
| [AGENTS.md](AGENTS.md) | Hard rules for Cursor / other agents |
| [docs/PLUGINS.md](docs/PLUGINS.md) | Catalog, dependencies, when to download which |
| [docs/INSTALL.md](docs/INSTALL.md) | Clone + junction details |

## Privacy

Everything here is public. No secrets, no PII, no private IPs, no machine-local dumps — including under `.agents`, `.cursor`, and `docs`.
