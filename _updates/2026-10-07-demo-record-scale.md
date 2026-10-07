---
title: Demo recording text size
category: Engine improvements
summary: The on-screen RECORDING line used a fixed size and could sit on top of the HUD. You can scale it down so demos stay obvious without covering ammo and score.
date: 2026-10-07
area: client
---

## Overview

**The problem.** While a demo is recording, the client draws `RECORDING ...` with the demo name and size near the bottom of the screen. That text used a fixed font size. On common HUD layouts it sat over ammo, score, or other bottom elements and got in the way.

**What we improved.** `cg_demoRecordScale` multiplies that line's size. Lower it when the default covers the HUD. `1.0` keeps the stock size. The message and position stay the same - only the scale changes.

## Stock

`SCR_DrawDemoRecording` runs when `clc.demorecording` is set. It builds the string from the demo name and file size in kilobytes, places it at logical `(5, 479)`, and draws with the console font at a fixed normalized scale of `0.333...`:

```
xScale = R_NormalizedTextScale(cls.consoleFont, 0.33333334);
ScrPlace_ApplyRect(...);
R_AddCmdDrawText("RECORDING %s: %ik", ...);
```

There was no cvar to shrink or grow that line. Official CoD4x used the same fixed path.

## What changed

`cg_demoRecordScale` is registered as a float, archive, range `0.5` to `1.0`, default `1.0`. The draw path multiplies the stock normalized scale by that value before `ScrPlace_ApplyRect`:

```
userScale = cg_demoRecordScale;
xScale = R_NormalizedTextScale(cls.consoleFont, 0.33333334) * userScale;
```

`0.5` is half size. `1.0` matches stock. Position stays `(5, 479)`. Show/hide and custom position cvars exist in the source as commented extras; the live client only exposes scale.

<figure class="doc-figure">
  <img src="{{ '/assets/demo-record-scale.svg' | relative_url }}" alt="Stock recording text covering the HUD versus a smaller scaled line">
  <figcaption>Left: stock scale over the bottom HUD. Right: the same line at 0.7, small enough to leave ammo and score readable.</figcaption>
</figure>
