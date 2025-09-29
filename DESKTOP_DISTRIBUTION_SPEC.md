# ComfyUI Desktop Distribution Specification

## 1. Purpose and Scope
This specification describes how ComfyUI is packaged for end users across operating systems, with emphasis on the desktop builds that bundle Python and third-party AI dependencies. It references the core repository to show how runtimes, model assets, and the web frontend are prepared so that non-developers can run ComfyUI locally without manual dependency management.

## 2. Release Channels
ComfyUI coordinates three repositories to publish end-user packages on a roughly weekly cadence. Core releases are tagged here, the ComfyUI Desktop project turns them into platform installers, and the standalone frontend repository ships the web client updates that are merged into the core codebase for each release window.【F:README.md†L112-L126】

## 3. Supported Distributions
### 3.1 Desktop Application Installers
- Distributed via https://www.comfy.org/download for Windows and macOS.
- Bundled installers embed a Python runtime and all required packages so users only provide model files after installation.【F:README.md†L41-L47】【F:README.md†L112-L126】

### 3.2 Windows Portable Package
- Ships as a 7z archive that already contains Python, ComfyUI, and dependencies.
- Users extract the archive, place checkpoints into `ComfyUI\models\checkpoints`, and run the included launcher.
- Model sharing between UIs is enabled by editing the bundled `extra_model_paths.yaml` template.【F:README.md†L168-L181】

### 3.3 Manual Installation
- Supports Windows, Linux, and macOS (including Apple Silicon) with GPU- or CPU-only operation.
- Requires Python 3.12+ (3.13 preferred) and the dependencies listed in `requirements.txt`.
- GPU-specific PyTorch wheels are installed separately per vendor (ROCm for AMD, XPU/IPEX for Intel, CUDA for NVIDIA, torch-directml for Windows DirectML, and vendor-specific extensions for Ascend, Cambricon, and Iluvatar).【F:README.md†L191-L295】【F:requirements.txt†L1-L30】

## 4. Bundled Runtime Composition
Installer and portable builds mirror the dependency manifest shipped in `requirements.txt`, which pins the `comfyui-frontend-package` and other runtime libraries (PyTorch, transformers, aiohttp stack, SQLAlchemy, etc.). This manifest is reused for manual installs via `pip install -r requirements.txt`, ensuring consistency between bundled and source-based deployments.【F:requirements.txt†L1-L30】【F:README.md†L245-L252】

## 5. Frontend Delivery
The web interface is distributed through the `comfyui-frontend-package` wheel. At runtime the backend verifies that the installed frontend matches the required version in `requirements.txt`, and can fetch alternate builds from GitHub releases when requested. Desktop bundles install the correct wheel ahead of time so end users receive a pre-synced frontend.【F:app/frontend_management.py†L1-L78】【F:app/frontend_management.py†L101-L180】

## 6. Model and Asset Layout
### 6.1 Default Directories
`folder_paths.py` defines the canonical folder tree created inside every distribution (`models/checkpoints`, `models/vae`, `models/loras`, `models/diffusion_models`, `models/controlnet`, etc.), alongside user-writable `input`, `output`, `temp`, and `user` directories. These paths are initialized relative to the install root so that portable builds and installers alike can run immediately after extraction.【F:folder_paths.py†L16-L146】

### 6.2 Configurable Overrides
End users can place an `extra_model_paths.yaml` file next to the executable to remap folders (or share assets with other UIs). The template in the repository shows how to point at external checkpoints, LoRAs, or additional custom nodes, and installers may pre-populate this file to reflect bundled defaults.【F:extra_model_paths.yaml.example†L1-L47】

## 7. Runtime Initialization Flow
When a bundled launcher executes `python main.py`, the entry point applies model path overrides, registers default save locations inside the `output` tree, honors CLI flags that redefine input/user directories, and executes optional pre-start scripts for custom nodes. It then configures device visibility (`CUDA_VISIBLE_DEVICES`, `HIP_VISIBLE_DEVICES`, `ONEAPI_DEVICE_SELECTOR`) before importing heavy modules and launching the server loop.【F:main.py†L1-L140】

## 8. Server Responsibilities
The asynchronous server built on `aiohttp` exposes HTTP and WebSocket endpoints, manages the prompt queue, and initializes middleware such as compression, CORS/origin checks, and cache control. It also provisions model and custom-node managers and resolves the packaged frontend root, ensuring desktop builds serve the embedded UI without extra setup.【F:server.py†L1-L200】

## 9. Hardware-Specific Guidance
Desktop documentation directs users to install accelerator-appropriate PyTorch builds when using manual installs or hardware outside the bundled CUDA-targeted binaries. Environment variable tweaks (e.g., ROCm overrides) are also documented so users on unsupported GPUs can still run the application.【F:README.md†L202-L315】

## 10. User Responsibilities After Installation
Regardless of distribution, users must supply their own model weights by copying checkpoints, VAEs, embeddings, and LoRAs into the relevant folders. They can then launch the packaged runtime (`python main.py` or bundled executables) to start the local server that serves the frontend and executes workflows.【F:README.md†L174-L295】【F:main.py†L1-L140】【F:folder_paths.py†L16-L146】

