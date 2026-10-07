---
title: Higher default xasset pools
category: Engine improvements
summary: Heavy mods can run out of model slots at the official CoD4x default. The default xmodel pool is raised from 1000 to 2000.
date: 2026-10-07
area: client
---

## Overview

**The problem.** CoD4x keeps a fixed pool of model slots. Heavy mods with a lot of xmodels can hit the official default of 1000 and fail to load more. You could raise limits with `r_xassetnum`, but the built-in default stayed low.

**What we improved.** When `r_xassetnum` is not set, the default **xmodel** pool is **2000** instead of **1000**. That is the only pool we raise. Other asset types keep the usual CoD4x defaults. You can still override pools with `r_xassetnum` if a mod needs more. The console command `xassetusage` prints how full each pool is (loaded, pool size, percent used).

## Stock (official CoD4x)

`XAssetsInitStdCount` sets `ASSET_TYPE_XMODEL` to **1000**. If `r_xassetnum` is empty, that count stays. Only a custom `r_xassetnum` string relocates the pools. There is no built-in command to dump pool usage.

## What changed

If `r_xassetnum` is empty, after copying the std table we set:

```
XAssetRequestedCount[ASSET_TYPE_XMODEL] = 2000;
```

and call `DB_RelocateXAssetMem()`. Models get twice the default room for heavy mods. Everything else is unchanged from the CoD4x std table. If `r_xassetnum` is set, behaviour matches official.

`xassetusage` walks the asset entry pool and prints per type:

```
Asset Type           Loaded     Pool   Used%
...
TOTAL                ...        ...    ...
```

That shows which pool is near the ceiling before a mod breaks.

<figure class="doc-figure">
  <img src="{{ '/assets/xasset-limits.svg' | relative_url }}" alt="Official xmodel pool of 1000 versus raised default of 2000">
  <figcaption>Official default xmodel pool 1000. Ours 2000 when r_xassetnum is not set.</figcaption>
</figure>
