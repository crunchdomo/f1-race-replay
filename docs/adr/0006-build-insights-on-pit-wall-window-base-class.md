---
id: ADR-0006
title: Build insights on the PitWallWindow base class
status: accepted
date: 2025-11-21
rules:
  - id: insights-subclass-pit-wall-window
    description: Every insight window class subclasses PitWallWindow.
    must:
      every: { path: "src/insights/**", kind: class }
      except: [{ name: "_*" }, { name: "StintBar" }, { name: "LapAxisWidget" }]
      has: { name: "*PitWallWindow" }
      via: INHERITS
---

# Build insight windows on the PitWallWindow base class

Date: 2026-02-17
Status: Accepted

## Context

After the telemetry stream landed (0005), each new insight window repeated
the same boilerplate: create a stream client, wire three Qt signals, track
connection state, count messages, and close the client cleanly. Most
insights are contributed by different people, so the duplication was
drifting in subtle ways and raising the bar for a first contribution.

## Decision

Provide `PitWallWindow` (`src/gui/pit_wall_window.py`), a `QMainWindow`
subclass that owns the stream client and status bar. An insight implements
two methods, `setup_ui()` and `on_telemetry_data(data)`, and nothing else.
Ship a copy-and-rename template at `src/gui/pit_wall_window_template.py`
and a worked example in `src/insights/example_pit_wall_window.py`. New
insights are registered as a button in `src/gui/insights_menu.py`.
`docs/PitWallWindow.md` and `docs/InsightsMenu.md` are the contributor
guides.

## Consequences

- A new insight is one file plus one menu entry. Driver telemetry, lap
  times, track position, tyre strategy, race control feed and sector times
  all arrived this way, several from first-time contributors.
- Every insight pays the full cost of a Qt process and a stream connection,
  even trivial ones. Acceptable for a handful of windows on a desktop.
- The base class is the single place to change if the stream protocol
  changes, which makes 0005's lack of versioning more survivable.
- Insights that need something other than the live frame (whole-session
  lap data, for example) still have to load it themselves via FastF1.
