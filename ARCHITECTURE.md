# Architecture and Dependency Management

This index links to dedicated specification documents that describe each major subsystem involved in launching and operating ComfyUI. Use these specs to understand how the runtime is assembled, how assets are organised, and how the server communicates with bundled frontends.

## Runtime Lifecycle
- [Runtime Initialization Specification](docs/specs/runtime_initialization.md) — step-by-step breakdown of `main.py`, covering CLI parsing, environment setup, custom node loading, device selection, prompt queue threading, and shutdown hygiene.【F:main.py†L1-L369】
- [Prompt Server Specification](docs/specs/prompt_server_spec.md) — explains aiohttp middleware, websocket negotiation, REST endpoints, and how the `PromptQueue` interfaces with the worker thread.【F:server.py†L1-L371】【F:main.py†L168-L243】

## Asset and Model Handling
- [Model and Asset Management Specification](docs/specs/model_asset_management.md) — documents the directory schema defined in `folder_paths.py`, override mechanisms, cache helpers, and how extra model paths integrate into bundled installs.【F:folder_paths.py†L1-L214】【F:extra_model_paths.yaml.example†L1-L47】【F:main.py†L25-L57】

## Python Environment Expectations
- [Python Environment and Dependency Specification](docs/specs/python_environment_spec.md) — details the supported interpreter versions, pinned dependencies, frontend wheel enforcement, and accelerator-specific install paths that all distributions must respect.【F:main.py†L353-L360】【F:requirements.txt†L1-L30】【F:app/frontend_management.py†L1-L81】【F:README.md†L191-L315】
- [Hardware Support Specification](docs/specs/hardware_support_spec.md) — consolidates README guidance for NVIDIA, AMD, Intel, Apple Silicon, DirectML, Ascend, Cambricon, and Iluvatar devices so packages can surface the correct driver and wheel instructions.【F:README.md†L200-L315】

## Installation Pathways
- [Manual Installation Specification](docs/specs/manual_install_spec.md) — describes how to clone, configure, and run ComfyUI from source while mirroring the bundled experience.【F:README.md†L191-L315】【F:main.py†L25-L350】
- [User Responsibilities Specification](docs/specs/user_responsibilities_spec.md) — lists the post-install tasks end users must complete regardless of distribution, such as copying model weights and running bundled updaters.【F:README.md†L174-L200】【F:main.py†L25-L350】

## Frontend Delivery
- [Frontend Delivery Specification](docs/specs/frontend_delivery_spec.md) — covers how the `comfyui-frontend-package` wheel is managed, how alternate frontends are fetched, and how the server serves static assets with feature flag negotiation.【F:requirements.txt†L1-L2】【F:app/frontend_management.py†L19-L214】【F:server.py†L175-L245】

## Reference High-Level Topology
Use this ASCII diagram and accompanying pseudocode as a starting point for new projects inspired by ComfyUI’s architecture.

```
┌────────────┐       ┌─────────────┐       ┌──────────────────┐
│ Frontend   │◀────▶│ Prompt API  │◀────▶│ Execution Workers │
└────────────┘   WS │ (aiohttp)   │ REST │  (local/remote)   │
        ▲           └─────────────┘       └──────────────────┘
        │                  ▲                       ▲
        │                  │                       │
        │                  │                       │
        ▼                  │                       │
┌────────────┐       ┌────────────┐        ┌────────────────┐
│ Model Repo │◀────▶│ Asset Map  │◀──────▶│ Remote Adapters │
└────────────┘       └────────────┘        └────────────────┘
```

```python
def launch_application():
    config = load_config_files()
    registry = build_asset_registry(config.local_paths, config.remote_endpoints)
    prompt_server = build_prompt_server(registry, feature_flags=config.frontend_flags)
    worker_pool = start_execution_workers(prompt_server.queue, registry)
    prompt_server.run(worker_pool)
```

Swap `build_execution_workers` for cloud queues or hosted inference when planning remote-first deployments, and feed the same registry into the frontend API so users can discover both local folders and API-backed models without special casing.
