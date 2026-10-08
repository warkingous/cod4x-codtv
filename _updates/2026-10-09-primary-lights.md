---
title: Primary light script control
category: Engine improvements
summary: Map primary lights can be dimmed, recolored, and turned off from GSC with the same calls singleplayer already had.
date: 2026-10-09
area: server
---

## Overview

**The problem.** A lamp on a multiplayer map is baked at compile time. Campaign scripts can call `setlightintensity` on a light and the room goes dark. Official multiplayer GSC has no such methods, so a mod cannot turn a primary light off, change its color, or shrink it while the map is running.

**What we improved.** A map light that Radiant compiled as a primary light accepts the same GSC methods as singleplayer: color, intensity, radius, spot FOV, and exponent. `setlightintensity(0)` turns that light off. The client already draws primary-light entities, so the change shows up with no new cvar.

## Stock

Map load keeps a `light` entity only when it has a `pl#` key. That key is the index into the BSP primary-light lump. Lights without `pl#` are stripped and never spawn.

`SP_light` copies the compiled color, intensity, radius, and cone onto an `ET_PRIMARY_LIGHT` entity and links it. From then on the light is fixed. The multiplayer script VM does not register `getlightcolor`, `setlightintensity`, or the other singleplayer light calls. A mod can find the entity with `getent`, and still cannot change how it looks.

## What changed

The multiplayer script VM registers the singleplayer light methods. They run only on a `light` entity. Any other classname script-errors.

| Method | Arguments | Result |
| --- | --- | --- |
| `getlightcolor` | | Color vector, each channel 0 to 1 |
| `setlightcolor` | color vector | Stores each channel as a byte |
| `getlightintensity` | | Current intensity |
| `setlightintensity` | intensity | `>= 0`. `0` turns the light off |
| `getlightradius` | | Current radius |
| `setlightradius` | radius | `>= 0` and no larger than the radius compiled into the BSP |
| `getlightfovinner` | | Inner spot angle, degrees |
| `getlightfovouter` | | Outer spot angle, degrees |
| `setlightfovrange` | outer, optional inner | Outer FOV from 1 to 120. The cone cannot open wider than the compiled FOV. Inner FOV, if passed, is from 0 up to the outer FOV |
| `getlightexponent` | | Falloff exponent, 0 to 100 |
| `setlightexponent` | exponent | Integer from 0 to 100 |

Radius and spot angle can shrink. They cannot grow past what Radiant compiled, because the light grid and the cone were baked for that size. Color, intensity, and exponent can move inside the ranges above.

Give the light a `targetname` in Radiant, then drive it like any other map entity:

```
lamps = getentarray("lamp", "targetname");
for (i = 0; i < lamps.size; i++)
    lamps[i] setlightintensity(0);

lamp = getent("exit_lamp", "targetname");
lamp setlightcolor((0.2, 0.55, 1));
lamp setlightintensity(1.5);
```

<figure class="doc-figure">
  <img src="{{ '/assets/primary-lights.svg' | relative_url }}" alt="A compiled primary light next to the same light turned off from GSC">
  <figcaption>Left: the light as Radiant compiled it. Right: the same entity after setlightintensity(0).</figcaption>
</figure>
