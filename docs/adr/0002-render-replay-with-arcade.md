---
id: ADR-0002
title: Render the replay with Arcade
status: accepted
date: 2025-11-21
scope:
  - { path: "src/interfaces/**" }
---

# Render the replay window with Arcade

Date: 2025-11-21
Status: Accepted

## Context

The core feature is a smooth animation of twenty cars moving around a track
at up to 256x real time, with a leaderboard, HUD and keyboard controls drawn
on top. This needs a real-time render loop, not a plotting library. Options
considered were matplotlib animation, pygame, Arcade (built on pyglet and
OpenGL), and a Qt canvas.

## Decision

Render the replay window with Arcade. `F1RaceReplayWindow` in
`src/interfaces/race_replay.py` subclasses `arcade.Window` and draws the
track, cars and HUD in `on_draw`. Qualifying and practice replays follow the
same pattern in their own window classes.

## Consequences

- GPU-accelerated drawing with a simple Python API. Shapes, text and sprites
  are enough for the whole HUD, so no custom OpenGL has been needed.
- Arcade owns the main thread and its event loop. Anything else that wants
  an event loop, such as a Qt window, cannot run in the same process. This
  is the root reason the insights windows live in separate processes
  (see 0004 and 0005).
- Rendering cost grows with every HUD element, and lag on low-end machines
  is a known problem noted in `roadmap.md`. Heavy work must stay out of
  `on_draw`.
- Contributors need to learn Arcade's coordinate system and camera to touch
  the replay window.
