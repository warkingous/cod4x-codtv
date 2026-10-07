---
title: Car explosions through walls
summary: A grenade behind a wall could detonate a car, and the car's own splash then hit players over that wall. The car samples now follow the car's facing, and the splash point is lower.
date: 2026-10-07
area: server
---

Two different blasts were getting through cover. The grenade set the car off. The car's own explosion then damaged the player.

## Stock

### Grenade into the car

Splash on a car (`script_model`) is `G_RadiusDamage`, then `CanDamage`. The engine takes the model's local bounds and slides them to `currentOrigin`. `currentAngles` are ignored, so the box stays aligned to the map axes. That is an AABB.

A car parked at an angle does not fill that box. The box has to cover the diagonal, so it is larger than the body and a corner often sticks through a nearby wall onto the grenade's side. `CanDamage` tests five points, the center and four corners, and one clear line is enough. The grenade is not hitting the car through the wall. It is hitting an empty corner that is already on its side of the wall. The car takes the damage and the destructible script blows it up.

Rebuilding a world AABB after rotating the bounds makes the box larger, not tighter, so the leak stays.

### Car splash into the player

When the car dies, Promod plays the effect on `tag_death_fx`, then calls `radiusdamage` from a second point: `self.origin + (0, 0, 80)`.

The entity origin sits near the ground and the roof is about 50 to 60 units up, so +80 hangs above the wreck and above a lot of nearby cover. `CanDamage` rays from that point clear the top of the wall and hit the player on the other side.

Radius is 250, or 375 for `vehicle_80s_sedan1_*`. Damage is 300 at the center and 20 at the edge.

## What changed

### Grenade into the car

For a `script_model`, the five sample points can come from an oriented box. The eight corners of the local bounds are rotated by `currentAngles` and moved to the origin. The samples are the center of that box and the four corners that face the blast. The line of sight test is the same. The points sit on the car, so there is no empty corner on the near side of the wall. A grenade behind the wall does not get a clear sample, and the car does not explode. A grenade in the open next to the car still does.

`g_radiusDamageAngles` `0` keeps the stock AABB. `1` uses the oriented box. `2` uses it and, with the debug draw on, also draws the old AABB.

### Car splash into the player

In `maps/mp/_destructible.gsc` only the height of the car's splash point changed, from 80 to 60:

```
origin = self.origin + (0, 0, 60);
self radiusdamage(origin, rng, 300, 20, self.damageOwner);
```

The effect stays on `tag_death_fx`. Radius, center damage, and edge damage are the same. A player in the open next to the car still takes the splash. A player crouched behind a wall taller than the car does not, because the rays stop on the wall instead of clearing it.
