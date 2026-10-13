# Plugin catalog

All repos are **public** under `https://github.com/tingaidehua/<name>`.
Clone into `~/workspace/agent-plugins/<name>` **only when the user asks**.

## Dependency graph

```text
plugin-cdp-cli
    └── plugin-cdp-immersivetranslate   # needs cli first (bundle order)

plugin-ops-store
    └── plugin-frp                      # resolves sibling ops-store when possible

plugin-deploy-nginx                     # standalone (ops)
plugin-deploy-web                       # standalone (ops)
plugin-manage-server                    # standalone (ops)
plugin-hermes-bridge                    # standalone (WeChat ↔ DSH)
```

## Catalog

| Repo | Role | Clone when user wants… | Depends on |
|------|------|------------------------|------------|
| `plugin-cdp-cli` | CDP multi-endpoint fabric (`ctx.cdp`), page algebra | Browser/CDP tooling, extension ports | — |
| `plugin-cdp-immersivetranslate` | Immersive bilingual page overlay (FAB 原/双/译) | Webpage translate on CDP targets | **`plugin-cdp-cli`** |
| `plugin-ops-store` | Shared ops SQLite / resolve helpers | Shared ops state across deploy plugins | — |
| `plugin-frp` | FRP client/tunnel helpers for Harness | FRP / remote tunnel | Prefer sibling `plugin-ops-store` |
| `plugin-deploy-nginx` | Nginx deploy helpers | Nginx deploy workflows | — |
| `plugin-deploy-web` | Web static/site deploy helpers | Web deploy workflows | — |
| `plugin-manage-server` | Server manage helpers | Remote server manage | — |
| `plugin-hermes-bridge` | WeChat ↔ DSH bridge (Hermes) | WeChat inbound/outbound bridge | — |

## Suggested bundles (examples)

**Desktop webpage translate (CDP):**

1. `plugin-cdp-cli`
2. `plugin-cdp-immersivetranslate`

**Ops / tunnel:**

1. `plugin-ops-store`
2. `plugin-frp`
3. optionally `plugin-deploy-*` / `plugin-manage-server` as needed

**WeChat bridge:**

1. `plugin-hermes-bridge`

Never install a plugin that is not cloned under `~/workspace/agent-plugins/`.

## Where details live

- Per-plugin API, smoke scripts, and DEVELOPMENT notes → inside that plugin’s own repo (`README.md`, `DEVELOPMENT.md`, `scripts/`).
- Cross-plugin relationships and “which to download” → **this file**.
- Clone + junction procedure → [INSTALL.md](INSTALL.md).
