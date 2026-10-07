---
title: Player hitboxes
summary: Bullet hits on players use capsules along the live skeleton, the same idea as CS2, instead of the stock bone boxes.
date: 2026-10-07
area: both
---

Stock Call of Duty 4 answers a bullet with `DObjTraceline` against baked boxes on the bones. Those boxes stay blocky when the player crouches, leans, or aims, and a shot can count on a corner that is only the box, not the body.

Player hits now test capsules. A capsule is a cylinder with a hemisphere on each end, swept between two bones. It turns with the skeleton, so the volume follows the pose.

![Capsule hitboxes, front view]({{ '/assets/hitbox-v2.svg' | relative_url }})

The picture is the standing pose in the same colors the client draws when `cg_playerHitboxColor` is `0`. Numbers are the radius in game units.

## What gets traced

Each segment is one capsule. The server, the listen-server hook, and the debug draw share one table.

| Segment | Bones | Radius |
| --- | --- | --- |
| Head | `J_Head` → `J_Helmet` | 5.0 tube, lower end past the chin |
| Neck | `J_Neck` → `J_Head` | 3.0 |
| Upper chest | `J_Clavicle_*` → `J_Shoulder_*` | 2.0 |
| Waist | `J_Hip_LE` → `J_Hip_RI` | 4.0 |
| Upper arm | shoulder → elbow | 2.2 at the shoulder, 3.1 at the elbow |
| Forearm | elbow → wrist | 2.7 |
| Hand | wrist → finger | 2.2, tapering to 1.15 |
| Thigh | hip → knee | 3.9 |
| Shin | knee → ankle | 3.1 |
| Foot | ankle → ball | 2.3, tapering to 2.0 |

The chest is not one box. Four capsules run up the spine from `J_SpineLower` through `J_SpineUpper` to `J_Spine4`, with radii 4.8, 5.0, 5.2, and 4.8, the same tube as the waist. The lower ones are pulled slightly toward the belly so the waist is not a hole between the hips and the chest. The first of those four is a stomach hit. The other three are upper torso.

Hands and feet taper, the same way CS narrows the end of a limb. The heel of each boot is tucked 0.6 units back into the shin capsule so the sole stays on the ball of the foot and the heel does not stick out as its own target.

`J_Ball_*` is not posed by the stock trace. Its position is the parent bone plus that child's offset from the model, so the foot capsule still has an end point.

## Where it runs

`sv_capsuleHitboxes` defaults to `1`. Set it to `0` and players fall back to the stock bone boxes. `sv_capsuleHitboxScale` multiplies every radius. The default is `1`, and the range is `0.5` to `2`.

A listen server traces inside the game executable, so the dedicated-server replacement never loads there. The client hooks that same trace and runs the capsules against the skeleton it is already drawing. Players do not fall back to the boxes on a listen server.

The hit group written into the trace is the normal CoD location: head, neck, upper or lower torso, and each arm, hand, leg, and foot. Damage and hitmarkers still use those groups.

## Seeing them

`cg_drawPlayerHitboxes` is a cheat cvar. It draws only on a `devmap` server or while a demo is playing.

- `0` off
- `1` wireframe
- `2` solid

`cg_playerHitboxColor` `0` is the per-part colors in the picture, `1` is white, `2` is red. The debug draw does not apply `sv_capsuleHitboxScale`. The trace does.
