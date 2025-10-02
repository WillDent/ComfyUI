# Prompt Server Specification

## Scope
Defines the responsibilities of `server.py` and the `PromptServer` class that wraps aiohttp for all ComfyUI distributions. It explains middleware, websocket lifecycle, REST endpoints, queue coordination, and frontend delivery expectations that installers must satisfy.

## Core Initialization
- `PromptServer` wires together user, model, and custom node managers, exposes internal API routes, and instantiates the shared `PromptQueue` that orchestrates workflow execution.【F:server.py†L143-L157】
- The aiohttp application enforces maximum upload sizes using `--max-upload-size` and attaches cache-control plus optional compression and CORS middleware so packaged frontends experience consistent HTTP behavior.【F:server.py†L162-L176】【F:server.py†L171-L172】
- Frontend assets resolve through `FrontendManager.init_frontend`, allowing bundles to pin specific frontend wheels or serve custom roots via `--front-end-root`. The resolved path is logged to simplify debugging custom distributions.【F:server.py†L175-L181】

## Middleware Strategy
- `cache_control` headers prevent browsers from caching frequently changing JSON responses, while `compress_body` negotiates gzip for JSON/text payloads when clients support it. This keeps websocket and REST payloads lean on slower networks.【F:server.py†L42-L61】
- When no explicit CORS origin is configured, `create_origin_only_middleware` rejects requests where the Host and Origin differ on loopback interfaces, guarding local installs against CSRF-style queue injections from malicious web pages.【F:server.py†L107-L140】

## Websocket Lifecycle
- The `/ws` route provisions a websocket per client, reuses session identifiers across reconnects, and immediately sends queue status plus the actively executing node when applicable. Feature-flag negotiation occurs on the first message to tailor server responses (e.g., preview metadata) to each client.【F:server.py†L188-L236】
- Socket references and negotiated capabilities are stored separately so prompt progress hooks in `main.py` can address the correct connection when broadcasting updates.【F:server.py†L188-L205】【F:server.py†L226-L236】

## HTTP API Surface
- `GET /` streams the packaged `index.html` with no-cache headers, ensuring desktop builds always load the bundled frontend version.【F:server.py†L238-L245】
- Model discovery endpoints (`/models`, `/models/{folder}`, `/embeddings`) read from `folder_paths` so any path overrides applied during startup automatically flow to the UI. A dedicated `/extensions` route returns discovered frontend extensions from both the shipped web root and registered custom node directories.【F:server.py†L247-L286】
- Upload handlers normalize writes into the appropriate `input`, `output`, or `temp` directories while preventing directory traversal. Optional hash comparison avoids duplicating identical uploads, mirroring the expectations of portable deployments where disk space is limited.【F:server.py†L288-L371】

## Prompt Queue Integration
- `PromptServer` exposes `prompt_queue`, which the worker thread created in `main.py` consumes. REST and websocket handlers enqueue prompts, while queue state is reflected back through `send`/`send_sync` events so the UI can display live status.【F:server.py†L143-L157】【F:main.py†L168-L234】

## Publishing and Multi-Address Support
- `run()` coordinates simultaneous binding to multiple comma-separated listen addresses and starts the publish loop that drains queued websocket notifications, supporting installers that expose ComfyUI on IPv4 and IPv6 simultaneously.【F:main.py†L237-L243】

## Frontend Package Overrides
- `FrontendManager` can fetch alternate frontend builds from GitHub releases when operators specify `--front-end-version`. Bundled runtimes may preseed custom versions, but they must still keep the CLI hooks accessible for advanced users.【F:server.py†L175-L181】【F:app/frontend_management.py†L1-L120】

## TLS and Client Session Reuse
- The server maintains an aiohttp client session for outbound requests (e.g., fetching remote custom nodes) and respects TLS settings provided at launch, allowing desktop builds to bundle certificates or reuse system ones without modification.【F:server.py†L143-L160】

## Reference Service Blueprint
The snippet below turns these behaviours into a portable template suitable for projects that orchestrate local and remote inference endpoints.

```python
from __future__ import annotations

import asyncio
from dataclasses import dataclass
from typing import Any, AsyncIterator, Callable

from aiohttp import web


@dataclass
class PromptJob:
    id: str
    graph: dict[str, Any]
    assets: dict[str, Any]


class PromptGateway:
    def __init__(self, execute: Callable[[PromptJob], AsyncIterator[dict[str, Any]]]):
        self._execute = execute
        self._queue: asyncio.Queue[PromptJob] = asyncio.Queue()
        self._subscribers: set[Callable[[str, dict[str, Any]], Any]] = set()

    async def submit(self, job: PromptJob) -> str:
        await self._queue.put(job)
        return job.id

    def subscribe(self, callback: Callable[[str, dict[str, Any]], Any]) -> None:
        self._subscribers.add(callback)

    async def worker_loop(self) -> None:
        while True:
            job = await self._queue.get()
            async for update in self._execute(job):
                await self.broadcast(job.id, update)

    async def broadcast(self, job_id: str, payload: dict[str, Any]) -> None:
        for callback in list(self._subscribers):
            callback(job_id, payload)


async def create_app(gateway: PromptGateway) -> web.Application:
    app = web.Application()
    app["gateway"] = gateway

    async def submit_handler(request: web.Request) -> web.Response:
        body = await request.json()
        job = PromptJob(**body)
        job_id = await gateway.submit(job)
        return web.json_response({"id": job_id})

    app.router.add_post("/api/prompts", submit_handler)
    # Add websocket handler mirroring ComfyUI's `prompt_socket_handler`.
    return app


def start_prompt_server(execute_fn: Callable[[PromptJob], AsyncIterator[dict[str, Any]]]) -> None:
    gateway = PromptGateway(execute_fn)
    loop = asyncio.get_event_loop()
    loop.create_task(gateway.worker_loop())
    web.run_app(loop.run_until_complete(create_app(gateway)))
```

Design cues:

- **Async iterators** let `execute_fn` stream preview updates from either local GPU inference or hosted APIs, matching ComfyUI’s websocket cadence.【F:server.py†L188-L236】
- The pub/sub registry in `PromptGateway.broadcast` echoes how ComfyUI fans notifications to every websocket session, which is vital when multiple clients collaborate on the same queue.【F:server.py†L126-L173】
- Keeping the HTTP endpoint names aligned with ComfyUI simplifies frontend reuse: an installer can swap in a different backend while leaving the shipped web client untouched.
