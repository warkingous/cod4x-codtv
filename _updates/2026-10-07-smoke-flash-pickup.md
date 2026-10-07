---
title: Smoke and flash pickup
category: Engine improvements
summary: You can pick up smoke or flash from a dead player even if your class had the other one. A live thrown grenade no longer shows a fake PICKUP prompt.
date: 2026-10-07
area: both
---

## Overview

**The problem.** Two annoyances. If your loadout was smoke, you could not take a flash off a corpse (and the other way around). And when someone cooked a nade and left it on the ground, the UI could show both “throw back” and “pickup” — even though a live grenade waiting to explode is not loot.

**What we improved.** From a corpse drop you can take smoke or flash either way; picking one replaces the other in your secondary slot. A cooking thrown nade is not a pickup: frag still offers throw back, smoke and flash show no pickup line. Only grenades that fell off a body use PICKUP.

## Stock

A player carries one secondary offhand class, smoke or flash. `BG_PlayerCanPickUpWeaponType` refuses the other class. A corpse can drop either grenade, but if your loadout says smoke you cannot take the flash on the ground, and the other way around.

Thrown grenades are a different entity. A live cook, still waiting to explode, is `ET_MISSILE`. The server fills `throwBackGrenadeTimeLeft` for that missile. A grenade that fell off a dead player is `ET_ITEM`, with the timer at zero. Frag has a throwback path. Smoke and flash do not.

The cursor hint string and the throwback icon are not the same draw. If the client thinks the entity is a pickup while the player state still has a throwback timer, you can see both at once.

## What changed

### Server: either secondary

`g_allowAnyOffhandPickup` defaults to `1`. With it on, `BG_PlayerCanPickUpWeaponType` always allows the weapon. Picking up smoke or flash still keeps one secondary slot: the opposite class is stripped, `offhandSecondary` is set to the one you took, and that weapon becomes the equipped offhand.

`0` keeps the stock loadout check. `toggleoffhand` swaps between smoke and flash when both bits are present and the cvar is on.

Only corpse drops and other `ET_ITEM` pickups give you the weapon. A live `ET_MISSILE` is still a cooking throw, not loot.

### Client: hint matches the entity

`CG_GetWeaponUseString` treats a grenade as live when the hinted entity is `ET_MISSILE`, or when the offhand is frag, smoke, or flash and `throwBackGrenadeTimeLeft` is greater than zero. The timer is authoritative even if the snapshot type is missing or stale, so the throwback icon and a PICKUP line do not stack.

| Entity | Frag | Smoke / flash |
| --- | --- | --- |
| Live throw (`ET_MISSILE`, timer &gt; 0) | THROW BACK | no hint, no icon |
| Corpse drop (`ET_ITEM`, timer 0) | PICKUP | PICKUP |

The hint entity type is read from the next snapshot state for that entity number. It does not fall back to `pose.eType`, which can still hold a value from a previous occupant of the slot.

<figure class="doc-figure">
  <img src="{{ '/assets/pickup-nade-hints.svg' | relative_url }}" alt="Live ET_MISSILE throwback versus ET_ITEM corpse pickup">
  <figcaption>Left: a cooking throw. Frag gets THROW BACK. Smoke and flash get nothing. Right: a drop from a corpse. Frag, smoke, and flash all get PICKUP.</figcaption>
</figure>

<figure class="doc-figure">
  <img src="{{ '/assets/pickup-nade-double-hint.svg' | relative_url }}" alt="Stale entity type causing throwback and pickup at once">
  <figcaption>Yellow is the player-state timer that still means throwback. Red is a bad or missing entity type that used to force the PICKUP string. Both could draw together before the timer was trusted.</figcaption>
</figure>
