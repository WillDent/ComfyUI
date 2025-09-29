# ComfyUI Desktop Distribution Specification

This document indexes the detailed specifications that describe every aspect of ComfyUI’s packaged releases. Use it as a roadmap when building or auditing desktop installers, the Windows portable bundle, or manual distribution guides.

## Release Coordination
- [Distribution Channel Specification](docs/specs/distribution_channels_spec.md) — explains how the core, desktop, and frontend repositories coordinate on a weekly cadence and how portable archives stay aligned.【F:README.md†L112-L180】【F:requirements.txt†L1-L2】

## Packaged Runtime Expectations
- [Desktop Installer Specification](docs/specs/desktop_installers_spec.md) — outlines platform targets, bundled Python requirements, directory layout, launcher behavior, and user onboarding steps for native installers.【F:README.md†L41-L199】【F:requirements.txt†L1-L30】【F:folder_paths.py†L16-L83】
- [Windows Portable Package Specification](docs/specs/windows_portable_spec.md) — defines archive contents, update mechanisms, and shared model support for the self-contained Windows release.【F:README.md†L168-L180】【F:main.py†L297-L308】

## Manual Path Reference
- [Manual Installation Specification](docs/specs/manual_install_spec.md) — mirrors the instructions for setting up ComfyUI from source so documentation teams can produce consistent guides.【F:README.md†L191-L315】【F:requirements.txt†L1-L30】

## Runtime Behavior and Frontend Delivery
- [Runtime Initialization Specification](docs/specs/runtime_initialization.md) — captures how launchers set environment variables, register model paths, execute custom node hooks, and start the prompt queue.【F:main.py†L1-L369】
- [Frontend Delivery Specification](docs/specs/frontend_delivery_spec.md) — documents how the frontend wheel is validated, how alternate builds are fetched, and how the server exposes static assets with feature flag negotiation.【F:app/frontend_management.py†L19-L214】【F:server.py†L175-L245】

## User Guidance
- [User Responsibilities Specification](docs/specs/user_responsibilities_spec.md) — lists the post-install tasks end users must perform, regardless of whether they use an installer, portable package, or manual setup.【F:README.md†L174-L200】【F:main.py†L25-L350】
- [Hardware Support Specification](docs/specs/hardware_support_spec.md) — aggregates accelerator-specific installation instructions to include in release notes or onboarding dialogs.【F:README.md†L200-L315】
