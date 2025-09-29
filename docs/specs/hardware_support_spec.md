# Hardware Support Specification

## Scope
Summarizes the officially documented procedures for configuring ComfyUI across different accelerator vendors so packaging teams can surface the right guidance during installation.

## NVIDIA GPUs
- Users install CUDA-enabled PyTorch via `pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu129`, with optional nightly wheels for early adopters. Troubleshooting steps advise reinstalling torch when CUDA support is missing.【F:README.md†L227-L244】

## AMD GPUs
- Linux users install ROCm 6.4 wheels (stable or nightly) before launching ComfyUI. Additional environment variables such as `HSA_OVERRIDE_GFX_VERSION` target unsupported RDNA2/RDNA3 cards, while `TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL` and `PYTORCH_TUNABLEOP_ENABLED` can unlock optimizations.【F:README.md†L200-L315】

## Intel GPUs
- Intel Arc users install PyTorch XPU wheels from the dedicated index URL, with guidance to consult Intel’s documentation. Optionally they can leverage Intel Extension for PyTorch for further optimizations.【F:README.md†L221-L226】

## Apple Silicon
- Apple Silicon instructions direct users to install nightly Metal-enabled PyTorch builds from Apple’s guide before following the general manual installation steps and copying models into the appropriate directories.【F:README.md†L255-L264】

## DirectML Fallback
- AMD cards on Windows without ROCm support can use `pip install torch-directml` and launch ComfyUI with `--directml`, though the README labels this path as poorly supported and recommends ROCm when available.【F:README.md†L266-L271】

## Ascend, Cambricon, and Iluvatar Accelerators
- Each vendor requires installing its toolkit and PyTorch extension (torch-npu, torch_mlu, or Iluvatar Corex Toolkit) before running `python main.py`. The README links to vendor-specific installation guides to ensure users meet prerequisites.【F:README.md†L272-L295】

## Reference Device Negotiation Helper
Bundle launchers can detect accelerator availability and select the appropriate CLI flags before invoking `main.py` by using a helper like the following.

```python
from __future__ import annotations

import os
import shutil
import subprocess


def detect_accelerator() -> str:
    if shutil.which("nvidia-smi"):
        return "cuda"
    if shutil.which("rocminfo"):
        return "rocm"
    if shutil.which("intel_gpu_top"):
        return "xpu"
    if os.name == "posix" and subprocess.call(["sysctl", "machdep.cpu.brand_string"], stdout=subprocess.DEVNULL) == 0:
        return "metal"
    if shutil.which("dml_diag"):
        return "directml"
    return "cpu"


def build_launch_command() -> list[str]:
    accelerator = detect_accelerator()
    match accelerator:
        case "cuda":
            return ["python", "main.py"]
        case "rocm":
            return ["python", "main.py", "--hip"]
        case "xpu":
            return ["python", "main.py", "--use-xpu"]
        case "metal":
            return ["python", "main.py", "--use-metal"]
        case "directml":
            return ["python", "main.py", "--directml"]
        case _:
            return ["python", "main.py", "--cpu"]


if __name__ == "__main__":
    print("Launching:", " ".join(build_launch_command()))
```

Mapping detections to the documented CLI switches keeps end-user experiences consistent with the README guidance while allowing distributors to plug in remote inference fallbacks when no supported accelerator is present.【F:README.md†L200-L315】
