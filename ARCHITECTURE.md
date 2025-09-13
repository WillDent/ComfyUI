# Architecture and Dependency Management

## Overview
ComfyUI is a Python-based application for building and running stable diffusion workflows. The core code is executed by `main.py`, which configures runtime options and delegates requests to an asynchronous server.

## Python Backend and Execution
- `main.py` reads command-line arguments, configures model directories, and applies user-specified paths. It sets environment variables for devices and loads custom nodes before starting the main modules.
- The prompt execution and HTTP interface are provided by `server.py`, which uses `aiohttp` and `websockets` to communicate with the frontend and manage a queue of jobs.

## Model and Asset Management
- Paths for models and other resources are centralized in `folder_paths.py`, which sets up a `models` directory with subfolders for checkpoints, VAEs, LoRAs, diffusion models, etc., and defines output, temp, input, and user directories.
- Users can extend or override these locations using an `extra_model_paths.yaml` configuration. The sample file shows how additional paths can be declared for multiple UIs or shared model storage.

## Python Environment and Dependencies
- The project requires Python 3.9 or newer and defines its packaging metadata in `pyproject.toml`.
- All runtime dependencies—including PyTorch, transformers, and server libraries—are listed in `requirements.txt` for easy installation via `pip`.

## Running ComfyUI Locally
- The README provides several paths for end users: a pre-built desktop application, a Windows portable package, and manual installation. Manual setup involves cloning the repository, placing model files under `models/`, installing dependencies with `pip install -r requirements.txt`, and running `python main.py`.
- GPU-specific instructions are included to guide users in installing the appropriate PyTorch build for AMD, Intel, or NVIDIA hardware.

## Desktop Builds and OS-Specific Dependencies
- A dedicated **ComfyUI Desktop** project assembles platform binaries from the stable core release so that end users receive a ready-to-run package built with the latest code.
- Prebuilt desktop installers are published for Windows and macOS, bundling Python and required libraries so users only need to supply model files.
- Windows users can alternatively download a portable archive that contains a full Python environment; they extract it, place checkpoints under `ComfyUI\models\checkpoints`, and run the included launcher.
- Manual installation covers all operating systems and GPU types, but some platforms require special PyTorch builds—for example, a Metal-enabled nightly on Apple Silicon or `torch-directml` for AMD GPUs on Windows.

## Summary
Through a combination of a Python backend, an asynchronous web server, and a configurable model directory structure, ComfyUI packages complex AI dependencies in a way that allows non-developers to run advanced diffusion workflows locally. Users can either download a ready-to-run application or manually install Python and the required libraries to execute the system on their own hardware.

