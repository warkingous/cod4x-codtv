---
title: Shootable live frag grenades
category: Engine improvements
summary: A live frag can be shot or meleed. It explodes as a normal frag, the thrower keeps the kill, and the shooter gets score.
date: 2026-10-09
area: server
---

## Overview

**The problem.** A frag in the air or on the ground is a timer. Bullets can touch the model, and nothing happens. You cannot shoot it out before it cooks, and a nade sitting in a doorway just waits.

**What we improved.** Live frags have hit points. A bullet or a melee that spends that health detonates the nade on the spot, with the normal frag blast. Anyone the blast kills is the thrower's kill. The player who popped it gets score, the same size as an assist, and the matching rank XP. Smoke, flash, and other projectiles stay as they were.

| Cvar | Default | Effect |
| --- | --- | --- |
| `g_shootableGrenades` | `1` | `1` = frags take bullet and melee damage. `0` = stock |
| `g_grenadeHealth` | `50` | Hit points when the frag spawns. Clamped to at least 1 |

## Stock

A thrown grenade is `ET_MISSILE` with a small clip box, so a shot trace can find it. `takedamage` is off, and the grenade handler has no die function, so `G_Damage` returns before health matters. The fuse is the only way it explodes. `frag_grenade_mp` and the short death-perk frag are the same in that regard.

## What changed

On spawn, a frag (`OFFHAND_CLASS_FRAG_GRENADE`, so the normal frag and the short frag) with `g_shootableGrenades` on gets `takedamage` and `health` from `g_grenadeHealth`. Your own frag and a teammate's frag count. Smoke and flash stay undamageable.

Only a direct hit spends that health: pistol bullet, rifle bullet, headshot, or melee. Splash, another grenade, and explosive radius are ignored, so one blast does not pop the frag next to it. Shotgun pellets are separate bullet hits, and a pellet can spend the whole 50 on its own.

At 0 health the nade runs the normal explode path. The radius attacker stays the thrower (`parent`), and the means of death stays grenade splash. The server then notifies the frag with `grenade_shot` and the player who landed the killing hit. Promod gives that player the `grenade_shot` score, 3, and the same amount of rank XP. Kills and deaths are unchanged. If the fuse runs out on its own, there is no notify and no score. If two players shoot the same frag, the hit that crosses 0 is the one that scores.

<figure class="doc-figure">
  <img src="{{ '/assets/shootable-frags.svg' | relative_url }}" alt="A bullet popping a live frag, with the kill on the thrower and score on the shooter">
  <figcaption>A bullet or melee spends the frag's health. The blast kill stays with the thrower. The shooter gets score.</figcaption>
</figure>
