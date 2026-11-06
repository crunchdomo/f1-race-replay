# Use PySide6 for menus and insight windows

Date: 2026-01-20
Status: Accepted

## Context

Beyond the replay canvas, the app needs conventional desktop UI: a session
picker with dropdowns, a settings dialog, and a growing set of data windows
with tables, charts and tabs (the "insights"). Arcade is a game framework and
has no widgets. Building forms and tables from Arcade primitives was slow and
produced poor results. A CLI picker built on `questionary` already existed
but did not scale to the insights.

## Decision

Use PySide6 (Qt for Python) for every non-replay window: the race selection
menu, the settings dialog, the insights menu, and all insight windows.
Keep the CLI picker behind `--cli` as an alternative entry point. Arcade
keeps the replay windows only.

## Consequences

- Rich, native-looking widgets, Qt signals for async plumbing, and
  matplotlib embeds for charts, all with no custom widget code.
- Two UI toolkits in one application. Arcade and Qt each want to own the
  main thread's event loop, so Qt windows cannot run inside the replay
  process. Every insights window is launched as a separate Python process
  with `subprocess` from `src/run_session.py`, and they communicate over a
  socket (see 0005).
- Contributors who only touch insights never need to learn Arcade, and
  vice versa. That split has held up well with outside contributions.
- PySide6 is a heavy dependency and lengthens install time.
