# claude-auto-screenshot

Window-aware screen capture skill for Claude Code on Windows hosts. Enumerate open windows, target by title/PID/HWND, crop to client area. Multi-monitor aware, handles minimized + GPU-accelerated windows.

Originally shipped 2026-04-16 as `smart-screenshots` inside `claude-code-optimizations`; extracted and renamed `claude-auto-screenshot` as part of the modular reorg. v2 (raise-without-focus + capture-overlay confirmation + audit fixes) is in flight — see `docs/specs/2026-04-17-v2-design.md` and `docs/plans/2026-04-17-v2-implementation.md`.

**Requires:** Windows host with PowerShell 5+; WSL access to `powershell.exe` (if running from WSL); Pester v5 for tests.

See `setup.md` to install.

**Historical context:** `docs/specs/2026-04-16-v1-design.md` + `docs/specs/2026-04-17-v2-design.md` + `docs/specs/2026-04-17-v2-followups.md` (designs), `docs/plans/2026-04-16-v1-implementation.md` + `docs/plans/2026-04-17-v2-implementation.md` (plans), `docs/changelog-original.md` (the changelog from when this lived in claude-code-optimizations).
