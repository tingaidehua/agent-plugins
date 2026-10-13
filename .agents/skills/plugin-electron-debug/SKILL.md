---
name: plugin-electron-debug
description: >-
  DeepSeek Harness Electron desktop + agent-plugins (plugin-cdp-cli / plugin-cdp-immersivetranslate)
  co-debug playbook: restart app, new session, stress prompts, trajectories, CDP screenshots,
  correct Windows junctions, and how to make plugin source edits take effect. Use when
  debugging Harness Desktop, CDP port 9225, FAB/IMT mode bugs, false allPass gates, or
  after renaming dsh-cdp → plugin-cdp-cli.
---

# DeepSeek Harness Electron × 插件联调

## 何时用本 skill

- Electron 桌面版 Harness 与本仓库（`agent-plugins`）联调
- 改了 `plugin-cdp-cli` / `plugin-cdp-immersivetranslate` 源码却「不生效」
- 悬浮球三态（原/双/译）错位、重叠、每次切换结果不同
- 门禁报 allPass 但用户截图仍坏

本 skill 随仓库放在 `.agents/skills/plugin-electron-debug/`，克隆后即可给 agent 使用。下文 `$REPO` = 本仓库根目录。

## 架构速记

```
DeepSeek Harness.exe (Electron)
  └─ 加载外部插件 (junction → $REPO/…)
       ├─ plugin-cdp-cli          # CDP fabric / page algebra
       └─ plugin-cdp-immersivetranslate  # IMT watcher + FAB + 原/双/译
            └─ CDP → Chrome :9225 上的真实标签页
```

插件注入的是**目标网页 DOM**，不是 Harness 自己的 UI。验证必须对着 Chrome 页（或 CDP 截图），不要只看对话里的 “ok”。

---

## 1. 重启 Electron Harness

安装路径（本机常见）：

- 快捷方式：`%USERPROFILE%\Desktop\DeepSeek Harness.lnk`
- 可执行文件：`%LOCALAPPDATA%\Programs\DeepSeek Harness\DeepSeek Harness.exe`

**干净重启（PowerShell）：**

```powershell
Get-Process "DeepSeek Harness" -ErrorAction SilentlyContinue | Stop-Process -Force
Start-Sleep -Seconds 2
Start-Process "$env:LOCALAPPDATA\Programs\DeepSeek Harness\DeepSeek Harness.exe"
```

或双击桌面快捷方式（启动器通常会先杀旧实例）。

**不要**把「重启 Harness」和「重启被测 Chrome（9225）」混为一谈：

| 进程 | 作用 | 何时重启 |
|------|------|----------|
| DeepSeek Harness.exe | 跑 agent / 加载插件 / watcher | 改插件 **Node 入口**、junction、插件注册后 |
| Chrome `--remote-debugging-port=9225` | 被测网页 | 标签挂了、CDP 连不上时 |

改 **in-page** 脚本（`imt-algebra.js` / CSS）可用 CDP **热注入**立刻验证，不必每次杀 Electron；但要让 watcher **新开标签**也带上修复，仍需 Harness 进程加载到新代码（见 §6）。

---

## 2. 新建会话 / 发压测提示词

1. Harness UI 里 **New chat / 新会话**（避免旧会话缓存工具结果与错误假设）。
2. 确认插件已启用：`dsh plugin` 列表里有 `plugin-cdp-immersivetranslate`（及 cli）。
3. 压测提示词示例（按场景选用）：

```
cdp 翻译本页
```

```
把 Crafting Interpreters scanning 页切原/双/译各 10 次，截图对比右栏，不要只报 allPass
```

```
严格对照原文：正文+右栏，检查双语和译文是否与原文布局一致（顺序、重叠、路径条）
```

用户说「悬浮球循环」时：**用真实 FAB click 路径**（`#dsh-imt-fab.click()`），不要只用 `__imtSetMode` 绕过 UI。

---

## 3. 获取轨迹（trajectory / transcript）

| 来源 | 路径 |
|------|------|
| Cursor agent 对话轨迹 | Cursor 项目下 `agent-transcripts/<uuid>.jsonl` |
| IMT 探针产物 | `$REPO/plugin-cdp-immersivetranslate/.probe/*.json` / `*.png`（本地调试，已 gitignore） |
| Harness 安装/运行日志 | `%USERPROFILE%\deepseek-harness-install.txt`；仓库 `logs\web.log`（若用源码启动） |

引用历史对话时用：`[<短标题>](<uuid不含.jsonl>)`。

---

## 4. 「打印」网页版（截图 / 量布局）

对 **9225** 上的标签用 `plugin-cdp-cli` 的 `CdpFabric`：

```js
// 模式切换后
await session.evaluate(makeImtAlgebraModeScript('dual'))
const png = await session.send('Page.captureScreenshot', { format: 'png', fromSurface: true })
// 写入 .probe/strict-dual.png 再 Read 读图（像素级 QA）
```

现成门禁脚本（优先跑这些，别口头 allPass）：

| 脚本 | 用途 |
|------|------|
| `$REPO/plugin-cdp-immersivetranslate/scripts/strict-compare.mjs` | 原/双/译对照正文+右栏 |
| `$REPO/plugin-cdp-immersivetranslate/scripts/fab-x30v2.mjs` | FAB 真实 click ×30 |
| `$REPO/plugin-cdp-immersivetranslate/scripts/shot-repl.mjs` | REPL 侧栏 + chip 重叠 |
| `$REPO/plugin-cdp-immersivetranslate/scripts/aside-gate.mjs` | 侧栏门禁 |

Chrome 要先以调试口打开，例如：`http://127.0.0.1:9225/json/list` 能列出目标页。

---

## 5. 正确 junction（最常见「改了不生效 / 无 FAB」根因）

**事故复盘：** Desktop / 插件目录曾 junction 到已删除的 `dsh-cdp`，watcher 注入失败 → 无悬浮球。

检查：

```powershell
cmd /c "dir /AL $REPO"
# 或查 Harness 插件安装目录里指向本仓库的 link
Get-Item <plugin-install-path> | Format-List FullName, LinkType, Target
```

原则：

1. **目标必须存在**：rename `dsh-cdp` → `plugin-cdp-cli` 后，所有 junction/`package.json` 依赖路径一起改。
2. Windows 用 **junction**（目录）或 pnpm link，不要复制过期副本。
3. 改完 junction 后 **重启 Electron**（§1）。
4. 安装插件用本机绝对路径：

```sh
dsh plugin --profile web add <REPO>/plugin-cdp-immersivetranslate
```

（会连带/需要 `plugin-cdp-cli`。）

---

## 6. 改了插件源码如何确保生效

分两层：

### A. In-page 运行时（代数 / CSS / FAB）

页面里跑的是注入字符串。改 `src/imt-algebra.js`、`imt-dual.css`、`float-ball.js`、`margin-column.js` 后：

1. **热验证（当前标签）：** CDP `makeBootImtAlgebraScript(CSS)` + `makeEnsureFloatBallScript()`（**必须 rebind FAB**，旧节点会留旧 click handler）。
2. **持久验证：** 刷新页面或新开标签，让 watcher 重新 inject；若 watcher 仍加载旧 bundle → 重启 Harness。

### B. Node 侧 watcher / pipeline

改 `watcher.js`、`overlay.js`、`pipeline.js`、`fabric.js`：

1. 确认 junction 指到你正在编辑的目录（§5）。
2. **重启 Electron**，新会话，再触发翻译。
3. 不要假设 Electron 热更新了 `node_modules` 里的 junction 内容——杀进程最稳。

### C. 假生效清单（必查）

- [ ] `typeof window.__imtSetMode` / `__imtClearClipPoison` / `__imtPackAbsoluteAsides` 是新逻辑
- [ ] `#dsh-imt-fab` 是 **rebound** 后的节点（ensure 脚本应 `remove` 再创建）
- [ ] `document.documentElement` 上 `imt-state` 与 `dsh-view` **成对**（dual↔compose，translation↔overlay，source↔base）
- [ ] `.probe` 截图与用户视口一致（同一 URL、滚到同一侧栏）

---

## 7. 防返工：门禁必须量什么

**禁止**只报 `childOv=0` 或「看起来 OK」。至少：

| 门禁 | 原文 | 双语 | 译文 |
|------|------|------|------|
| 正文 EN∩ZH | — | 0 | — |
| aside 内 EN∩ZH | 0 | 0 | 0 |
| aside↔aside | 0 | 0 | 0 |
| `.source-file`↔aside | 0 | 0 | 0 |
| `.source-file`↔chip（同 Y 重影） | 0 | 0 | 0 |
| aside 顺序 vs 原文（按 top 排序的 idx） | 基准 | 同序 | 同序（顶与双语冻结一致） |
| FAB ×30 指纹稳定 | — | 每态唯一 fp | — |

细节与「为何反复返工」见 [rework-lessons.md](rework-lessons.md)。

---

## 8. 推荐联调顺序（最短路径）

```
1. 确认 9225 有目标页 + Harness 在跑
2. junction 指向 plugin-cdp-cli / immersivetranslate（非已删 dsh-cdp）
3. 改源码 → CDP 热 boot+ensure FAB → strict-compare / fab-x30v2
4. 绿了再重启 Harness → 新会话 → 用户提示词压测
5. 用户截图仍坏 → 按截图像素定位（chip / aside / 正文），补门禁，禁止口头修好
```

## 相关路径

- 插件：`$REPO/plugin-cdp-cli`、`$REPO/plugin-cdp-immersivetranslate`
- 探针：`$REPO/plugin-cdp-immersivetranslate/.probe/`（gitignore）
- Electron：`%LOCALAPPDATA%\Programs\DeepSeek Harness\`
- 返工教训：[rework-lessons.md](rework-lessons.md)
