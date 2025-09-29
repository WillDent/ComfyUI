# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ComfyUI is a node-based visual AI engine for stable diffusion and other AI models. It uses a graph/flowchart interface where users create workflows by connecting nodes. The application runs as a Python web server with a TypeScript/Vue frontend.

## Core Architecture

### Execution Flow
- **Entry point**: `main.py` initializes the application, loads custom nodes, and starts the server
- **Server**: `server.py` implements the aiohttp web server and WebSocket API for real-time communication
- **Execution engine**: `execution.py` processes workflow graphs, manages node execution, and handles caching
- **Node definitions**: `nodes.py` contains built-in node implementations (samplers, loaders, image processing, etc.)

### Key Directories
- `comfy/`: Core library containing model management, samplers, and diffusion implementations
- `comfy_execution/`: Workflow execution engine (graph processing, caching, validation, progress tracking)
- `comfy_api/`: Versioned API system for node development with backward compatibility
- `comfy_api_nodes/`: API-based nodes for external services
- `comfy_extras/`: Additional node implementations (hypernetworks, LoRA, upscaling, etc.)
- `app/`: Application layer (user management, frontend management, logging, settings)
- `api_server/`: REST API route handlers
- `custom_nodes/`: User-installed extensions (see `custom_nodes/example_node.py.example`)
- `models/`: Default location for AI models (checkpoints, VAEs, LoRAs, etc.)
- `web/`: Compiled frontend assets from the separate frontend repository

### Model Management
- `folder_paths.py` manages model search paths and file discovery
- `extra_model_paths.yaml.example` shows how to configure additional model directories
- Models are organized by type: checkpoints, vae, loras, text_encoders, diffusion_models, clip_vision, controlnet, upscale_models, etc.

### Custom Node System
- Custom nodes extend ComfyUI with new functionality
- Located in `custom_nodes/` directory
- Can include `prestartup_script.py` for initialization
- Use `comfy_api` versioned system for compatibility (see `comfy_api/version_list.py`)

## Development Commands

### Running ComfyUI
```bash
python main.py
```

Common command-line arguments:
- `--listen 0.0.0.0` - Listen on all interfaces
- `--port 8188` - Set server port (default: 8188)
- `--preview-method taesd` - Enable high-quality latent previews
- `--cpu` - Run on CPU only (slow)
- `--directml` - Use DirectML for AMD cards on Windows
- `--cuda-device 0` - Specify CUDA device
- `--auto-launch` - Automatically open browser

For all available options, see `comfy/cli_args.py` or run `python main.py --help`

### Installing Dependencies
```bash
pip install -r requirements.txt
```

For GPU-specific installations:
- **NVIDIA**: `pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu129`
- **AMD (Linux)**: `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.4`
- **Intel**: `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/xpu`
- **Apple Silicon**: Follow PyTorch nightly installation for Metal support

### Testing
```bash
# Run all tests
pytest

# Run unit tests only
pytest tests-unit/

# Run integration tests
pytest tests/

# Run specific test markers
pytest -m "not inference"  # Skip inference tests
pytest -m "not execution"  # Skip execution tests

# Run specific test file
pytest tests-unit/comfy_test/test_file.py
```

Test configuration is in `pytest.ini`. Tests are split into:
- `tests/`: Integration tests (execution, inference, comparison)
- `tests-unit/`: Fast unit tests

### Linting
```bash
# Check code with Ruff
ruff check .

# Auto-fix issues
ruff check --fix .
```

Ruff configuration is in `pyproject.toml`. Enabled rules focus on syntax errors, undefined names, and suspicious patterns.

## Important Patterns

### Node Implementation
Nodes inherit from `ComfyNodeABC` and use the versioned API system. The API provides backward compatibility across versions (v0_0_1, v0_0_2, latest). See `comfy_api_nodes/` for examples.

### Execution System
- Workflows are represented as `DynamicPrompt` objects containing nodes and connections
- The execution engine in `execution.py` processes nodes in dependency order
- Caching system in `comfy_execution/caching.py` avoids recomputing unchanged nodes
- Progress tracking via `comfy_execution/progress.py` for real-time updates

### Model Loading
- Models are loaded lazily through `comfy.model_management`
- Smart memory management offloads models between GPU/CPU as needed
- Supports multiple precision modes (fp16, fp32, bf16, fp8) via command-line args

### WebSocket Protocol
- Binary event types defined in `protocol.py`
- Server pushes execution progress, previews, and results to connected clients
- Client sends workflow execution requests via REST API

## Frontend Development

The frontend is maintained separately at https://github.com/Comfy-Org/ComfyUI_frontend. The `web/` directory contains compiled releases updated fortnightly. To use the latest frontend:

```bash
python main.py --front-end-version Comfy-Org/ComfyUI_frontend@latest
```

Report frontend issues at the separate frontend repository, not the core repository.

## Release Process

ComfyUI follows a weekly release cycle (typically Friday). Three interconnected repos:
1. **ComfyUI Core** (this repo): Stable releases (e.g., v0.7.0)
2. **ComfyUI Desktop**: Builds desktop app using latest stable core
3. **ComfyUI Frontend**: Weekly updates merged into core; frontend frozen for each release

## Notes

- Only workflow parts with correct inputs and outputs are executed
- Unchanged parts of workflows are not re-executed (smart caching)
- Workflows can be saved/loaded as JSON
- Generated PNGs contain embedded workflow metadata (can be dragged onto interface)
- Use `(word:1.2)` for emphasis in prompts, `{day|night}` for dynamic prompts
- Textual inversions go in `models/embeddings/` and are used as `embedding:filename.pt`