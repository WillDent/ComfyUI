# Desktop Installer Specification

## Scope
Defines expectations for the official ComfyUI Desktop installers published for Windows and macOS, clarifying what runtime components must be bundled and how launchers should invoke the Python backend.

## Target Platforms
- The README lists the desktop application as the primary onboarding path for both Windows and macOS users, highlighting it as the easiest option for non-technical audiences.【F:README.md†L41-L44】

## Bundled Runtime Requirements
- Installers must embed a compatible Python interpreter (3.12 or newer) and preinstall every package listed in `requirements.txt`, matching the versions that manual installers fetch via pip. This guarantees parity between installer builds and source deployments.【F:requirements.txt†L1-L30】【F:README.md†L245-L252】
- The bundled environment should include the `comfyui-frontend-package` wheel specified in `requirements.txt` because the backend validates the installed version at startup and logs warnings when mismatched.【F:app/frontend_management.py†L1-L81】

## Directory Layout
- Installer payloads should create the canonical folder tree defined in `folder_paths.py` (`models/checkpoints`, `models/vae`, `models/loras`, etc.) plus runtime directories (`input`, `output`, `temp`, `user`) so the UI can run immediately after installation without additional setup.【F:folder_paths.py†L16-L83】
- Extra configuration templates such as `extra_model_paths.yaml.example` may be shipped alongside the executable to help users remap model folders or share assets across UIs.【F:extra_model_paths.yaml.example†L1-L47】

## Launcher Behavior
- Native wrappers ultimately execute `python main.py` inside the bundled environment. Launchers should forward CLI flags that desktop users commonly toggle (e.g., `--auto-launch`, `--listen`, `--port`) and respect the runtime initialization sequence documented in `main.py` so progress reporting, custom node startup scripts, and prompt queue threading behave identically to manual runs.【F:main.py†L25-L350】
- Windows installers may optionally expose `--windows-standalone-build` to trigger updater maintenance before startup, ensuring the packaged updater binaries remain current.【F:main.py†L303-L308】

## Post-Install User Steps
- After installation the only required user action is to copy model weights into the appropriate subfolders (checkpoints, VAEs, LoRAs, etc.), mirroring the manual instructions documented in the README.【F:README.md†L174-L199】

## Reference Installer Blueprint
Packaging teams can model their bundler after the following pseudocode to ensure parity with the runtime expectations while leaving room for remote model configuration.

```python
from __future__ import annotations

from pathlib import Path

import installer


def build_package(source: Path, target: Path) -> None:
    env = installer.VirtualEnv(python="3.12.7")
    env.install_requirements(source / "requirements.txt")

    layout = {
        "root": target,
        "app": target / "ComfyUI",
        "python": target / "python",
    }

    installer.copy_tree(source, layout["app"], exclude={".git", "output", "temp"})
    installer.embed_python(env, layout["python"])

    for folder in ["input", "output", "temp", "user"]:
        (layout["app"] / folder).mkdir(parents=True, exist_ok=True)

    (layout["app"] / "user" / "remote_endpoints.yaml").write_text(
        "sdxl: https://api.example.com/v1/inference\n"
    )

    installer.create_shortcut(
        name="ComfyUI",
        target=layout["python"] / "python.exe",
        arguments="main.py --auto-launch",
        icon=source / "assets" / "icon.ico",
    )


build_package(Path.cwd(), Path("dist/ComfyUI-Desktop"))
```

Integrate vendor-specific GPU runtimes, notarization, or code signing within this scaffold while keeping the bundled Python and directory layout aligned with `folder_paths.py` so downstream documentation remains accurate.【F:folder_paths.py†L16-L83】【F:main.py†L25-L350】
