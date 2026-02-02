# uv-cache-issue

Reproduction repository for an issue with the official `astral-sh/setup-uv` GitHub Action caching.

## Problem

Even when the cache is hit (shows "Cache restored"), `uv sync` still downloads every package from PyPI on each run.

## Expected Behavior

When cache is restored, packages should be served from the cache without downloading from PyPI.

## Actual Behavior

Every `uv sync` run downloads all packages from PyPI, even with a cache hit.

## To Reproduce

1. Run the workflow once to populate cache
2. Run it again (manually via workflow_dispatch or push a change)
3. Observe that despite "Cache restored" message, all packages are downloaded again

## Dependencies

- requests
- pandas
- numpy
- pyspark

These packages were chosen to make download times visible.
