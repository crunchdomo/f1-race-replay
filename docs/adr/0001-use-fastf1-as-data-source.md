---
id: ADR-0001
title: Use FastF1 as the only telemetry source
status: accepted
date: 2025-11-21
rules:
  - id: fastf1-only-in-f1-data
    description: Only src/f1_data.py may import fastf1; everything else goes through it.
    only:
      from: { path: "src/f1_data.py" }
      to: { kind: unit, name: "fastf1*" }
      via: IMPORTS
---
# Use FastF1 as the only telemetry source

Date: 2025-11-21
Status: Accepted

## Context

The project needs per-car position, speed, gear, DRS, tyre and lap data for
every driver across a whole session, plus event schedules, track status and
race control messages. Formula 1 does not publish an official API. The
options were scraping the live timing feed ourselves, paying for a commercial
data provider, or using FastF1, an open-source Python library that wraps the
F1 live timing archive and the Ergast/Jolpica schedule API.

## Decision

Use FastF1 for all session data. Load sessions through `load_session()` in
`src/f1_data.py` and read everything else (laps, telemetry, weather, track
status, race control) off the FastF1 `Session` object. Enable FastF1's own
HTTP cache before any load so repeat runs do not hit the network.

## Consequences

- Free, well maintained, and already parses the awkward live timing format.
  Most contributors know it, which lowers the bar for pull requests.
- The app is only as good as FastF1's data. Position data is sampled at a
  few hertz and has gaps, so we interpolate (see 0003). Some things are
  simply absent, such as Safety Car GPS (see 0007).
- Breaking changes upstream break us. The README already tells users to
  `pip install --upgrade fastf1` when loading fails.
- First load of a session is slow because FastF1 downloads and parses raw
  timing data. This is what motivated the processed-frame cache in 0003.
