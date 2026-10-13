# Acceptance evidence �?immersive translate (dialogue-only)

## Non-negotiable

Webpage translation is accepted **only** when DeepSeek Harness **Desktop chat** calls `immersive_translate` (trajectory shows a durable tool result). Cursor-run `translatePage` / `retranslate-scanning.mjs` does **not** count.

## Implemented

| Item | Status |
|------|--------|
| Required inject `['tools','systemPrompt','llm','cdp']` | done �?tools register synchronously via `createRequire(defineTool)` |
| Hard systemPrompt (order 100) forbids shell/web_* for CDP translate | done |
| Tools: `immersive_translate`, `immersive_translate_set_mode`, `immersive_translate_revise` | done |
| IMT DOM/CSS dual + translation + source | done |
| SQLite cache `~/.dsh/imt-store/imt.sqlite` | done (`node:sqlite`) |
| Lazy viewport extract + IntersectionObserver drain | done |
| Glossary + revise (low-context) | done |
| Unit tests | **pass** 11/11 |

## Desktop checklist (human / agent after reload)

1. Fully quit DeepSeek Harness; confirm junctions:
   - `profiles/desktop/node_modules/plugin-cdp-cli` �?`C:\workspace\agent-plugins\plugin-cdp-cli`
   - `�?plugin-cdp-immersivetranslate` �?`C:\workspace\agent-plugins\plugin-cdp-immersivetranslate`
   - bundles order: `plugin-cdp-cli` then `plugin-cdp-immersivetranslate`
2. Start Desktop.
3. **New** chat: `cdp 9225 翻译 scanning 这个页面`
4. Trajectory must include `immersive_translate` (not shell curl).
5. Optional: `immersive_translate_set_mode` with `mode: translation`.
6. Second run should report high `cacheHits`.

## Prior false positive

Chinese on Scanning from Cursor HTTP smoke (2026-10-02) is **not** dialogue acceptance.
