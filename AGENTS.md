# AGENTS.md — agent-plugins

Meta-repo that **governs** public Harness `plugin-*` repositories.
Read [README.md](README.md), [docs/PLUGINS.md](docs/PLUGINS.md), and [docs/INSTALL.md](docs/INSTALL.md) before cloning or installing anything.

## Hard rules

1. **Public by default.** Every `plugin-*` is open source on GitHub. Maximize generality.
2. **No privacy data anywhere in this meta tree** (`.agents`, `.cursor`, `docs`, `AGENTS.md`, `README.md`) **or** in any `plugin-*` working tree. Forbidden: secrets, tokens, cookies, chat logs, trajectories, browser profiles, real LAN IPs / hostnames, PII. Config values live in `~/.env` / `~/.dsh` only; ship `.env.example` with key names, never values.
3. **One plugin = one git repo = one public GitHub repo.** This meta repo must not swallow plugin history.
4. **On-demand clone only.** After cloning `agent-plugins`, **do not** clone any `plugin-*` unless the user **explicitly** names which plugin(s) they need. Default working tree has **zero** plugin checkouts.
5. **Harness install = junction / `link:`** to `~/workspace/agent-plugins/plugin-*` (never a copied snapshot). That is how source edits stay live across machines and sessions.
6. **Scripts live inside each plugin** (`plugin-*/scripts/`). Do not add a root `package.json` or root `scripts/` to the meta repo.

## Bootstrap protocol (for any agent)

```text
1. Ensure meta exists:
     git clone https://github.com/tingaidehua/agent-plugins.git ~/workspace/agent-plugins
   (skip if already present)

2. Ask / wait: which plugin-* does the user need?
   If none named → STOP. Do not download plugins.

3. For each named plugin (see docs/PLUGINS.md for deps):
     cd ~/workspace/agent-plugins
     git clone https://github.com/tingaidehua/<name>.git <name>
   Pull deps first (e.g. plugin-cdp-cli before plugin-cdp-immersivetranslate).

4. Install into Harness via junction / dsh plugin add (docs/INSTALL.md).
   Quit Desktop before changing junctions.
```

Path convention: prefer `~/workspace/agent-plugins`. On Windows that is `$HOME\workspace\agent-plugins`; if the machine already uses `C:\workspace`, keep that convention consistently.

## Layout (meta only)

```
agent-plugins/                 # this git repo
  README.md
  AGENTS.md
  docs/PLUGINS.md              # catalog + relationships
  docs/INSTALL.md              # clone + junction
  .cursor/rules/               # Cursor constraints (no secrets)
  plugin-*/                    # NOT in meta git — cloned on demand
```

## Naming

| Pattern | Role |
|---------|------|
| `plugin-*-cli` | Interface / fabric (no product UI) |
| `plugin-*` | Product, bridge, or ops plugin |

Directory name, `package.json` `name`, and cordis `name` must match.

## Privacy scan before any commit/push

Scan **meta files and every checked-out `plugin-*`** for leaks (see `.cursor/rules/privacy-public-scan.mdc`). Refuse to commit secrets or machine-identifying data.
