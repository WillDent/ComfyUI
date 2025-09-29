# Frontend Delivery Specification

## Scope
Describes how ComfyUI packages, validates, and serves its web frontend so desktop bundles, portable archives, and manual installs stay synchronized with backend releases.

## Packaging Strategy
- The compiled frontend is published as the `comfyui-frontend-package` wheel and pinned in `requirements.txt`, ensuring every distribution installs an identical UI build for a given release.【F:requirements.txt†L1-L2】

## Version Enforcement
- At startup `FrontendManager.check_frontend_version()` reads the installed wheel version, compares it to the required version from `requirements.txt`, and logs a structured warning if the runtime is behind, including remediation instructions for missing wheels. Bundled runtimes must therefore ship the correct wheel to avoid first-run warnings.【F:app/frontend_management.py†L19-L81】

## Alternate Frontend Sources
- `FrontendManager` supports specifying `--front-end-version` in the form `owner/repo@tag`, fetching `dist.zip` assets from GitHub releases (stable or prerelease) when requested. This enables installers to prebundle custom frontends while still allowing advanced users to switch builds at runtime.【F:app/frontend_management.py†L82-L170】
- The manager extracts downloaded zips into `web_custom_versions`, making them discoverable without modifying the packaged assets. Desktop builds can pre-populate this directory with curated alternatives if desired.【F:app/frontend_management.py†L171-L214】

## Server Integration
- `PromptServer` chooses between the installed frontend root and any custom override supplied by `--front-end-root`, logging the final path so operators can confirm which UI is being served. Static assets are then delivered via the `/` route with cache disabled to ensure hot updates reach the browser immediately.【F:server.py†L175-L245】

## Feature Flag Negotiation
- Websocket clients send their supported feature flags on the first message; the server records them and replies with server-side capabilities. Frontend releases rely on this negotiation to opt into features like preview metadata without breaking older builds.【F:server.py†L188-L236】

## Reference Frontend Sync Script
Operators who fork the UI can automate wheel promotion and backend validation with the following Python utility.

```python
from __future__ import annotations

import subprocess
from pathlib import Path

REPO = "org/custom-frontend"
TAG = "v1.2.3"


def download_frontend(dist_dir: Path) -> Path:
    asset = f"https://github.com/{REPO}/releases/download/{TAG}/dist.zip"
    archive = dist_dir / "dist.zip"
    archive.write_bytes(subprocess.check_output(["curl", "-L", asset]))
    subprocess.run(["unzip", "-o", str(archive), "-d", str(dist_dir)], check=True)
    return dist_dir / "dist"


def install_frontend(frontend_root: Path) -> None:
    subprocess.run([
        "python", "-m", "pip", "install", "comfyui-frontend-package==1.*", "--upgrade"
    ], check=True)
    custom_root = Path("web_custom_versions") / TAG
    if custom_root.exists():
        subprocess.run(["rm", "-rf", str(custom_root)])
    subprocess.run(["cp", "-R", str(frontend_root), str(custom_root)], check=True)


if __name__ == "__main__":
    dist_dir = Path("build")
    dist_dir.mkdir(exist_ok=True)
    install_frontend(download_frontend(dist_dir))
```

Desktop bundles can call this during CI to stage alternates inside `web_custom_versions` while still relying on the pinned wheel as the default experience, aligning with `FrontendManager`’s discovery logic.【F:app/frontend_management.py†L82-L214】
