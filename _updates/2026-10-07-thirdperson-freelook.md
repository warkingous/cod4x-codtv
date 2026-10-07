---
title: Spectator third-person free-look
category: Engine improvements
summary: Spectators in third person could only follow the player's facing. Free-look lets you orbit the camera, with an optional CS-style lock that ignores the player's yaw and pitch.
date: 2026-10-07
area: client
---

## Overview

**The problem.** In spectator third person the camera stuck behind whoever you were watching. When they turned, so did your view. You could not look around the fight on your own. Official CoD4x had no free-look orbit for that mode.

**What we improved.** For spectators only, third person can free-look. Toggle third person with the grenade key (usually `G`). Turn on `cg_thirdPersonFreeLook` and mouse orbit moves the camera around the player. Optional `cg_thirdPersonFreeLookCS` picks how the orbit base works.

| Cvar | Description |
| --- | --- |
| `cg_thirdPersonFreeLook` | Free-look orbit in 3rd person - look around instead of locked follow |
| `cg_thirdPersonFreeLookCS` | Only when free-look is on |

| `cg_thirdPersonFreeLookCS` | Behaviour |
| --- | --- |
| `0` (default) | Camera base follows player yaw/pitch |
| `1` | CS-style: orbit locked to fixed world angles |

## Stock

Stock and official CoD4x third person (`CG_OffsetThirdPersonView`) places the camera behind `refdefViewAngles` with range/angle dvars. There is no mouse orbit and no spectator free-look mode. The view always tracks the spectated player's facing.

## What changed

Both cvars are bool, archive, default `0`. Free-look is only active when `cg_thirdPersonFreeLook` is on, you are spectating someone else (not your own dead body alone), and the view is not hard-locked.

While free-look is on, mouse deltas from the usercmd ring accumulate into orbit yaw/pitch (pitch clamped about `-80` to `60`). The camera sits at third-person range along those angles and traces against the world so it does not clip through walls.

`cg_thirdPersonFreeLookCS` only matters with free-look on:

- `0` - base yaw/pitch come from the spectated player each frame; orbit is relative to them, so when they turn the base turns with them
- `1` - base is seeded when you start spectating (or switch target) and stays fixed in world space; orbit is relative to that seed, so their turns do not drag the camera

Changing spectated client resets orbit and reseeds the CS base. Death on yourself without a target, or a locked view, clears free-look for that frame.

<figure class="doc-figure">
  <img src="{{ '/assets/thirdperson-freelook.svg' | relative_url }}" alt="Follow-player freelook base versus CS-style locked base">
  <figcaption>Left: FreeLookCS 0 - orbit rides on the player's facing. Right: FreeLookCS 1 - orbit rides on a fixed world seed.</figcaption>
</figure>
