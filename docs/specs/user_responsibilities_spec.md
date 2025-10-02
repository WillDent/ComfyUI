# User Responsibilities Specification

## Scope
Captures the minimal steps end users must perform after installing ComfyUI via any distribution so support teams can provide consistent onboarding guidance.

## Model Asset Provisioning
- Users must supply their own Stable Diffusion checkpoints, VAEs, LoRAs, embeddings, and other models by copying them into the corresponding subdirectories under `models/`. README instructions explicitly call out `models/checkpoints` and `models/vae` as mandatory placements.【F:README.md†L174-L200】

## Optional Path Customization
- Advanced users can rename `extra_model_paths.yaml.example` to `extra_model_paths.yaml` and edit it to point at shared model folders or additional custom node directories. Desktop bundles should highlight this mechanism for users with existing model libraries.【F:README.md†L178-L180】【F:extra_model_paths.yaml.example†L1-L47】

## Launching the Application
- After models are in place, users launch the packaged runtime (either a native executable or `python main.py`). The startup sequence automatically prepares directories, loads custom nodes, and starts the prompt server, so no further manual configuration is required.【F:main.py†L25-L350】

## Keeping Installations Updated
- Portable and installer distributions may provide helper scripts (e.g., `update_comfyui.bat`) or expose the `--windows-standalone-build` flag to refresh bundled components. Users should follow distribution-specific instructions to stay current between official release packages.【F:main.py†L297-L308】

## Reference First-Run Checklist
Distributors can embed an interactive checklist to confirm end users have staged both local and remote assets before starting the server.

```python
from __future__ import annotations

from pathlib import Path

def confirm_first_run() -> None:
    checkpoints = Path("models/checkpoints")
    if not any(checkpoints.glob("*.safetensors")):
        print("⚠️  Add at least one checkpoint to models/checkpoints")

    extra_path = Path("extra_model_paths.yaml")
    if extra_path.exists():
        print("✅ Custom model paths configured")
    else:
        print("ℹ️  Create extra_model_paths.yaml to reference shared or remote stores")

    remote_cfg = Path("user/remote_endpoints.yaml")
    if remote_cfg.exists():
        print("✅ Remote inference endpoints detected")
    else:
        print("ℹ️  Optional: define user/remote_endpoints.yaml for API-based models")


if __name__ == "__main__":
    confirm_first_run()
```

Pairing this checklist with onboarding docs helps non-technical users understand that model files are user-supplied and that optional remote providers require separate credentials, reinforcing the expectations documented in the README.【F:README.md†L174-L200】
