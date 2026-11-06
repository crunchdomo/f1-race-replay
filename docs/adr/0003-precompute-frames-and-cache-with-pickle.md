# Precompute fixed-rate frames and cache them with pickle

Date: 2025-12-02
Status: Accepted

## Context

FastF1 returns each driver's telemetry as a separate, irregularly sampled
time series. The replay needs, for every render tick, the position and state
of every car at the same instant, and it needs to seek backwards and forwards
instantly. Computing this on the fly during rendering caused stutter, and
recomputing it on every launch took over three and a half minutes for a race.

## Decision

Before the window opens, resample every driver onto one shared timeline at
25 frames per second (`FPS` in `src/f1_data.py`) and build a list of frame
dicts, one per tick, holding every car's state. Process drivers in parallel
with `multiprocessing.Pool`. Write the result to
`computed_data/<event>_<session>_telemetry.pkl` with pickle, and on later
runs load that file instead of recomputing. The `--refresh-data` flag
forces a rebuild.

## Consequences

- Playback, seeking and speed changes are array lookups, so they are
  effectively free. Load time dropped from about 3m40s to about 60s on the
  first run and to seconds afterwards.
- Pickle was chosen over JSON because it is 10 to 100 times faster for this
  payload and preserves numpy types. The cost is that the files are not
  readable by other tools and are tied to the Python object layout.
- There is no schema version in the cache file. Any change to the frame
  shape (new fields, such as the Safety Car data in 0007) silently produces
  stale replays until the user passes `--refresh-data`. The README warns
  about this. A version stamp is the obvious follow-up.
- Memory use scales with session length times driver count times fields.
  Fine for a race at 25 fps today, but a ceiling to watch if frame rate or
  fields grow.
- `computed_data/` and `.fastf1-cache/` are git-ignored, and their locations
  are user-configurable through the settings manager.
