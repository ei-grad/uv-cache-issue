# setup-uv cache effectiveness issue

Reproduction repository demonstrating that the default `setup-uv` cache configuration still downloads packages from PyPI on every run.

## Problem

With default settings (`enable-cache: true`), the GitHub Actions cache shows "Cache hit" and restores ~434MB, but `uv sync` still downloads all packages from PyPI.

## Root Cause

The default `prune-cache: true` removes **pre-built wheels** before saving the cache, keeping only source distributions. This means:

1. First run: Downloads wheels, saves sdists to cache
2. Second run: Cache hit, but wheels were pruned → downloads wheels again

Additionally, `cache-python` is `false` by default, so Python toolchain (~35MB) is downloaded every run.

## Fix

```yaml
- uses: astral-sh/setup-uv@v5
  with:
    enable-cache: true
    prune-cache: false      # Keep pre-built wheels
    cache-python: true      # Cache Python installations
```

## Evidence

| Configuration | Cache Size | Package Downloads |
|--------------|------------|-------------------|
| Default (`prune-cache: true`) | ~434MB | Every run |
| `prune-cache: false` | ~1.7GB | Only first run |

See [workflow runs](../../actions) for detailed logs.

## Reproduction

1. Run workflow with default settings
2. Second run shows "Cache hit" but still downloads packages
3. Set `prune-cache: false`, bust cache (change deps)
4. Second run no longer downloads packages

## Issue

The default behavior is documented but surprising. Most users expect cached runs to avoid downloads. Consider:
- Changing default to `prune-cache: false`
- Or adding a warning when cache hit + downloads detected
