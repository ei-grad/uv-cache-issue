# setup-uv cache effectiveness issue

Reproduction repository demonstrating that the default `setup-uv` cache configuration still downloads packages from PyPI on every cached run.

## Problem

With default settings (`enable-cache: true`), GitHub Actions shows "Cache hit" and restores ~434MB, but `uv sync` still downloads all packages from PyPI.

## Root Cause

The default `prune-cache: true` removes **pre-built wheels** before saving the cache, keeping only source distributions. From [setup-uv docs](https://github.com/astral-sh/setup-uv/blob/main/docs/caching.md):

> "the uv cache is pruned after every run, removing pre-built wheels, but retaining any wheels that were built from source"

This means:
1. First run: Downloads wheels, saves only sdists to cache
2. Second run: Cache hit, but wheels were pruned → downloads wheels again

## Evidence

From [workflow run #21582182179](https://github.com/ei-grad/uv-cache-issue/actions/runs/21582182179) (second run with cache hit):

### With `prune-cache: true` (default) — [job log](https://github.com/ei-grad/uv-cache-issue/actions/runs/21582182179/job/62181694686)

```
Cache hit for: setup-uv-1-...-with-prune
Cache Size: ~434 MB (455338011 B)

Downloading cpython-3.14.2-linux-x86_64-gnu (download) (34.4MiB)
Downloading numpy (15.8MiB)
Downloading pandas (10.4MiB)
 Downloaded numpy
 Downloaded pandas
Prepared 15 packages in 794ms
Installed 15 packages in 38ms
```

Cache hit, but numpy (15.8MB) and pandas (10.4MB) are downloaded from PyPI.

### With `prune-cache: false` — [job log](https://github.com/ei-grad/uv-cache-issue/actions/runs/21582182179/job/62181694727)

```
Cache hit for: setup-uv-1-...-without-prune
Cache Size: ~1758 MB (1843537026 B)

Downloading cpython-3.14.2-linux-x86_64-gnu (download) (34.4MiB)
Installed 15 packages in 74ms
```

Cache hit, packages installed directly from cache. Only Python toolchain downloaded (not cached by setup-uv).

## Comparison

| Setting | Cache Size | Package Downloads | Time |
|---------|-----------|-------------------|------|
| `prune-cache: true` (default) | 434 MB | numpy, pandas, etc. | ~800ms prepare |
| `prune-cache: false` | 1758 MB | None (only Python) | 74ms install |

## Fix

```yaml
- uses: astral-sh/setup-uv@v5
  with:
    enable-cache: true
    prune-cache: false      # Keep pre-built wheels in cache
```

## Issue

The default behavior (`prune-cache: true`) is documented but surprising:
- Users expect cached runs to avoid downloads
- The cache shows "hit" but downloads happen anyway
- 434MB of cache provides no benefit for pre-built wheels

Consider:
- Changing default to `prune-cache: false`
- Or documenting this trade-off more prominently
- Or showing a warning when cache hit occurs but downloads are still needed
