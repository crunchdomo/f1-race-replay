# Architecture Decision Records

Short records of the significant decisions behind this project, in the
[Nygard format](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions):
Context, Decision, Consequences.

Records 0001–0007 were written retroactively in November 2026 from the code
and git history. Dates are when the decision landed in `main`, not when the
record was written.

Add a new record by copying the shape of an existing one, using the next
number and a present-tense verb phrase for the file name. Do not edit an
accepted record to change the decision; write a new one and mark the old one
`Superseded by`.

| # | Decision | Status |
|---|----------|--------|
| [0001](0001-use-fastf1-as-data-source.md) | Use FastF1 as the only telemetry source | Accepted |
| [0002](0002-render-replay-with-arcade.md) | Render the replay window with Arcade | Accepted |
| [0003](0003-precompute-frames-and-cache-with-pickle.md) | Precompute fixed-rate frames and cache them with pickle | Accepted |
| [0004](0004-use-pyside6-for-menus-and-insights.md) | Use PySide6 for menus and insight windows | Accepted |
| [0005](0005-stream-telemetry-over-local-tcp.md) | Stream telemetry to other processes over a local TCP socket | Accepted |
| [0006](0006-build-insights-on-pit-wall-window-base-class.md) | Build insight windows on the PitWallWindow base class | Accepted |
| [0007](0007-simulate-safety-car-position.md) | Simulate the Safety Car position from the leader | Accepted |
