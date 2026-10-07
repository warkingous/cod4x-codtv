---
title: Car explosions through walls
summary: A destroyed car no longer splashes players through cover. The damage point sat 80 units above the car and cleared walls that were taller than the car. It now sits at 60.
date: 2026-10-07
area: server
---

When a car dies, Promod plays the explosion effect on `tag_death_fx`, then calls `radiusdamage` from a second point. That point used to be `self.origin + (0, 0, 80)`. The entity origin is near the ground and the roof is about 50 to 60 units up, so +80 hangs above the wreck and above a lot of nearby cover. `CanDamage` rays from that hub clear the top of the wall and hit the player on the other side.

The grenade that sets the car on fire is a different blast. It starts at the grenade. The splash that hops the wall is the car's own `radiusdamage`, a moment later.

## What changed

In `maps/mp/_destructible.gsc` the hub is now 60 units up:

```
origin = self.origin + (0, 0, 60);
self radiusdamage(origin, rng, 300, 20, self.damageOwner);
```

The effect tag is unchanged, so the visual blast stays on the wreck. Radius is still 250, or 375 for `vehicle_80s_sedan1_*`. A player in the open next to the car still takes the splash. A player crouched behind a wall taller than the car does not, because the rays stop on the wall instead of clearing it.

This is the script offset. It is separate from the engine check that decides whether a grenade can damage the car itself.
