# Simulate the Safety Car position from the leader

Date: 2026-02-23
Status: Accepted

## Context

Users asked to see the Safety Car on track during neutralised periods. FastF1
provides when a Safety Car period starts and ends (track status code `4`)
but no position telemetry for the car itself, so there is nothing real to
draw. The choices were to show only a status banner, to animate a car along
the track at a guessed speed, or to place it relative to the race leader.

## Decision

Simulate it. In `_compute_safety_car_positions()` in `src/f1_data.py`, place
the Safety Car about 500 metres ahead of the current race leader along the
track reference polyline while track status is `4`. Animate a deploy phase
from the pit lane and a return phase back to it, each about three seconds,
with alpha and phase stored on every frame under a `safety_car` key. Draw it
visibly different from real cars, with a label, so it is never mistaken for
telemetry.

## Consequences

- The visual matches what viewers expect during a Safety Car period without
  any extra data source.
- It is an approximation and must be presented as one. The README says so,
  and the distinct styling is part of the decision, not decoration.
- Because the position is baked into frames at precompute time, old cached
  files have no `safety_car` field and show nothing until rebuilt with
  `--refresh-data`. This is the first concrete instance of the cache
  versioning gap noted in 0003.
- Virtual Safety Car (status `6`/`7`) is handled only by the status banner.
  Extending this approach to it would be a new, separate decision.
