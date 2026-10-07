---
title: Car explosions through walls
summary: Stock car splash starts 80 units above the entity and clears walls taller than the car. The splash point is now 60 units up. The effect and the radius are unchanged.
date: 2026-10-07
area: server
---

## Stock

When a car dies, Promod plays the explosion effect on `tag_death_fx`, then calls `radiusdamage` from a second point: `self.origin + (0, 0, 80)`.

The entity origin sits near the ground and the roof is about 50 to 60 units up, so +80 hangs above the wreck and above a lot of nearby cover. `CanDamage` rays from that hub clear the top of the wall and hit the player on the other side.

The grenade that sets the car on fire is a different blast. It starts at the grenade. The splash that hops the wall is the car's own `radiusdamage`, a moment later. Radius is 250, or 375 for `vehicle_80s_sedan1_*`. Damage is 300 at the center and 20 at the edge.

## What changed

In `maps/mp/_destructible.gsc` only the height of that splash point changed, from 80 to 60:

```
origin = self.origin + (0, 0, 60);
self radiusdamage(origin, rng, 300, 20, self.damageOwner);
```

The effect stays on `tag_death_fx`. Radius, center damage, and edge damage are the same. A player in the open next to the car still takes the splash. A player crouched behind a wall taller than the car does not, because the rays stop on the wall instead of clearing it.

This is the script offset. It is separate from the engine check that decides whether a grenade can damage the car itself.
