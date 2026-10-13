# Architecture ? CDP fabric for a super AI browser

This document is the **unified integration contract** for `agent-plugins`.
Implementers porting Chrome extensions (or attaching arbitrary WebView2 / Electron / Chromium) should treat CDP **targets** as first-class harness contexts.

## Goals

1. **Multi-endpoint**: talk to many CDP bases at once (e.g. Sumatra Web `9225`, AI `9224`, Chrome, Edge).
2. **Extension ports**: re-host Chrome extension *behavior* via content-script injection + background logic in DSH plugins ? not by loading CRX binaries into the repo.
3. **Official Sidebar Browser**: eventual authenticated, target-scoped broker (harness Desktop decision). Contract lives here; Host broker is a later harness PR.
4. **Coexist with `ctx.browserUse`**: that seam is exclusive single-provider MCP automation. `ctx.cdp` is a separate multi-instance fabric.

## Object model

| Type | Meaning |
|------|---------|
| `CdpEndpointRef` | One debugging base (`http-attach` or `sidebar-broker`) |
| `CdpTargetRef` | One page/worker (`id = endpointId:devtoolsId`) |
| `CdpSelector` | Resolve by `port` / `endpointId` / `urlIncludes` / `titleIncludes` / `targetId` |
| `CdpHandle` | Open session + lease |
| `ExtensionRuntime` | Content-script inject, CSS inject, in-page message bus |

**Selector rule:** preserve caller semantics. Zero matches ? hard error. Multiple matches ? hard error unless `allowMultiple: true`.

## Package seams (component mode)

Mirror `@deepseek-ai/dsh-experimental-agent-team-profile`: thin bundles compose rows.

**Taxonomy** (see `.cursor/rules/dsh-plugin-taxonomy.mdc`):

| Pattern | Role |
|---------|------|
| `dsh-xxx-cli` | Interface layer ? **`plugin-cdp-cli`** for Chromium extension ports |
| `dsh-tool-xxx` | Agent tool components |
| `dsh-client-ui-xxx` | Desktop panel / UI components |

```
plugin-cdp-cli                   ? Service/tools fabric (row id: cdp)  [INTERFACE]
plugin-cdp-immersivetranslate    ? bundle patch inserts after cdp     [port; split tool/ui later]
```

Future ports should:

1. Depend on `plugin-cdp-cli` (not the deprecated `dsh-cdp` shim)
2. Use `ExtensionRuntime` + `fabric.open(selector)`
3. Ship `cordis.patch.yml` inserting their row after `cdp`
4. Prefer splitting into `dsh-tool-*` + `dsh-client-ui-*` when UI and tools diverge
5. Document DOM/CSS contracts in `DEVELOPMENT.md` ? page work is **secondary frontend development**

## Secondary development mindset

Page-inject plugins (translate, annotate, ?) are **frontend features on someone else's site**.
Debug with CDP as DevTools: switch source / dual / translation, measure layout, fix like CSS/DOM bugs.
Today prefer Cursor for deep loops; expose the same page hooks (`__dshImtTimeline`, `__imtSetMode`, `__dshAlgebra`) so DeepSeek Harness can drive the same probes via `plugin-cdp-cli`.

## Endpoint adapters

### `http-attach` (implemented)

- Discover: `GET {base}/json/list`
- Connect: target `webSocketDebuggerUrl`
- Works for Chrome, Edge, WebView2 remote debugging, SumatraPDF panels, etc.

### `sidebar-broker` (contract only)

Official Desktop Browser guests intentionally have **no** global remote-debugging port.
Required broker API (to be implemented in `apps/desktop`):

```ts
interface SidebarBrokerAdapter {
  describe(): Promise<CdpEndpointRef>           // status: ok | auth-required
  listTargets(): Promise<CdpTargetRef[]>        // lease-scoped guests only
  attach(targetId: string): Promise<{ wsUrl: string, token?: string }>
}
```

Stub in `plugin-cdp-cli` returns `auth-required` until the Host broker ships.

## Config (`~/.env`)

Never commit secrets. Keys:

- `DSH_CDP_ENDPOINTS=9225,9224`
- `DSH_CDP_DEFAULT_PORT=9225`
- `DEEPSEEK_API_KEY` / `DEEPSEEK_BASE_URL` / `DEEPSEEK_MODEL` (standalone translation)

## Roadmap (super AI browser)

1. **Now**: external CDP fabric (`plugin-cdp-cli`) + first extension port (immersive translate webpage)
2. **Next**: Desktop sidebar broker; map Sidebar tabs ? `CdpTargetRef`
3. **Then**: Chrome API shims (`tabs`, `storage`, `runtime.sendMessage`) on ExtensionRuntime
4. **Later**: unified ?browser context? in session events so models reason over live pages as objects

## Security

Treat every remote-debugging port as full browser control. Do not expose endpoints on non-loopback interfaces without auth.
