# Runtime Initialization Specification

## Scope
This document details how `main.py` prepares the ComfyUI runtime before the server loop starts. It covers CLI parsing, environment configuration, path remapping, custom-node bootstrapping, and loop orchestration so distribution teams can reproduce the exact startup behavior in bundled runtimes.

## Argument Parsing and Logger Setup
- ComfyUI enables CLI argument parsing immediately via `comfy.options.enable_args_parsing()` so every launch honours the shared parser defined under `comfy/cli_args.py`.
- `setup_logger` applies verbosity and stdout redirection flags before any other modules emit logs, ensuring uniform logging whether the app is launched by a bundled executable or a manual `python main.py` run.【F:main.py†L1-L24】

## Environment Guards for Desktop Builds
- When the entrypoint is executed directly, ComfyUI disables Hugging Face telemetry to keep offline packages compliant with privacy-sensitive distribution channels.【F:main.py†L18-L21】
- Windows builds set `MIMALLOC_PURGE_DELAY=0` so the bundled mimalloc allocator releases memory promptly, matching expectations of portable archives that start and stop frequently.【F:main.py†L107-L115】

## Model Path Resolution
- `apply_custom_paths()` loads `extra_model_paths.yaml` from the installation root and any additional YAML files passed through `--extra-model-paths-config`, merging them into the central `folder_paths` registry that the rest of the runtime consumes.【F:main.py†L25-L34】
- Output, input, and user directories are overridden when `--output-directory`, `--input-directory`, or `--user-directory` are provided. The helper also auto-registers checkpoint, CLIP, VAE, diffusion model, and LoRA subfolders beneath the active output directory so save nodes can write assets without extra configuration.【F:main.py†L35-L57】

## Custom Node Pre-Startup Hooks
- Before heavy imports, bundled builds iterate every directory returned by `folder_paths.get_folder_paths("custom_nodes")` and execute `prestartup_script.py` modules unless the user disables them via `--disable-all-custom-nodes`. Execution times are logged, allowing desktop packages to surface failures that would otherwise be silent.【F:main.py†L60-L104】

## Device Selection Policy
- Command-line switches populate accelerator-specific environment variables prior to importing PyTorch. `--default-device` rewrites CUDA and HIP visible device lists to move the preferred GPU to the front. `--cuda-device` pins execution to a single index, while `--oneapi-device-selector` selects Intel GPUs. Deterministic mode enforces `CUBLAS_WORKSPACE_CONFIG` before torch loads so behavior is reproducible across restarts.【F:main.py†L117-L140】

## Module Import Ordering
- The launcher validates that `torch` has not been imported yet (issuing a warning if it has) to prevent environment variables from being ignored. It then loads the execution engine, prompt server, protocol constants, node registry, and version metadata in a controlled order so side effects—like model manager initialization—see the finalized directory layout.【F:main.py†L142-L154】

## Prompt Execution Thread
- `prompt_worker` runs in a dedicated daemon thread. It selects an execution cache policy from CLI flags, executes queued prompts, and coordinates garbage collection, model unloading, and progress reporting back to the server. Bundled runtimes rely on this worker to keep UI interactions responsive even under long-running graph executions.【F:main.py†L156-L234】

## Progress Hook Integration
- `hijack_progress()` installs a global callback that forwards node-level progress (including preview images) to the active websocket client while also updating shared execution state. This hook is registered before the server loop starts so every distribution reports consistent status updates.【F:main.py†L237-L275】

## Temporary and Database Preparation
- `cleanup_temp()` purges the temp directory on each launch to avoid leaking intermediate files across portable sessions.【F:main.py†L277-L280】
- `setup_database()` lazily initializes optional SQL-backed features when dependencies are available, logging actionable errors if bundled runtimes omit required modules.【F:main.py†L283-L289】

## Windows Updater Bridge
- Windows standalone builds call into `new_updater.update_windows_updater()` when `--windows-standalone-build` is present, allowing the shipped updater binaries to refresh themselves before the UI starts.【F:main.py†L297-L308】

## Event Loop and Auto-Launch
- `start_comfyui()` optionally adopts an externally supplied asyncio loop, constructs the `PromptServer`, initialises extra nodes (both custom and API), re-applies saved function hooks, and wires the prompt queue thread before returning the coroutine that starts listening on the configured addresses.【F:main.py†L292-L350】
- When `--auto-launch` is set, desktop builds open the system browser against the resolved host/port combination as soon as the server reports readiness, normalizing the first-run experience across installers.【F:main.py†L333-L347】

## Runtime Version Reporting and Shutdown
- On direct execution the launcher logs the Python interpreter version and the packaged ComfyUI version, warns when running under Python <3.10, and then runs the event loop until interruption. Cleanup always removes the temp directory—even on Ctrl+C—so repeated launches remain deterministic.【F:main.py†L353-L369】
