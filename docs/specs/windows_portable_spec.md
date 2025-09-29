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

## Reference Packing Script
The example below demonstrates how to assemble a portable archive that mirrors the official distribution while leaving hooks for alternative model sources.

```powershell
$ErrorActionPreference = 'Stop'

$root = "C:\\ComfyUI-Portable"
$python = "$root\\python"

New-Item -ItemType Directory -Force -Path $root | Out-Null

Copy-Item -Recurse -Force .\ComfyUI $root
Copy-Item -Recurse -Force .\python_embeded $python

& "$python\\python.exe" -m pip install --upgrade pip
& "$python\\python.exe" -m pip install -r "$root\\ComfyUI\\requirements.txt"

$modelFolders = @(
    "models\\checkpoints",
    "models\\vae",
    "models\\loras",
    "input",
    "output",
    "temp"
)

foreach ($folder in $modelFolders) {
    New-Item -ItemType Directory -Force -Path (Join-Path $root "ComfyUI\\$folder") | Out-Null
}

Set-Content -Path "$root\\ComfyUI\\user\\remote_endpoints.yaml" -Value "sdxl: https://api.example.com/v1/inference"

Compress-Archive -Path "$root\\*" -DestinationPath "ComfyUI_windows_portable_custom.7z"
```

Replace the remote endpoint placeholder with organisation-specific APIs when bundling hybrid local/remote experiences, and ensure the generated archive preserves the folder casing expected by `folder_paths.py` when extracted on Windows.【F:folder_paths.py†L16-L83】
