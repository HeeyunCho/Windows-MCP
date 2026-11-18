# Handoff: client fit, idle CPU, flash and screenshot memory (2026-11-18)

## Branches (fork `HeeyunCho/Windows-MCP`)

| Branch | Upstream PR (CursorTouch/Windows-MCP) | Contents |
|---|---|---|
| `main` | #628 | Control threads block instead of polling (idle CPU 3.4% → ~0.2%), batch-cap rewake |
| `client-fit` | #629 | Server instructions, Snapshot `maxResultSizeChars`, lazy dxcam/posthog, README/manifest |
| `python-pin` | #631 | `.python-version` 3.14, `python-preference = "managed"`, `.gitattributes` LF, free-threaded startup guard |
| `cheap-flash` | #632 | Edge-strip glow, cached blur, 16 opacity levels, one-level-per-frame cap, typed helpers |
| `screenshot-memory-release` | #633 | Drop screenshot after encode, dxcam idle release (`WINDOWS_MCP_DXCAM_IDLE_SECONDS`, 30 s, clamped), release at exit |
| `client-fit-and-agent-skills` | none | All of the above plus agent-skills docs; dxcam is lazy here |

PR branches are cherry-picked onto upstream `main` (`61d38b1`, includes #630). Every fix is also on `client-fit-and-agent-skills`.

## Open findings (not fixed yet)

1. **High, #632:** the per-frame cap also limits the fade-out. With frames slower than ~76 ms the region flash lags and snaps from up to ~96/255 to 0 at time-out, and the fade-in may not reach 255. Fix: cap only the rise (`min(level + 1, target)`) and add a 110 ms slow-frame test case.
2. **Medium, #633:** the atexit subprocess test strips named env vars; `PYTHONPROFILEIMPORTTIME=1` still fails it and `PYTHONINSPECT=1` hangs it. Fix: run the child with `python -E` and put `src` on `sys.path` in the child code.
3. Issue #1 (dxcam misses monitors attached after import): `ready-for-human`, needs a second display.
4. Issue #2 (COM state at exit) is fixed by `56271bf` on #633; close it after review.

## Working notes

- No `upstream` remote: `git fetch https://github.com/CursorTouch/Windows-MCP.git main:refs/remotes/cursortouch/main`.
- PR branches still carry upstream's `3.14.7` pin, so `uv run` fails there. Test with the main checkout's venv and the worktree's source: `PYTHONPATH=src C:/Orca_Project/Windows-MCP/.venv/Scripts/python.exe -m pytest <files> -q -p no:cacheprovider`.
- The shared venv imports `windows_mcp` from the main checkout unless `PYTHONPATH=src` is set.
- Write repo files as LF (Python text mode on Windows produces CRLF).
- Upstream has lint debt; run ruff on changed files only.
- Known flaky under load: `test_powershell_detached_child.py::test_returns_when_the_process_exits[3]`, `test_click_modal_dialog.py::test_hover_only_reports_no_warning` (needs a real cursor).
