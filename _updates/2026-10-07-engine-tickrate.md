---
title: Engine improvements
summary: The game simulation now runs at 125 Hz, one exact 8 ms step, and player interpolation on the server and client follows that tick instead of the old 50 ms step.
date: 2026-10-07
area: both
---

Call of Duty 4 simulates the world at 20 Hz. One server frame is 50 ms, and a lot of movement code still assumed that number. Raising the tick without changing interpolation makes other players stutter, because the client keeps drawing them as if the next snapshot were still 50 ms away.

## 125 Hz simulation

The server tick is 125 Hz. Server time is an integer number of milliseconds, and 1000 / 125 is exactly 8, so every frame adds 8 ms. Player commands, acceleration, and collision run on that step instead of the old 50 ms chunk.

The rate is `sv_fps 125`. A matching client uses `snaps 125` and `cl_maxpackets 125`. Promod forces all three.

## Server

When `g_smoothClients` is on, each player entity is sent as a short linear trajectory (`TR_LINEAR_STOP`):

- `trDelta` is the player velocity
- `trTime` is the command time
- `trDuration` is one server frame, `1000 / framerate`

At 20 Hz that window is 50 ms. At 125 Hz it is exactly 8 ms, so the origin is only extrapolated across the frame that was actually simulated.

## Client

Other players are drawn between two snapshots. `CG_InterpolateEntityPosition` blends origin, view angles, movement direction, and lean by `frameInterpolation`, from the snapshot you already have to the one that just arrived.

Stock remote players use `TR_INTERPOLATE`. That type holds the last origin until the next snapshot, then pops. If the server already put a velocity in `trDelta`, the client treats the segment as `TR_LINEAR_STOP` and sets its length from the real gap between those two snapshots (`nextSnap->serverTime - snap->serverTime`). The old 50 ms fallback is only used when that gap is missing. Remote players then move across the actual 8 ms snapshot interval instead of a hardcoded 20 Hz step.
