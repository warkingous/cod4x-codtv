---
title: ADS speed while reloading
category: Engine improvements
summary: Stock ADS during a reload keeps the zoom and the hip strafe, about 152. The move scale now also sees the aiming flag, so that strafe drops to the normal ADS speed, about 61.
date: 2026-10-07
area: both
---

## Stock

ADS is supposed to slow you in two steps. Holding aim sets `PMF_SIGHT_AIMING` (`0x10`), which drives the zoom. `PM_UpdatePlayerWalkingFlag` also sets `PMF_WALKING` (`0x40`) when ADS is legal. `PM_CmdScale_Walk` then scales wish speed by `0.4` when walking is set, and by `1.0` otherwise. On an assault rifle that is about 152 strafe from the hip and about 61 while aiming.

Every frame, walking is cleared and then set again. The set path refuses every reload weapon state, `WEAPON_RELOADING` through `WEAPON_RELOADING_INTERUPT` (`0x7` to `0xB`), even when `PMF_SIGHT_AIMING` is already on. The view stays zoomed. Walking stays off. The speed check only looks at walking, so the scale stays `1.0`. You strafe at hip speed with an ADS picture. That is the reload-shot side boost: a corner peek at full hip speed while the sight is already up. Scopes make it obvious. Most assault rifles hide it a little, because their only ADS slowdown is that `0.4`.

<figure class="doc-figure">
  <img src="{{ '/assets/reload-ads-flags.svg' | relative_url }}" alt="Normal ADS has both flags set. Reload ADS keeps aiming and clears walking.">
  <figcaption>Green is a flag that is set. Yellow is sight-aiming, which only drives the view. The empty red bar is walking, cleared for the whole reload. Same ADS request. Left is slow. Right stays at hip speed.</figcaption>
</figure>

<figure class="doc-figure">
  <img src="{{ '/assets/reload-ads-speed.svg' | relative_url }}" alt="Assault rifle strafe bars for hip, ADS, and reload ADS">
  <figcaption>Assault rifle strafe, units per second. Cyan is hip, about 152. Green is a normal ADS, about 61. Red is stock reload ADS: the picture is aimed and the number is still hip.</figcaption>
</figure>

## What changed

The walk scale now treats sight-aiming like walking. The test mask went from `0x40` to `0x50` (`PMF_WALKING` or `PMF_SIGHT_AIMING`). Reload still clears walking, and the zoom still comes from sight-aiming. During a reload that bit stays set, so the `0.4` scale applies and the strafe drops to about 61.

<figure class="doc-figure">
  <img src="{{ '/assets/reload-ads-mask.svg' | relative_url }}" alt="Bit 6 is walking and bit 4 is sight-aiming. The new mask accepts either.">
  <figcaption>Yellow is bit 6, walking, 0x40. Cyan is bit 4, sight-aiming, 0x10. The stock mask 0x40 misses a reload, because only bit 4 is set. 0x50 matches either bit.</figcaption>
</figure>

Pmove is predicted, so both sides use the same mask. The client pokes the stock `and edi, 0x40` at `0x40F14B` so the immediate at `0x40F14D` is `0x50`. The server's inlined walk scale tests `byte [ebp-0x120]` with `0x50`. Hip reload stays about 152. A normal ADS stays about 61.

Stance scale is unchanged. The weapon's extra `adsMoveSpeedScale` still follows the walking flag, so a scope can remain a little faster than a true non-reload ADS. The assault-rifle snap from 152 to 61 is the `0.4`, and that one is in.
