# Distribution Channel Specification

## Scope
Explains how ComfyUI coordinates releases across core, desktop, and frontend repositories so packaging teams know when to trigger installer builds and how to keep components in sync.

## Weekly Cadence and Tagging
- The README documents a weekly release target (typically Friday) subject to change for major features or model drops. Core releases are tagged in this repository and act as the reference point for downstream packaging.【F:README.md†L112-L118】

## Desktop Repository Responsibilities
- The dedicated ComfyUI Desktop repository consumes the latest core tag and produces platform-specific installers. It also embeds a complete Python runtime plus dependencies so end users only supply model files after installation.【F:README.md†L120-L126】【F:README.md†L41-L47】

## Frontend Coordination
- The standalone frontend repository merges weekly UI updates back into core ahead of each release while continuing development for the next cycle. Desktop bundles must capture the frontend version locked in `requirements.txt` for the tagged release to avoid mismatches.【F:README.md†L123-L126】【F:requirements.txt†L1-L2】

## Portable Archive Alignment
- The Windows portable archive ships from the same release train as the installers. Because it bundles Python and dependencies, it stays compatible with the tagged frontend and only requires users to extract and launch the packaged runner.【F:README.md†L168-L180】

## Manual Installation as Reference
- Manual installation instructions mirror the bundled dependency set, ensuring that testing a release from source (`pip install -r requirements.txt; python main.py`) reproduces the exact runtime that installers ship. Packaging teams can use manual install smoke tests as a baseline for verifying release quality.【F:README.md†L191-L252】

## Reference Release Workflow
The following GitHub Actions outline keeps desktop installers, portable archives, and documentation synchronized around each tag.

```yaml
name: release-train

on:
  push:
    tags:
      - 'v*'

jobs:
  build-core:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: pytest
      - uses: actions/upload-artifact@v4
        with:
          name: comfyui-core-wheel
          path: dist/*.whl

  build-desktop:
    needs: build-core
    uses: comfyui/desktop/.github/workflows/build.yml@main
    with:
      core-wheel: ${{ needs.build-core.outputs.core-wheel }}

  build-portable:
    needs: build-core
    uses: comfyui/desktop/.github/workflows/portable.yml@main

  publish-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: mkdocs gh-deploy --force
```

Mirroring the weekly cadence documented in the README, this workflow ensures every tagged release drives automated builds for installers and the portable archive while running regression tests against the shared dependency manifest.【F:README.md†L112-L180】【F:requirements.txt†L1-L30】 Adapt the reusable workflows to inject platform-specific steps (e.g., notarizing macOS builds) without diverging from the synchronized release process.
