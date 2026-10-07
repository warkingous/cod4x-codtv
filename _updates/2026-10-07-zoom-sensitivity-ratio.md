---
title: Zoom sensitivity ratio
category: Engine improvements
summary: Official CoD4x scaled ADS mouse sensitivity with the wrong gate, so the ratio could stick during zoom-out. It now applies only when you are fully aimed in.
date: 2026-10-07
area: client
---

## Overview

**The problem.** `zoom_sensitivity_ratio` is meant to scale your mouse only while aimed down sights. On the official CoD4x client the check used a weapon-position flag that flips only at the ends of the ADS animation. Fully aimed in worked, but during zoom-out the scaled sensitivity could stay on until you were back at the hip. The feel of the ratio did not match “ADS only.”

**What we improved.** The ratio applies when the ADS blend is essentially full, with a small hysteresis so it does not flicker. Hipfire and most of the zoom-in stay on stock FOV sensitivity. Once you leave full ADS, the ratio turns off cleanly. Default `1.0` still means “no extra scale.”

## Stock (official CoD4x)

Stock CoD4 already lowers mouse scale with FOV via `cg.zoomSensitivity` (`tan(fov/2) / (2/π)` from `CG_UpdateFov`). CoD4x added `zoom_sensitivity_ratio` on top of that in `CG_DrawActive`:

```
FOVSensitivityScale = cg.zoomSensitivity;
if (cg.playerEntity.bPositionToADS == 0)
    FOVSensitivityScale *= zoom_sensitivity_ratio;
```

`bPositionToADS` is not “are we in ADS.” The engine sets it only at the extremes of `fWeaponPosFrac`:

| `fWeaponPosFrac` | `bPositionToADS` | Meaning |
| --- | --- | --- |
| `0.0` (hip) | `1` | next motion is toward ADS |
| `1.0` (full ADS) | `0` | next motion is toward hip |
| between | unchanged | last extreme kept |

So official applies the ratio when the flag is `0`: that is true at full ADS, but also for the whole zoom-out until hip. During zoom-in the flag stays `1`, so the ratio stays off until you hit full ADS. Shellshock was applied only when `shellshock.sensitivity != 0`, so a zero (freeze) value skipped the multiply.

## What changed

The gate is `predictedPlayerState.fWeaponPosFrac` with hysteresis:

- latch **on** when frac ≥ `0.995`
- latch **off** when frac ≤ `0.980`

While latched, `scale *= zoom_sensitivity_ratio`. Hipfire and early zoom-in stay unlatched. Leaving full ADS drops below `0.980` and clears the ratio, so zoom-out does not keep the ADS scale. Shellshock always multiplies. The result is clamped to `[0, 10]` and written to `cl.cgameFOVSensitivityScale`.

Optional `cg_zoomSensitivityAuto` (`0` default) switches the base scale to an OSP2-style model (hipfire fixed at `1`, zoomed by vertical FOV over a stock reference). The ratio latch still uses the same `fWeaponPosFrac` gate in both modes.

<figure class="doc-figure">
  <img src="{{ '/assets/zoom-sens-latch.svg' | relative_url }}" alt="Official bPositionToADS gate versus fWeaponPosFrac hysteresis latch">
  <figcaption>Left: official flag. Ratio on at full ADS and through zoom-out. Right: frac latch. Ratio on near full ADS only, off once you leave it.</figcaption>
</figure>
