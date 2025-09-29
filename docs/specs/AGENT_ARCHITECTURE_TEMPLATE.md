# Agent-Driven Architecture Specification Template

## Purpose

This document provides a **reusable architectural paradigm** extracted from ComfyUI's specification suite, designed for coding agents (like Claude) to create new projects or refactor existing ones following battle-tested patterns for **distributed, plugin-based, multi-platform runtime systems**.

Use this template when building applications that need:
- Modular plugin/extension systems
- Multi-platform deployment (desktop, portable, cloud)
- Asset/resource management with remote capabilities
- Long-running background processing with progress reporting
- WebSocket-based real-time UI updates
- Flexible runtime configuration and dependency injection

---

## Core Architectural Principles

### 1. **Separation of Runtime Concerns**

Projects should cleanly separate these layers:

```
┌─────────────────────────────────────────┐
│  Entry Point (main.py)                  │
│  - CLI parsing                          │
│  - Environment setup                    │
│  - Path configuration                   │
│  - Plugin initialization                │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Server Layer (server.py)               │
│  - HTTP/WebSocket server                │
│  - Middleware (CORS, compression, auth) │
│  - Route handlers                       │
│  - Session management                   │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Execution Engine (execution.py)        │
│  - Task queue management                │
│  - Dependency graph resolution          │
│  - Caching and memoization             │
│  - Progress tracking                    │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│  Core Logic (nodes.py, modules/)        │
│  - Business logic implementations       │
│  - Plugin/node definitions              │
│  - Data transformations                 │
└─────────────────────────────────────────┘
```

**Why**: Each layer has a single responsibility, making testing, distribution, and extension easier.

---

## Specification Structure

Every project following this paradigm should document these aspects:

### 📋 1. Runtime Initialization Specification

**Purpose**: Define the startup sequence so distributors can reproduce exact runtime behavior.

**Required Sections**:
- **CLI Argument Parsing**: Document all flags, their defaults, and precedence
- **Environment Configuration**: List environment variables that affect behavior
- **Path Resolution**: Explain how the system locates assets, plugins, and user data
- **Plugin Bootstrap**: Detail pre-startup hooks and initialization order
- **Device/Hardware Selection**: Document accelerator detection and configuration
- **Module Import Order**: Specify critical import sequences and side effects
- **Worker Thread Setup**: Explain background execution threads/processes
- **Lifecycle Hooks**: Document cleanup, shutdown, and restart behaviors

**Template**:
```markdown
# Runtime Initialization Specification

## Scope
[Describe what this document covers and who the audience is]

## Argument Parsing
- Primary parser location: `[path/to/cli_args.py]`
- Key flags:
  - `--flag-name`: [purpose, default, type]

## Environment Setup
- Required variables: `[VAR_NAME=default]`
- Optional tuning: `[PERFORMANCE_FLAG=value]`

## Path Resolution
- Base directory: `[resolution logic]`
- Asset directories: `[mapping from type to paths]`
- Override mechanism: `[CLI flags or config files]`

## Plugin Initialization
- Discovery path: `[where plugins live]`
- Load order: `[alphabetical, priority-based, etc.]`
- Pre-startup hooks: `[script names, execution timing]`

## Reference Bootstrap Code
[Provide minimal Python/JS/etc. snippet showing initialization flow]
```

---

### 🌐 2. Server & API Specification

**Purpose**: Define how clients interact with the runtime and how state is synchronized.

**Required Sections**:
- **Core Initialization**: Server instance creation, dependency injection
- **Middleware Stack**: CORS, compression, caching, authentication
- **WebSocket Lifecycle**: Connection, reconnection, feature negotiation
- **HTTP API Surface**: All REST endpoints with request/response schemas
- **Queue Integration**: How work is submitted and status is reported
- **Frontend Delivery**: Static asset serving, version management
- **TLS/Security**: Certificate handling, secure defaults

**Template**:
```markdown
# Server & API Specification

## Core Initialization
- Server class: `[ClassName at path]`
- Dependencies: `[ManagerA, ManagerB, Queue]`
- Configuration sources: `[CLI flags, config files]`

## Middleware Stack
Order matters! List from outermost to innermost:
1. `[middleware_name]`: [purpose]
2. `[cors_handler]`: [when enabled, behavior]

## WebSocket Protocol
- Route: `[/ws, /socket, etc.]`
- Session management: `[how clients are identified]`
- Initial handshake: `[feature flags, auth]`
- Message format: `[JSON schema or binary protocol]`
- Reconnection policy: `[client or server-driven]`

## HTTP Endpoints
### `GET /endpoint`
- Purpose: [what it does]
- Query params: `[param: type, description]`
- Response: `[schema]`
- Error codes: `[404 if X, 400 if Y]`

[Repeat for all endpoints]

## Queue Integration
- Submission: `[POST /queue or websocket command]`
- Status updates: `[polling vs. push]`
- Cancellation: `[mechanism]`

## Reference Service Blueprint
[Minimal async server skeleton in target language]
```

---

### 🐍 3. Environment & Dependency Specification

**Purpose**: Lock down runtime versions and document platform-specific setup.

**Required Sections**:
- **Supported Runtimes**: Language versions (Python 3.12+, Node 18+, etc.)
- **Dependency Manifest**: Pinned versions, justification for pins
- **Accelerator-Specific Setup**: CUDA, ROCm, Metal, DirectML instructions
- **Optional Dependencies**: Feature flags that require extra packages
- **Frontend Package Enforcement**: How to verify UI/backend compatibility

**Template**:
```markdown
# Environment & Dependency Specification

## Supported Runtimes
- Primary: `[Language X.Y.Z]`
- Minimum: `[Language X.Y]`
- Tested: `[versions actually CI-tested]`

## Dependency Manifest
Location: `[requirements.txt, package.json, Cargo.toml]`

Critical pins:
- `[package==version]`: [why pinned]
- `[package>=version]`: [why minimum]

## Platform-Specific Setup
### Linux + NVIDIA
```bash
[installation commands]
```

### Windows + AMD
```bash
[alternative installation]
```

### macOS + Apple Silicon
```bash
[Metal-specific setup]
```

## Verification Script
[Code snippet to check environment before launch]
```

---

### 📦 4. Asset & Resource Management Specification

**Purpose**: Define how the system discovers, loads, and serves files or remote resources.

**Required Sections**:
- **Default Directory Map**: Where each asset type lives by default
- **Dynamic Overrides**: CLI flags or config files to change paths
- **Extra Path Configuration**: YAML/JSON schemas for multi-source setups
- **File Enumeration & Caching**: How to efficiently list large directories
- **Annotated Paths**: Conventions for embedding paths in metadata
- **Temp Directory Hygiene**: Cleanup policies for transient files
- **Remote Resource Integration**: How to blend local and API-based assets

**Template**:
```markdown
# Asset & Resource Management Specification

## Default Directory Map
Relative to base directory (override with `--base-dir`):
- `[type]`: `[default/path]` (supports: `[.ext1, .ext2]`)

## Dynamic Overrides
- CLI: `--asset-dir-[type]=/custom/path`
- Env: `ASSET_PATH_[TYPE]=/custom/path`
- Config: `asset_paths.yaml` (loaded from base or `--config`)

## Extra Paths Configuration
Schema for `asset_paths.yaml`:
```yaml
[type]:
  - /path/one
  - /path/two
remote:
  [type]: https://api.example.com/v1/[type]
```

## Enumeration Strategy
- Cache key: `[how cache is keyed]`
- Invalidation: `[mtime-based, on-demand, etc.]`
- Pagination: `[for APIs returning 1000+ items]`

## Reference Asset Registry
[Code showing how to register local + remote sources]
```

---

### 🎨 5. Frontend Delivery Specification

**Purpose**: Synchronize UI builds with backend releases.

**Required Sections**:
- **Packaging Strategy**: How frontend is versioned and distributed (npm, wheel, CDN)
- **Version Enforcement**: Startup checks to warn about mismatches
- **Alternate Sources**: Mechanism to load different UI builds
- **Server Integration**: How backend serves static assets
- **Feature Flag Negotiation**: Protocol for capability detection

**Template**:
```markdown
# Frontend Delivery Specification

## Packaging
- Format: `[npm package, Python wheel, Docker layer]`
- Registry: `[where published]`
- Version file: `[how backend reads version]`

## Version Enforcement
On startup, check:
```python
required = parse_requirements()
installed = get_installed_version("[package]")
if installed != required:
    warn(f"UI mismatch: {installed} != {required}")
```

## Alternate UI Sources
- Flag: `--frontend-version [source@tag]`
- Download location: `[temp or persistent cache]`
- Precedence: `[custom > installed > bundled]`

## Serving Strategy
- Static route: `[/ or /ui]`
- Cache headers: `[no-cache for index.html, long-lived for assets]`
- SPA fallback: `[how to handle client-side routes]`

## Feature Negotiation
WebSocket handshake includes:
```json
{
  "type": "feature_flags",
  "data": {
    "supports_streaming": true,
    "supports_binary_preview": false
  }
}
```
```

---

### 🚀 6. Distribution Channel Specification

**Purpose**: Coordinate releases across repositories and artifacts.

**Required Sections**:
- **Release Cadence**: Weekly, monthly, on-demand
- **Artifact Types**: Desktop installers, portable archives, Docker images
- **Versioning Scheme**: SemVer, CalVer, commit-based
- **Cross-Repo Coordination**: How core, frontend, and desktop repos sync
- **Automation Hooks**: CI/CD triggers and artifact promotion

**Template**:
```markdown
# Distribution Channel Specification

## Release Cadence
- Target: `[weekly Friday, monthly first Tuesday]`
- Flexibility: `[may shift for breaking changes]`

## Artifact Matrix
| Artifact | Platform | Format | Auto-Build |
|----------|----------|--------|------------|
| Desktop Installer | Windows | `.exe` | ✅ |
| Portable Archive | Windows | `.7z` | ✅ |
| AppImage | Linux | `.AppImage` | ✅ |
| Docker Image | All | `org/repo:tag` | ✅ |

## Versioning
- Core: `v[MAJOR].[MINOR].[PATCH]`
- Frontend: `v[MAJOR].[MINOR].[PATCH]`
- Bundled: Core version is canonical

## Cross-Repo Flow
1. Frontend merges weekly updates
2. Core tags release (e.g., `v0.8.0`)
3. Desktop repo watches core tags, triggers installer builds
4. Docker workflow pulls core tag, bakes image
5. Portable archive built from same tag

## Reference CI Workflow
```yaml
on:
  push:
    tags: ['v*']
jobs:
  test-core:
    # ...
  build-installers:
    needs: test-core
    uses: org/desktop/.github/workflows/build.yml@main
```
```

---

### 🖥️ 7. Hardware Support Specification

**Purpose**: Document setup for different accelerators.

**Required Sections**:
- **Vendor-Specific Instructions**: NVIDIA, AMD, Intel, Apple, others
- **Environment Variables**: Tuning knobs for each platform
- **Fallback Strategies**: CPU-only, remote API
- **Detection Logic**: How runtime chooses accelerator

**Template**:
```markdown
# Hardware Support Specification

## NVIDIA GPUs
Install: `pip install torch --extra-index-url [CUDA URL]`
Env vars: `CUDA_VISIBLE_DEVICES=0`

## AMD GPUs (Linux)
Install: `pip install torch --index-url [ROCm URL]`
Tuning: `HSA_OVERRIDE_GFX_VERSION=10.3.0` for older cards

[Repeat for each platform]

## Runtime Detection
```python
def detect_accelerator():
    if shutil.which("nvidia-smi"): return "cuda"
    if shutil.which("rocminfo"): return "rocm"
    # ...
    return "cpu"
```

## Fallback Strategy
If no accelerator: `[--cpu flag, or redirect to API endpoint]`
```

---

### 👤 8. User Responsibilities Specification

**Purpose**: Define the minimal steps end users must complete.

**Required Sections**:
- **Asset Provisioning**: What users must supply (models, credentials, etc.)
- **Optional Customization**: Config files users can tweak
- **Launching**: How to start the application
- **Updates**: How to stay current

**Template**:
```markdown
# User Responsibilities Specification

## Asset Provisioning
After installation, users must:
1. Copy `[asset type]` into `[directory]`
2. (Optional) Rename `[config.example]` to `[config]` and edit

## Launching
- Desktop: Click icon or run `[app.exe]`
- Manual: `[language] [entrypoint] [optional flags]`

## First-Run Checklist
```python
def verify_setup():
    if not Path("[required/asset]").exists():
        print("❌ Missing required asset")
    # ...
```

## Keeping Updated
- Desktop: `[built-in updater, or download new installer]`
- Manual: `git pull && pip install -r requirements.txt`
```

---

### 📦 9. Packaging Specification (per platform)

**Purpose**: Document how to create self-contained distributions.

**Required Sections** (Windows Portable Example):
- **Distribution Format**: Archive type, compression
- **Bundled Contents**: Runtime, dependencies, directory structure
- **Launch Scripts**: Batch files, shell scripts
- **Update Mechanism**: How users refresh without re-downloading

**Template**:
```markdown
# [Platform] Portable Package Specification

## Format
- Archive: `[.7z, .tar.gz, .dmg]`
- Naming: `[app]-[platform]-[arch]-v[version].[ext]`

## Contents
```
/
├── [runtime]/        # Python, Node, etc.
├── [app]/            # Application code
├── [app]/models/     # Asset directories
├── run.[bat|sh]      # Launch script
└── README.txt        # User instructions
```

## Launch Script
```bash
#!/bin/bash
cd "$(dirname "$0")"
./[runtime]/bin/[exe] [app]/[entrypoint] "$@"
```

## Update Script
```bash
#!/bin/bash
# Check for updates, download diff, apply patch
```

## Reference Packing Script
[PowerShell/Bash to assemble archive]
```

---

## Agent Implementation Checklist

When using this template to build or refactor a project, ensure:

### Phase 1: Architecture Definition
- [ ] Define core layers (entry, server, execution, logic)
- [ ] Choose communication protocol (REST, WebSocket, gRPC)
- [ ] Design plugin/extension system interface
- [ ] Plan asset/resource management strategy

### Phase 2: Specification Writing
- [ ] Write Runtime Initialization spec
- [ ] Write Server & API spec
- [ ] Write Environment & Dependency spec
- [ ] Write Asset Management spec
- [ ] Write Frontend Delivery spec (if applicable)
- [ ] Write Distribution Channel spec
- [ ] Write Hardware Support spec (if applicable)
- [ ] Write User Responsibilities spec
- [ ] Write per-platform Packaging specs

### Phase 3: Reference Implementation
- [ ] Create minimal bootstrap code for each spec
- [ ] Implement core functionality following specs
- [ ] Add plugin/extension loading mechanism
- [ ] Build execution queue and progress tracking
- [ ] Implement WebSocket real-time updates
- [ ] Create packaging scripts for target platforms

### Phase 4: Testing & Documentation
- [ ] Write integration tests for each layer
- [ ] Test on all target platforms
- [ ] Generate API documentation from specs
- [ ] Create user onboarding guide
- [ ] Document plugin development process

---

## Key Patterns to Replicate

### 1. **Explicit Path Resolution**
Don't rely on implicit working directories. Always:
- Accept `--base-directory` or equivalent
- Compute all paths relative to base
- Allow per-asset-type overrides
- Support external config files for shared resources

### 2. **Plugin Pre-Startup Hooks**
Before heavy imports:
- Scan plugin directories
- Execute `prestartup_script.[ext]` or equivalent
- Log timing for diagnostics
- Allow whitelist/blacklist via CLI

### 3. **Lazy Module Loading**
Import expensive dependencies only after:
- CLI parsing completes
- Environment variables are set
- Device selection is finalized

### 4. **Multi-Address Server Binding**
Support listening on multiple interfaces:
```python
addresses = [(addr, port) for addr in args.listen.split(",")]
await server.start_multi_address(addresses)
```

### 5. **Feature Flag Negotiation**
First WebSocket message from client:
```json
{"type": "feature_flags", "data": {"supports_X": true}}
```
Server responds with its capabilities, both sides adjust behavior.

### 6. **Progress Hook Integration**
Global callback for long-running operations:
```python
def progress_hook(current, total, preview, context):
    server.send_sync("progress", {...}, client_id)
```

### 7. **Temp Directory Hygiene**
On startup:
```python
cleanup_temp()  # Remove stale files
os.makedirs(temp_dir, exist_ok=True)
```
On shutdown (even KeyboardInterrupt):
```python
finally:
    cleanup_temp()
```

### 8. **Middleware Ordering**
```python
middlewares = [
    cache_control,       # Outermost - affects all responses
    compress_body,       # Compress JSON/text if client supports
    cors_or_origin_check # Security - validate request origins
]
```

### 9. **Reference Code Blocks**
Every spec includes a minimal, runnable code snippet showing the pattern in isolation.

---

## Example: Applying Template to New Project

**Scenario**: Building "ImageFlow" - a node-based image processing pipeline

### Step 1: Architecture
```
main.py          → Parse args, configure paths, load plugins
server.py        → FastAPI/aiohttp server with WebSocket
execution.py     → Topological sort of processing graph
nodes/           → Built-in filters, transforms, I/O nodes
plugins/         → User-installed nodes
models/filters/  → Pre-trained ML filters
models/styles/   → Style transfer models
```

### Step 2: Runtime Init Spec
```markdown
## CLI Arguments
- `--listen`: Bind address (default: 127.0.0.1)
- `--port`: Port (default: 8080)
- `--model-dir`: Override default model location
- `--plugin-dir`: Additional plugin search paths

## Path Resolution
Base: `--base-directory` or script location
Models: `{base}/models/{type}` + extra_paths.yaml overrides
Plugins: `{base}/plugins` + `--plugin-dir`
```

### Step 3: Asset Management Spec
```markdown
## Directory Map
- `models/filters/`: `.onnx, .pt` files
- `models/styles/`: `.safetensors` files
- `models/upscalers/`: `.pth` files

## Remote Fallback
extra_paths.yaml:
```yaml
remote:
  upscalers: https://cdn.example.com/models/upscalers/
```
When local model not found, fetch from CDN, cache to `{base}/cache/`
```

### Step 4: Implementation
Follow checklist, implement each layer, write specs as you go.

---

## Anti-Patterns to Avoid

❌ **Implicit Working Directory Assumptions**
```python
# Bad
models = os.listdir("models")

# Good
models = os.listdir(os.path.join(base_dir, "models"))
```

❌ **Hardcoded Paths**
```python
# Bad
CONFIG_PATH = "/usr/local/share/app/config.yaml"

# Good
CONFIG_PATH = os.path.join(
    args.config_dir or
    os.environ.get("APP_CONFIG_DIR") or
    os.path.join(base_dir, "config"),
    "config.yaml"
)
```

❌ **Synchronous Long-Running Operations in Server Thread**
```python
# Bad
@app.post("/process")
async def process(request):
    result = run_heavy_computation()  # Blocks event loop!
    return result

# Good
@app.post("/process")
async def process(request):
    job_id = await queue.submit(request.data)
    return {"job_id": job_id, "status": "queued"}
```

❌ **Silent Plugin Failures**
```python
# Bad
for plugin in plugins:
    try:
        load_plugin(plugin)
    except: pass

# Good
for plugin in plugins:
    try:
        load_plugin(plugin)
    except Exception as e:
        logger.error(f"Failed to load {plugin}: {e}")
```

---

## Specification Maintenance

### When to Update Specs

1. **Before Major Refactors**: Update specs first, then code to match
2. **After Feature Additions**: Document new CLI flags, endpoints, asset types
3. **On Breaking Changes**: Clearly mark deprecated behaviors, migration paths
4. **Per Release**: Review all specs, ensure examples still run

### Version Control

Specs should:
- Live in `docs/specs/` at repository root
- Use semantic filenames: `[domain]_[aspect]_spec.md`
- Reference source code with: `【F:path/file.ext†L10-L20】` notation
- Include "last updated" and "spec version" headers

### Tooling

Consider:
- **Spec Linter**: Check that all reference code blocks are valid syntax
- **Link Checker**: Verify file references point to existing code
- **API Contract Tests**: Generate test stubs from endpoint specs
- **Doc Generator**: Turn specs into user-facing guides

---

## Summary

This template extracts ComfyUI's architectural patterns into a **reusable blueprint** for building:
- ✅ Plugin-extensible systems
- ✅ Multi-platform runtimes (desktop, portable, cloud)
- ✅ Real-time UIs with progress tracking
- ✅ Hybrid local/remote resource systems
- ✅ Distribution-ready applications

**For AI Coding Agents**: Use this as a specification framework when the user asks to "build a system like X" or "refactor Y to be more modular." Each section maps to code you should generate, tests to write, and docs to produce.

**For Developers**: Use this when designing your next big project. Write specs first, code second, ship third. The specs become living documentation that makes onboarding, debugging, and extending your system dramatically easier.

---

## Credits

Derived from the ComfyUI specification suite (docs/specs/), which documents a production-grade system serving thousands of users across Windows, Linux, macOS, and cloud environments.

## License

This template is provided as-is. Adapt freely for your projects. Attribution appreciated but not required.

---

**Ready to build? Start with Runtime Initialization, implement the bootstrap, then layer in Server, Execution, and Assets. Specs guide implementation; implementation validates specs. Iterate until both are crystal clear.**