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
