---
title: Engine improvements
summary: The game simulation now runs at 128 Hz, and player interpolation on the server and client follows that tick instead of the old 50 ms step.
date: 2026-10-07
area: client
---

Call of Duty 4 simulates the world at 20 Hz. One server frame is 50 ms, and a lot of movement code still assumed that number. Raising the tick without changing interpolation makes other players stutter, because the client keeps drawing them as if the next snapshot were still 50 ms away.

## 128 Hz simulation

The server tick is now 128 Hz, about 7.8 ms per frame instead of 50 ms. Player commands, acceleration, and collision run on that shorter step, so movement is no longer advanced in 50 ms chunks.

The rate is `sv_fps`. A matching client asks for snapshots at the same rate with `snaps 128`, and sends input often enough with `cl_maxpackets 128`. Both cvars already accept 128 as the upper limit.

## Server

When `g_smoothClients` is on, each player entity is sent as a short linear trajectory (`TR_LINEAR_STOP`):

- `trDelta` is the player velocity
- `trTime` is the command time
- `trDuration` is one server frame, `1000 / framerate`

At 20 Hz that window is 50 ms. At 128 Hz it shrinks with the tick, so the origin is only extrapolated across the frame that was actually simulated.

## Client

Other players are drawn between two snapshots. `CG_InterpolateEntityPosition` blends origin, view angles, movement direction, and lean by `frameInterpolation`, from the snapshot you already have to the one that just arrived.

Stock remote players use `TR_INTERPOLATE`. That type holds the last origin until the next snapshot, then pops. If the server already put a velocity in `trDelta`, the client treats the segment as `TR_LINEAR_STOP` and sets its length from the real gap between those two snapshots (`nextSnap->serverTime - snap->serverTime`). The old 50 ms fallback is only used when that gap is missing. Remote players then move across the actual 128 Hz snapshot interval instead of a hardcoded 20 Hz step.
