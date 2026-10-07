---
title: Car explosions through walls
category: Engine improvements
summary: A grenade or car blast could hurt you through a wall. Cars now respect the wall for both lighting the car and the explosion that follows.
date: 2026-10-07
area: server
---

## In plain terms

**The problem.** Two separate cheesy deaths. Toss a nade on your side of a wall next to a parked car, and the car could still blow up. Then the car's own blast could reach you over that same wall even if you were crouched in cover. Cover did not feel like cover.

**What we improved.** The game checks damage against the real shape of the car, not a loose box that sticks through the wall. The car's explosion also starts a bit lower, so rays hit the wall instead of skipping over it. A nade or a player out in the open next to the car still works as before.

## Stock

### Grenade into the car

Splash on a car (`script_model`) is `G_RadiusDamage`, then `CanDamage`. The engine takes the model's local bounds and slides them to `currentOrigin`. `currentAngles` are ignored, so the box stays aligned to the map axes. That is an AABB.

A car parked at an angle does not fill that box. The box has to cover the diagonal, so it is larger than the body and a corner often sticks through a nearby wall onto the grenade's side. `CanDamage` tests five points, the center and four corners, and one clear line is enough. The grenade is not hitting the car through the wall. It is hitting an empty corner that is already on its side of the wall. The car takes the damage and the destructible script blows it up.

<figure class="doc-figure">
  <img src="{{ '/assets/car-aabb-through-wall.svg' | relative_url }}" alt="Stock AABB corner on the grenade side of the wall">
  <figcaption>Cyan is the stock AABB. Red is a clear sample on the grenade's side of the wall. Magenta is a sample the wall blocks. Yellow is the grenade. One clear sample is enough, so the car explodes.</figcaption>
</figure>

Rebuilding a world AABB after rotating the bounds makes the box larger, not tighter, so the leak stays.

<figure class="doc-figure">
  <img src="{{ '/assets/car-aabb-vs-obb.svg' | relative_url }}" alt="World AABB around a rotated car compared with an oriented box">
  <figcaption>Same car. Left, cyan: a world AABB rebuilt after the angles, with more overhang. Right, yellow: the oriented box tight around the model. The samples use the yellow box.</figcaption>
</figure>

### Car splash into the player

When the car dies, Promod plays the effect on `tag_death_fx`, then calls `radiusdamage` from a second point: `self.origin + (0, 0, 80)`.

The entity origin sits near the ground and the roof is about 50 to 60 units up, so +80 hangs above the wreck and above a lot of nearby cover. `CanDamage` rays from that point clear the top of the wall and hit the player on the other side.

<figure class="doc-figure">
  <img src="{{ '/assets/car-splash-z80.svg' | relative_url }}" alt="Side view of the plus 80 splash point clearing a wall">
  <figcaption>Yellow is the car splash point at origin + (0, 0, 80). Cyan is a clear line over the wall into the player. Magenta is a lower point, which stops on the wall.</figcaption>
</figure>

Radius is 250, or 375 for `vehicle_80s_sedan1_*`. Damage is 300 at the center and 20 at the edge.

## What changed

### Grenade into the car

For a `script_model`, the five sample points can come from an oriented box. The eight corners of the local bounds are rotated by `currentAngles` and moved to the origin. The samples are the center of that box and the four corners that face the blast. The line of sight test is the same. The points sit on the car, so there is no empty corner on the near side of the wall. A grenade behind the wall does not get a clear sample, and the car does not explode. A grenade in the open next to the car still does.

`g_radiusDamageAngles` `0` keeps the stock AABB. `1` uses the oriented box. `2` uses it and, with the debug draw on, also draws the old AABB.

<figure class="doc-figure">
  <img src="{{ '/assets/car-obb-fix.svg' | relative_url }}" alt="Oriented box samples all sit behind the wall">
  <figcaption>Yellow is the oriented box and its sample points. Green dots are those samples. Magenta rays stop on the wall, so CanDamage is 0 and the car does not explode. The faint cyan box is the old AABB, no longer used for the samples.</figcaption>
</figure>

### Car splash into the player

In `maps/mp/_destructible.gsc` only the height of the car's splash point changed, from 80 to 60:

```
origin = self.origin + (0, 0, 60);
self radiusdamage(origin, rng, 300, 20, self.damageOwner);
```

The effect stays on `tag_death_fx`. Radius, center damage, and edge damage are the same. A player in the open next to the car still takes the splash. A player crouched behind a wall taller than the car does not, because the rays stop on the wall instead of clearing it.
