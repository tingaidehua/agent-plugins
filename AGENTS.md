# AGENTS.md — agent-plugins

Meta-workspace that **owns and constrains** public Harness plugin repositories.
Each `plugin-*` folder is its **own git repo**, published to GitHub as a **public** repository under the same account.

Do not treat this folder as a pnpm monorepo. There is no root `package.json`. Scripts live inside each plugin.

## Hard rules (always apply)

1. **Everything under `plugin-*` is open source / public.** Maximize generality: no machine-specific paths as required config, no single-user assumptions, no private forks, no closed blobs.
2. **No privacy data in any plugin tree.** Forbidden before commit/push:
   - API keys, tokens, cookies, passwords, private keys, `.env` with values
   - Personal notes, chat logs, trajectories, session dumps, browser profiles
   - Real IPs / hostnames that identify a private network (use placeholders like `127.0.0.1`, `example.com`, env vars)
   - Phone numbers, emails, WeChat IDs, real names, home paths with usernames when avoidable
3. **Config lives outside the repo.** Prefer `~/.env`, `process.env`, or `~/.dsh`. Ship `.env.example` with **key names only**, never values.
4. **Cursor must scan for leaks** before claiming a plugin change is done (see `.cursor/rules/privacy-public-scan.mdc`).
5. **One plugin = one git repo = one GitHub public repo.** Parent `agent-plugins` only holds governance docs/rules and lists plugins; it must not swallow plugin history as a monorepo.
6. **Desktop install = junction / `link:`** to `C:\workspace\agent-plugins\plugin-*` (never a copied snapshot).

## Layout

```
agent-plugins/                          # meta repo (governance only)
  AGENTS.md
  docs/
  .cursor/rules/                        # Cursor constraints
  plugin-cdp-cli/                       # own git → GitHub public
  plugin-cdp-immersivetranslate/        # own git → GitHub public
  plugin-hermes-bridge/
  plugin-frp/
  plugin-ops-store/
  plugin-deploy-nginx/
  plugin-deploy-web/
  plugin-manage-server/
```

## Naming

| Pattern | Role |
|---------|------|
| `plugin-*-cli` | Interface / fabric for contributors (no product UI) |
| `plugin-*` | Product / ops / bridge plugins |
| Prefer `plugin-` prefix for **directory name, package.json `name`, and cordis `name`** |

## Desktop install (junction)

Quit DeepSeek Harness Desktop first:

```powershell
$dsh = "$env:LOCALAPPDATA\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd"
& $dsh plugin --profile desktop add C:\workspace\agent-plugins\plugin-cdp-cli
& $dsh plugin --profile desktop add C:\workspace\agent-plugins\plugin-cdp-immersivetranslate
& $dsh plugin --profile desktop add C:\workspace\agent-plugins\plugin-ops-store
& $dsh plugin --profile desktop add C:\workspace\agent-plugins\plugin-frp
& $dsh plugin --profile desktop list
```

Bundle order for CDP translate: **`plugin-cdp-cli` before `plugin-cdp-immersivetranslate`**.

## Per-plugin smoke

```powershell
node C:\workspace\agent-plugins\plugin-cdp-cli\scripts\smoke.mjs
node C:\workspace\agent-plugins\plugin-cdp-immersivetranslate\scripts\smoke.mjs
# IMT gates (inside plugin-cdp-immersivetranslate):
#   node scripts/frontend-sweep.mjs && node scripts/aside-gate.mjs && ...
```

## ~/.env keys (examples only)

| Key | Purpose |
|-----|---------|
| `DSH_CDP_ENDPOINTS` | Comma-separated ports or base URLs |
| `DSH_CDP_DEFAULT_PORT` | Default attach port |
| `DSH_IMT_STORE` | Optional SQLite path for immersive translate cache |
| `DEEPSEEK_API_KEY` / `DEEPSEEK_BASE_URL` / `DEEPSEEK_MODEL` | Standalone LLM fallback |
