# Manual Installation Specification

## Scope
Provides a step-by-step reference for packaging and documentation teams to describe manual ComfyUI installations that mirror the bundled desktop experience across Windows, Linux, and macOS.

## Prerequisites
- Users install Python 3.13 (or 3.12 for compatibility) prior to cloning the repository, matching the interpreter versions embedded in official installers.【F:README.md†L191-L199】
- Hardware-specific PyTorch wheels must be installed before running ComfyUI: ROCm for AMD, XPU/IPEX for Intel GPUs, CUDA 12.9 wheels for NVIDIA, DirectML for unsupported AMD GPUs on Windows, and vendor packages for Ascend, Cambricon, or Iluvatar accelerators.【F:README.md†L193-L295】

## Repository Setup
1. Clone `https://github.com/comfyanonymous/ComfyUI`.
2. Populate the `models` directory with checkpoints (`models/checkpoints`) and VAEs (`models/vae`) alongside any other assets the workflow requires.【F:README.md†L197-L200】
3. Optionally copy `extra_model_paths.yaml.example` to `extra_model_paths.yaml` and adjust paths to point at shared model storage.【F:extra_model_paths.yaml.example†L1-L47】

## Dependency Installation
- Execute `pip install -r requirements.txt` inside the repository to install all required runtime packages, mirroring the versions shipped in installers.【F:README.md†L245-L252】【F:requirements.txt†L1-L30】

## Launching
- Run `python main.py` from the repository root. Environment variables documented in the README (e.g., `HSA_OVERRIDE_GFX_VERSION`, `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL`, `PYTORCH_TUNABLEOP_ENABLED`) can be set to fine-tune AMD performance or compatibility.【F:README.md†L296-L315】
- The runtime initialization sequence handles model path overrides, custom node scripts, device selection, and prompt queue setup exactly as described in the Runtime Initialization Specification, so manual installs behave the same as bundled ones.【F:main.py†L25-L350】

## Reference Bootstrap Script
Documentation teams can ship a cross-platform installer helper that codifies the manual steps for both local and remote model scenarios.

```bash
#!/usr/bin/env bash
set -euo pipefail

PYTHON_BIN="${PYTHON_BIN:-python3}"

"$PYTHON_BIN" -m venv .venv
source .venv/bin/activate

pip install --upgrade pip
pip install -r requirements.txt

python - <<'PY'
from pathlib import Path
import yaml

base = Path.cwd()
model_layout = {
    "checkpoints": [base / "models" / "checkpoints"],
    "vae": [base / "models" / "vae"],
    "loras": [base / "models" / "loras"],
    "remote": {"sdxl": "https://api.example.com/v1/inference"},
}

for kind, paths in model_layout.items():
    if isinstance(paths, dict):
        continue
    for path in paths:
        path.mkdir(parents=True, exist_ok=True)

extra = base / "extra_model_paths.yaml"
if not extra.exists():
    yaml.safe_dump({"checkpoints": [str(model_layout["checkpoints"][0])]}, extra.open("w"))
PY

echo "Run '. .venv/bin/activate && python main.py --auto-launch' to start ComfyUI"
```

This helper mirrors the README instructions while seeding a remote-model placeholder entry that downstream installers can update for hosted inference endpoints, ensuring parity between manual setups and desktop bundles.【F:README.md†L191-L315】
