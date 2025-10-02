# Desktop Startup Sequence Specification

## Scope
This reference walks through the complete launch flow used by the Electron-based ComfyUI Desktop application. It traces execution from the moment the desktop bundler starts its embedded Python runtime until the browser window renders the UI, calling out the checks that guarantee dependencies, assets, and services are ready before the interface appears.

## 1. Bundler Process Bootstraps the Runtime
- Desktop packages embed Python along with every dependency listed in `requirements.txt`, so the backend binaries are available before the Electron shell spawns the interpreter.【F:requirements.txt†L1-L30】
- Launchers invoke the bundled interpreter with `main.py` and distribution-specific flags such as `--windows-standalone-build` and `--auto-launch`, which are defined in the shared CLI parser consumed by `comfy.options` at import time.【F:main.py†L1-L24】【F:comfy/cli_args.py†L33-L105】
- `app.logger.setup_logger` replaces `stdout`/`stderr` with interceptors, allowing the Electron host to stream structured logs (for readiness detection or UI consoles) as soon as the process starts emitting output.【F:app/logger.py†L13-L84】

### Reference Electron Launcher Skeleton
```ts
import { app, BrowserWindow } from 'electron';
import { spawn } from 'node:child_process';
import path from 'node:path';

let backend;
app.on('ready', () => {
  const python = path.join(process.resourcesPath, 'python', 'python.exe');
  backend = spawn(python, ['main.py', '--windows-standalone-build', '--auto-launch'], {
    cwd: path.join(process.resourcesPath, 'ComfyUI'),
    env: { ...process.env }
  });

  backend.stdout.on('data', (buf) => {
    const text = buf.toString();
    if (text.includes('To see the GUI go to:')) {
      BrowserWindow.getAllWindows()[0]?.loadURL('http://127.0.0.1:8188');
    }
  });
});
```
The watcher reacts to the readiness log generated later by the prompt server startup sequence.【F:server.py†L960-L986】

## 2. CLI Parsing and Logging Guarantees
- `comfy.options.enable_args_parsing()` primes the global argument parser before any other module imports, ensuring flags from the Electron launcher are honored consistently.【F:main.py†L1-L24】
- `setup_logger` applies verbosity settings immediately, so warnings about missing dependencies or old interpreters surface before the UI attempts to load static assets.【F:main.py†L1-L24】【F:app/logger.py†L54-L97】

## 3. Path and Dependency Validation
- `apply_custom_paths()` loads `extra_model_paths.yaml` from the installation root plus any overrides passed on the command line, then seeds canonical subdirectories (checkpoints, VAE, LoRA, diffusion models) beneath the chosen output folder. This ensures model discovery works even before users copy weights into place.【F:main.py†L25-L57】
- Input and user directories are remapped when flags are provided, letting installers locate shared storage or user profiles on first launch.【F:main.py†L35-L57】

## 4. Custom Node Pre-Startup Execution
- The runtime enumerates every folder registered under `custom_nodes`, running `prestartup_script.py` for each module unless the user disables or whitelists nodes. Errors are logged but do not crash startup, giving bundles a chance to report broken extensions before the UI loads them.【F:main.py†L60-L105】

## 5. Device and Environment Configuration
- Accelerator-related switches (`--default-device`, `--cuda-device`, `--oneapi-device-selector`, `--deterministic`) translate into environment variables before PyTorch is imported, guaranteeing the packaged backend honors GPU selection made in the Electron UI.【F:main.py†L117-L140】
- Windows builds additionally set `MIMALLOC_PURGE_DELAY=0` to keep the bundled mimalloc allocator responsive for repeated launches.【F:main.py†L113-L115】

## 6. Import Ordering and Safety Checks
- The launcher warns if `torch` was imported prematurely, protecting environment tweaks performed above. It then imports the execution engine, prompt server, protocol constants, node registry, and version metadata required for the desktop app.【F:main.py†L142-L154】
- `cuda_malloc_warning()` inspects the active GPU name against a blacklist and emits guidance about disabling async allocation when necessary, preventing UI freezes caused by unsupported hardware.【F:main.py†L156-L166】

## 7. Prompt Execution Thread and Progress Hooks
- `prompt_worker` starts in a daemon thread to process queued workflows without blocking the event loop. It selects cache policies based on CLI flags, reports execution time, manages garbage collection, and relays success or error states back to the server queue.【F:main.py†L168-L235】
- `hijack_progress()` installs a global hook so node-level progress and preview images stream to connected clients via websocket events, satisfying the desktop UI’s live feedback requirements.【F:main.py†L237-L275】

## 8. Temporary Workspace and Database Preparation
- `cleanup_temp()` clears the temp directory each launch, while `setup_database()` initialises optional SQL features when dependencies from the bundled environment are available. Failures become startup warnings instead of hard crashes, allowing the UI to appear with actionable guidance.【F:main.py†L277-L289】

## 9. Prompt Server Construction
- `start_comfyui()` optionally applies a custom temp directory, runs the Windows updater hook, creates the asyncio event loop, and instantiates `PromptServer` with that loop.【F:main.py†L292-L314】
- The server constructor wires middleware, enforces upload limits, initialises user/model/custom-node managers, and picks the frontend root by calling `FrontendManager.init_frontend`, which validates the installed `comfyui-frontend-package` wheel or downloads an alternate build requested via CLI.【F:server.py†L162-L205】【F:app/frontend_management.py†L200-L360】
- `PromptServer.setup()` creates a long-lived `aiohttp` client session used by certain routes before any connections arrive.【F:server.py†L781-L789】

## 10. Route Registration and Static Asset Exposure
- `PromptServer.add_routes()` registers REST endpoints for embeddings, models, file uploads, workflow templates, extension assets, and embedded docs before finally serving `index.html` from the resolved web root. This guarantees that by the time the Electron window navigates to the backend URL, all required assets and APIs are available.【F:server.py†L248-L360】
- Custom node web extensions and optional documentation packages are automatically mounted if present in the bundled site-packages, matching the dependencies declared in `requirements.txt`.【F:server.py†L228-L314】【F:requirements.txt†L1-L23】

## 11. Launch Signal and UI Display
- After route registration, `start_comfyui()` launches the prompt worker thread, ensures the temp directory exists, and registers an optional `call_on_start` callback when `--auto-launch` is active. Desktop bundles can reuse this callback to open a browser or notify the Electron renderer when the service becomes reachable.【F:main.py†L328-L347】
- `run()` concurrently awaits `PromptServer.start_multi_address()` and the websocket publish loop. Once listening sockets bind successfully, the server logs `To see the GUI go to: …` and executes `call_on_start`, providing the scheme, address, and port that the Electron shell should load.【F:main.py†L237-L350】【F:server.py†L948-L986】
- The top-level `__main__` block logs interpreter and ComfyUI versions, prints any queued startup warnings (such as outdated frontend packages), and finally blocks the event loop until shutdown, keeping the UI responsive while background workers handle prompts.【F:main.py†L353-L369】【F:app/logger.py†L90-L97】

## 12. Recommended Renderer Readiness Flow
Desktop builds can coordinate their renderer process with the backend readiness signal using the following pattern:

```ts
backend.stdout.on('data', (buf) => {
  const text = buf.toString();
  if (text.includes('To see the GUI go to:')) {
    const url = text.split('To see the GUI go to:')[1].trim();
    mainWindow.webContents.send('backend-ready', url);
    if (!mainWindow.isVisible()) mainWindow.show();
  }
});
```

The renderer listens for the `backend-ready` IPC event, displays a loading indicator until it fires, and only then embeds the ComfyUI frontend, guaranteeing all dependencies, routes, and websockets negotiated during startup are in place.【F:server.py†L962-L986】【F:main.py†L333-L365】
