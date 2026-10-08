---
title: Scoreboard K/D column
category: Engine improvements
summary: The scoreboard showed kills and deaths separately but not the ratio. An optional K/D column shows kills divided by deaths for each player.
date: 2026-10-08
area: client
---

## Overview

**The problem.** Tab showed kills and deaths as separate columns. You had to do the ratio in your head. Official CoD4x had no K/D column on the scoreboard.

**What we improved.** `cg_scoreboardShowKD` adds an optional **K/D** column. Default is off so the board stays stock-looking. Set it to `1` to show the ratio next to the other stats.

| Value | Effect |
| --- | --- |
| `0` (default) | No K/D column |
| `1` | Shows a K/D column |

## Stock

The scoreboard lists the usual columns (name, score, kills, assists, deaths, ping when enabled). There is no ratio field. Official CoD4x draws the same set.

## What changed

`cg_scoreboardShowKD` is int `0`/`1`, archive, default `0`. When on, the layout inserts an extra column (`LCT_EXTRA`) with header `K/D`. For each non-spectator row the value is:

```
kills / max(deaths, 1)
```

printed as two decimal places (e.g. `3.00`). Zero deaths use a divisor of `1` so the number stays defined. Spectators get an empty cell. Works with or without the ping text column.

<figure class="doc-figure">
  <img src="{{ '/assets/scoreboard-show-kd.svg' | relative_url }}" alt="Scoreboard without K/D versus with K/D column">
  <figcaption>Left: default, stock columns. Right: ShowKD 1, K/D next to kills and deaths.</figcaption>
</figure>
