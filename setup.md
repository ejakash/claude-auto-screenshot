# auto-screenshot — setup

**Requires:** Windows host, PowerShell 5+, WSL access to `powershell.exe`; Pester v5 for tests (`Install-Module Pester -Force -SkipPublisherCheck -Scope CurrentUser`)

## Per-machine values

- `<REPO_PATH>` — absolute path to this repo's clone (used in the symlink target).

## Files

The skill is symlinked into `~/.claude/skills/screenshot/` so edits in this repo flow through immediately without copy steps.

| File | Purpose |
|---|---|
| `SKILL.md` | Skill definition read by Claude Code (via symlink) |
| `capture.ps1` | PowerShell capture script (main entry point) |
| `tests/capture.Tests.ps1` | Pester v5 unit suite (29 tests covering all pure helpers) |
| `tests/pinvoke.smoke.ps1` | One-off smoke test for Win32 P/Invoke surface |

## Pre-requisite: Install Pester v5

```powershell
powershell.exe -Command "Install-Module Pester -Force -SkipPublisherCheck -Scope CurrentUser"
```

## Install

```bash
# SAFETY: verify it's a real directory or symlink, not unrelated
if [ -L ~/.claude/skills/screenshot ]; then
    rm ~/.claude/skills/screenshot
elif [ -d ~/.claude/skills/screenshot ]; then
    mv ~/.claude/skills/screenshot ~/.claude/skills/screenshot.bak
fi
# Replace <REPO_PATH> with this machine's clone path, e.g. /mnt/d/labs/auto-screenshot
ln -s <REPO_PATH> ~/.claude/skills/screenshot  # <-- edit per machine: repo clone path
```

After this the skill is live; no further deploy step is needed when this repo is updated.

## Verify

Quick smoke (run from WSL):

```bash
powershell.exe -ExecutionPolicy Bypass -File "$(wslpath -w ~/.claude/skills/screenshot/capture.ps1)" -Mode overview
```
Expected: `overview|<path>|1200x675|capture_id=<ID>`

Full verification scenarios:

1. **Backwards compat** — `overview` mode emits `overview|<path>|1200x675|capture_id=<ID>` (byte-identical to legacy output)

2. **Pester unit suite** — 29 tests passing:
   ```bash
   powershell.exe -Command "Import-Module Pester -MinimumVersion 5.0.0; Invoke-Pester -Path '$(wslpath -w ~/.claude/skills/screenshot/tests/capture.Tests.ps1)'"
   ```

3. **`list-windows`** — emits `windows|<N>` header, then N rows with `hwnd/pid/proc/title/rect/client/monitor/state/dpi/focus/zorder` fields:
   ```bash
   powershell.exe -ExecutionPolicy Bypass -File "$(wslpath -w ~/.claude/skills/screenshot/capture.ps1)" -Mode list-windows
   ```

4. **`window` mode** — target a foreground app by partial title:
   ```bash
   powershell.exe -ExecutionPolicy Bypass -File "$(wslpath -w ~/.claude/skills/screenshot/capture.ps1)" -Mode window -WindowTitle '<partial title>' -Best
   ```
   Expect: `window|<path>|<WxH>|capture_id=<ID>|...` with `auto_resolved=yes|<reason>` when heuristics fire.

5. **`window -Region content`** — image excludes title bar; dimensions smaller than full window.

6. **Known-problematic app (Chrome)** — auto-picks `strategy=restore` (not `printwindow`).

7. **Minimized window** — still captures successfully via restore strategy; window returns to minimized state afterward.

8. **`-Format json`** — every mode emits one-line JSON parseable by `jq .`.

9. **`window-crop`** from a `window` capture — percentages are window-relative.

10. **Error paths:** bogus title → `error|reason=no_match|...`; missing `-CaptureId` on window-crop → `error|reason=missing_capture_id`; ambiguous title → `error|reason=ambiguous` plus pre-ranked candidate rows.

## Uninstall

```bash
rm ~/.claude/skills/screenshot  # removes the symlink only; repo files are untouched
```
