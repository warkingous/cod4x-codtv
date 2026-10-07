---
title: Higher default xasset pools
category: Engine improvements
summary: Mods with many models can hit the official CoD4x xmodel pool and fail to load. The default model pool is doubled, and a console command shows how full each pool is.
date: 2026-10-07
area: client
---

## Overview

**The problem.** CoD4x reserves fixed pools for each asset type (models, materials, images, and so on). Heavy mods can run out of model slots at the official default. When the pool is full, new assets do not load cleanly. You could raise limits with `r_xassetnum` on the command line, but there was no higher built-in default and no easy way to see usage.

**What we improved.** If `r_xassetnum` is not set, the client applies custom defaults: the **xmodel** pool goes from **1000** to **2000**. A console command `xassetusage` prints loaded count, pool size, and percent used per type. You can still override everything with `r_xassetnum` when you need more.

| Type | Official default | Our default (no `r_xassetnum`) |
| --- | --- | --- |
| `ASSET_TYPE_XMODEL` | 1000 | 2000 |
| `ASSET_TYPE_MATERIAL` | 2048 | 2000 |

Models are the real raise (2x). Materials sit slightly under the official std count when the custom path runs; use `r_xassetnum` if a mod needs more materials than 2000.

## Stock (official CoD4x)

`XAssetsInitStdCount` sets the baseline pools, including xmodel **1000** and material **2048**. On zone init, if `r_xassetnum` has a string, `DB_ParseRequestedXAssetNum` and `DB_RelocateXAssetMem` resize pools from that string. If `r_xassetnum` is empty, official keeps the std counts and does not relocate. There is no built-in usage dump command in that path.

## What changed

When `r_xassetnum` is empty, `DB_ApplyCustomXAssetDefaults` copies the std table, then sets:

```
XAssetRequestedCount[ASSET_TYPE_MATERIAL] = 2000;
XAssetRequestedCount[ASSET_TYPE_XMODEL]   = 2000;
```

and calls `DB_RelocateXAssetMem()`. When `r_xassetnum` is set, behaviour matches official: parse the string and relocate from that.

`xassetusage` walks the asset entry pool, counts headers by type, and prints for each type with a pool or loaded assets:

```
Asset Type           Loaded     Pool   Used%
...
TOTAL                ...        ...    ...
```

That makes it obvious which pool is near the ceiling before a mod breaks.

<figure class="doc-figure">
  <img src="{{ '/assets/xasset-limits.svg' | relative_url }}" alt="Official xmodel pool of 1000 versus raised default of 2000">
  <figcaption>Left: official default pools. Right: our default without r_xassetnum - models doubled, materials set to 2000.</figcaption>
</figure>
