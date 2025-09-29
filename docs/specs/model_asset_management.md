# Model and Asset Management Specification

## Scope
Clarifies how ComfyUI discovers, categorizes, and persists model assets across bundled and manual installations. This specification supports packaging teams that must pre-create directory structures or customize search paths.

## Default Directory Map
- `folder_paths` anchors all model directories relative to the installation base (overridden by `--base-directory`), automatically wiring `models/checkpoints`, `models/vae`, `models/loras`, `models/diffusion_models`, ControlNet/T2I adapters, embeddings, hypernetworks, upscale models, and more.【F:folder_paths.py†L1-L44】【F:folder_paths.py†L49-L66】
- Custom node discovery defaults to `<install>/custom_nodes`, while runtime directories such as `output`, `temp`, `input`, and `user` live beside the codebase. Installers must ship these directories or create them at first launch to avoid permission issues.【F:folder_paths.py†L33-L55】【F:folder_paths.py†L68-L83】

## Dynamic Directory Overrides
- `folder_paths` exposes `set_output_directory`, `set_input_directory`, `set_temp_directory`, and `set_user_directory` so CLI flags can point to alternate storage locations (e.g., external drives in portable builds). These setters update the global registry consumed by upload handlers and save nodes.【F:folder_paths.py†L86-L119】
- `main.py` automatically adds save destinations for checkpoints, CLIP, VAE, diffusion models, and LoRAs under the active output directory, ensuring generated assets never pollute the read-only install area shipped inside installers.【F:main.py†L35-L47】

## Extra Model Paths Configuration
- Optional `extra_model_paths.yaml` files merge into the same directory map at startup. The sample configuration demonstrates per-UI base paths, multi-line folder lists (for diffusion models and LoRAs), and custom node directories, enabling shared model pools across different products.【F:main.py†L25-L34】【F:extra_model_paths.yaml.example†L1-L47】

## File Enumeration and Caching
- `folder_paths` maintains a `filename_list_cache` and exposes helpers like `filter_files_content_types` to efficiently serve lists of models, images, videos, or audio through the HTTP API, minimizing redundant filesystem scans in large deployments.【F:folder_paths.py†L57-L83】【F:folder_paths.py†L121-L170】

## Annotated Filepaths
- `annotated_filepath` and `get_annotated_filepath` interpret suffixes like `[output]` or `[input]` to resolve resource references back to the correct directory, which is essential when workflows embed relative paths inside generated images or metadata.【F:folder_paths.py†L172-L214】

## Temp Directory Hygiene
- Startup purges the active temp directory and recreates it before the server runs, preventing stale intermediates from leaking between sessions in portable bundles that may live on removable drives.【F:main.py†L277-L334】
