---
title: Scoreboard server IP toggle
category: Engine improvements
summary: The scoreboard always showed the server IP. You can hide it by default for privacy, or turn it back on when you need the address.
date: 2026-10-07
area: client
---

## Overview

**The problem.** Opening the scoreboard showed the server IP to everyone watching your screen - streams, demos, screenshots. That leaks an address that can be used for spam or DDoS. Official CoD4x had no way to turn it off.

**What we improved.** `cg_scoreboardShowIP` controls that line. Default is off: the board shows `IP Hidden` instead of the address. Set it to `1` when you want the real IP again. If the server hostname is itself an IP string, that gets masked too while the cvar is off.

| Value | Effect |
| --- | --- |
| `0` (default) | Hides the server IP (privacy / DDoS risk) |
| `1` | Shows the server IP |

## Stock

The scoreboard header draws the host name on the left and `CL_GetServerIPAddress()` on the right. Loopback is labeled `Listen Server`. There was no client cvar to suppress the address. Anyone with tab open, or a recording of tab, could read the IP.

## What changed

`cg_scoreboardShowIP` is a bool, archive, default `0`. In `CG_DrawScoreboardServerNameAddress`:

- `1` - draw the real address (same as stock; loopback still becomes `Listen Server`)
- `0` - draw `IP Hidden` instead

When hidden, if `cgs.szHostName` equals the address or looks like `a.b.c.d`, the hostname is replaced with `Online server` so the IP does not leak through the title either. Layout still sizes the font so hostname and address (or placeholder) fit the scoreboard width.

<figure class="doc-figure">
  <img src="{{ '/assets/scoreboard-show-ip.svg' | relative_url }}" alt="Scoreboard showing IP versus IP Hidden">
  <figcaption>Left: cvar on, address on the right. Right: default off, placeholder instead of the IP.</figcaption>
</figure>
