# Python Environment and Dependency Specification

## Scope
Outlines the Python runtime expectations for ComfyUI across manual installs, portable bundles, and desktop installers, highlighting how dependencies are versioned and how users tailor accelerator libraries per platform.

## Supported Python Versions
- The launcher warns when running under Python 3.9 or 3.10 but recommends 3.12+, with explicit messaging when it detects interpreters older than 3.10. Desktop bundles therefore embed Python 3.12 or newer to avoid the warning path.【F:main.py†L353-L360】
- Manual installation guidance states that Python 3.13 is fully supported while 3.12 remains an alternative for compatibility with older custom nodes.【F:README.md†L191-L199】

## Dependency Manifest
- `requirements.txt` pins the shipped frontend (`comfyui-frontend-package`), workflow templates, embedded docs, and core libraries including PyTorch, torchvision, torchaudio, transformers, safetensors, aiohttp, SQLAlchemy, and optional extras such as kornia and spandrel.【F:requirements.txt†L1-L30】
- Bundled runtimes mirror this manifest so end users never have to resolve Python packages manually; manual installers execute `pip install -r requirements.txt` to match the bundled environment.【F:README.md†L245-L252】

## Frontend Package Enforcement
- At startup the backend checks that the installed `comfyui-frontend-package` version matches the requirement file and surfaces warnings when the wheel is missing or outdated, ensuring desktop bundles ship the correct web UI snapshot.【F:app/frontend_management.py†L1-L81】

## Accelerator-Specific Guidance
- Windows, Linux, and macOS users follow vendor-specific PyTorch install commands: ROCm 6.4 wheels for AMD, XPU/IPEX binaries for Intel, CUDA 12.9 wheels for NVIDIA, and torch-directml for DirectML fallback. Each command is documented so manual setups or custom bundles can substitute the appropriate wheel before launching the UI.【F:README.md†L193-L270】
- Additional sections cover Ascend NPUs, Cambricon MLUs, and Iluvatar accelerators, directing users to vendor documentation before running `python main.py`. Bundled distributions can reference these notes when providing platform-specific installers.【F:README.md†L272-L295】

## Runtime Launch Commands
- Regardless of distribution, users ultimately run `python main.py` (or a wrapper) after installing dependencies. Environment variables like `HSA_OVERRIDE_GFX_VERSION` and `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL` provide optional tuning knobs for AMD GPUs, which installers may expose through helper scripts.【F:README.md†L296-L315】
