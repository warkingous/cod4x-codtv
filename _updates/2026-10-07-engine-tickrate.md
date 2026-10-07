---
title: Higher tickrate
category: Engine improvements
summary: Stock simulation is 20 Hz, one 50 ms step. The server and the client now run at 125 Hz, one exact 8 ms step, and interpolation follows that step.
date: 2026-10-07
area: both
---

## Stock

Call of Duty 4 simulates the world at 20 Hz. One server frame is 50 ms. Player commands, acceleration, and collision run in that 50 ms chunk, and a lot of movement code assumed that number.

When `g_smoothClients` is on, each player entity is sent as a short linear trajectory (`TR_LINEAR_STOP`):

- `trDelta` is the player velocity
- `trTime` is the command time
- `trDuration` is one server frame, `1000 / framerate`

At 20 Hz that window is 50 ms, so the origin is extrapolated across a 50 ms frame.

Other players are drawn between two snapshots. `CG_InterpolateEntityPosition` blends origin, view angles, movement direction, and lean by `frameInterpolation`. Stock remote players use `TR_INTERPOLATE`, which holds the last origin until the next snapshot and then pops. Where the client did follow a velocity already stored in `trDelta`, the segment length fell back to a hardcoded 50 ms.

Raising the tick alone leaves that 50 ms assumption in place, so other players stutter: the client still draws them as if the next snapshot were 50 ms away.

## What changed

The server tick is 125 Hz. Server time is an integer number of milliseconds, and 1000 / 125 is exactly 8, so every frame adds 8 ms. Player commands, acceleration, and collision run on that 8 ms step.

The rate is `sv_fps 125`. A matching client uses `snaps 125` and `cl_maxpackets 125`. Promod forces all three.

`trDuration` is still one server frame. At 125 Hz that is 8 ms, so the origin is only extrapolated across the frame that was actually simulated.

On the client, if the server already put a velocity in `trDelta`, the segment is treated as `TR_LINEAR_STOP` and its length is the real gap between the two snapshots (`nextSnap->serverTime - snap->serverTime`). The old 50 ms fallback is only used when that gap is missing. Remote players then move across the actual 8 ms snapshot interval.
