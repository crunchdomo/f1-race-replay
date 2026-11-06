---
id: ADR-0005
title: Stream telemetry over local TCP
status: accepted
date: 2025-11-21
rules:
  - id: sockets-only-in-stream
    description: Only the stream service touches sockets.
    only:
      from: { path: "src/services/stream.py" }
      to: { kind: unit, name: "socket" }
      via: IMPORTS
---

# Stream telemetry to other processes over a local TCP socket

Date: 2026-02-06
Status: Accepted

## Context

Insight windows run in separate processes from the replay (see 0002 and
0004), but they must show data that is in sync with the current replay
frame, including live playback position, pause state and seeking. The
options were shared memory, multiprocessing queues or pipes, a message
broker, or a plain socket. The project vision in `roadmap.md` is a personal
"pit wall", which implies arbitrary extra tools, possibly written by users,
subscribing to the replay.

## Decision

The replay process hosts `TelemetryStreamServer` (`src/services/stream.py`),
a TCP server on `localhost:9999`. On every rendered frame it broadcasts the
current frame as one newline-delimited JSON message to every connected
client. Clients use `TelemetryStreamClient`, a `QThread` that reconnects and
emits Qt signals with the parsed data. The stream is one-way; clients cannot
control the replay. The format is documented in `telemetry.md`.

## Consequences

- Any process in any language can subscribe with a socket and a JSON parser.
  This is the main enabler for the insight windows and for third-party
  tools.
- Windows start in any order. Clients retry until the server is up, which is
  why `run_session.py` only needs a short sleep before launching them.
- JSON encoding of a full frame 25 times a second is measurable CPU on the
  replay thread. It has been acceptable so far, but it is the first place to
  look if the replay lags with many windows open.
- Fixed port with no discovery. Two replays at once collide. No
  authentication, which is fine for a loopback-only server but means the
  port must stay bound to `localhost`.
- Messages have no version field. Changes to the frame shape need to be
  additive, or every insight window breaks at once.
