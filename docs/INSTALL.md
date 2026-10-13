# Install — on-demand clone + junction

## 1. Clone meta (once per machine)

```bash
mkdir -p ~/workspace
git clone https://github.com/tingaidehua/agent-plugins.git ~/workspace/agent-plugins
```

At this point there are **no** `plugin-*` folders. That is correct.

## 2. Clone only what the user named

```bash
cd ~/workspace/agent-plugins
git clone https://github.com/tingaidehua/<plugin-name>.git <plugin-name>
```

Examples:

```bash
git clone https://github.com/tingaidehua/plugin-cdp-cli.git plugin-cdp-cli
git clone https://github.com/tingaidehua/plugin-cdp-immersivetranslate.git plugin-cdp-immersivetranslate
```

If a plugin depends on another (see [PLUGINS.md](PLUGINS.md)), clone the dependency first.

Update an already-cloned plugin:

```bash
cd ~/workspace/agent-plugins/<plugin-name> && git pull
```

## 3. Install into DeepSeek Harness via junction / link

**Goal:** profile `node_modules/<plugin-name>` points at the source tree under `~/workspace/agent-plugins/`, so you can edit on any machine and Harness loads live source.

1. Fully quit DeepSeek Harness Desktop.
2. Prefer the official CLI (creates `link:` + junction):

```powershell
$root = Join-Path $HOME 'workspace\agent-plugins'
$dsh  = "$env:LOCALAPPDATA\Programs\DeepSeek Harness\resources\runtime\cli\bin\dsh.cmd"

& $dsh plugin --profile desktop add "$root\plugin-cdp-cli"
& $dsh plugin --profile desktop add "$root\plugin-cdp-immersivetranslate"
& $dsh plugin --profile desktop list
```

3. Manual junction fallback (Windows):

```powershell
$nm = "$env:USERPROFILE\.dsh\profiles\desktop\node_modules"
# If profile lives under OneDrive, use that path instead.
New-Item -ItemType Junction -Force `
  -Path "$nm\plugin-cdp-cli" `
  -Target "$HOME\workspace\agent-plugins\plugin-cdp-cli"
```

4. Ensure profile `dsh.profile.bundles` lists plugins in dependency order (cli before port).

**Do not** `npm pack` / copy trees into the profile for day-to-day development.

## 4. Linux / macOS note

Use `ln -s` (or `dsh plugin add` if the CLI creates a symlink) from the profile `node_modules/<name>` to `~/workspace/agent-plugins/<name>`.

## 5. Verify

```powershell
& $dsh plugin --profile desktop list
# Expect: plugin-* link:…/workspace/agent-plugins/plugin-*
```

After changing Node entrypoints, restart Harness so the process reloads the junctioned source.
