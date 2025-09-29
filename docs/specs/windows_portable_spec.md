# Windows Portable Package Specification

## Scope
Documents the expectations for the Windows portable 7z archive so release engineers can verify that the bundle remains fully self-contained and aligned with core releases.

## Distribution Format
- The README links a direct download to `ComfyUI_windows_portable_nvidia.7z`, positioning the archive as a portable option that tracks the latest commits and runs on NVIDIA GPUs or CPU-only systems.【F:README.md†L168-L176】

## Bundled Contents
- The archive must include a full Python runtime, a preinstalled copy of ComfyUI with dependencies from `requirements.txt`, launch scripts (`run_nvidia_gpu.bat`, etc.), and the default directory tree under `ComfyUI\models` plus `input`, `output`, and `temp` directories so no additional setup is required after extraction.【F:requirements.txt†L1-L30】【F:folder_paths.py†L16-L83】

## Model Placement Guidance
- Documentation instructs users to copy Stable Diffusion checkpoints into `ComfyUI\models\checkpoints` immediately after extraction. The archive should therefore ship with that path pre-created and referenced in any included README or helper scripts.【F:README.md†L174-L180】

## Shared Model Support
- The bundle includes `extra_model_paths.yaml.example` so users can rename it and point ComfyUI at shared model directories across other UIs, avoiding duplication on portable drives.【F:README.md†L178-L180】【F:extra_model_paths.yaml.example†L1-L47】

## Update Mechanism
- Windows standalone launches may pass `--windows-standalone-build`, triggering the updater bridge in `main.py` to refresh the packaged updater binaries. Portable distributions should either expose this flag or document how to run `update\update_comfyui.bat` to stay current between archive refreshes.【F:main.py†L297-L308】
