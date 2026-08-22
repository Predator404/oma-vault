# pi — upstream root

**pi** (`earendil-works/pi`, historically `badlogic/pi-mono`) is the upstream
project both Oh My Pi and Prime Agent fork. Authoritative for anything about the
*shared* substrate: the agent runtime, unified provider API, session/harness
model, TUI, and skill conventions that the forks inherit rather than invent.

- Website: https://pi.dev · Docs: https://pi.dev/docs/latest
- Repository: https://github.com/badlogic/pi-mono (now https://github.com/earendil-works/pi)
- License: **MIT**, © 2025 Mario Zechner (the root copyright present in *both*
  fork LICENSE files — the legal confirmation of the sibling-fork lineage).
- Packages:
  - `@earendil-works/pi-coding-agent` — interactive coding-agent CLI
  - `@earendil-works/pi-agent-core` — agent runtime (tool calling, state)
  - `@earendil-works/pi-ai` — unified multi-provider LLM API
  - `@earendil-works/pi-tui` — differential-rendering terminal UI

> Historical note: upstream published under `@mariozechner/*`, then
> `@earendil-works/*`; OMP rewrites both scopes to `@oh-my-pi/*` on port. See
> `../oh-my-pi/docs/porting-from-pi-mono.md` §3 for the exact scope map.

## What's here

Only pi-core material that is **not** already represented by the fork doc
mirrors (`../oh-my-pi/docs`, `../prime-agent/docs`), to avoid near-duplicates:

- `pi-upstream-README.md` — the monorepo README (package map, build, supply-chain
  hardening, containerization patterns).
- `agent-core-docs/harness.md` — **the** `AgentHarness` durable-runtime spec:
  three stores (entries / registers / usage ledger), lanes as named cursors, the
  operation state machine as a durable program counter, the effect-sandwich
  crash-recovery model. This is the deepest single explanation of *why* sessions
  survive crashes; OMP's and Prime's session docs are downstream views of it.
- `agent-core-docs/search.md` — agent-core search harness.
- `agent-core-docs/agent-core-README.md` — `@earendil-works/pi-agent-core` API.
- `agent-core-docs/containerization.md` — sandboxing patterns (Gondolin
  micro-VM, plain Docker, OpenShell); pi has **no built-in permission system**.

## Where the fork-specific truth lives

For how a *fork* actually behaves, prefer the fork's own mirror over pi:

- OMP behavior → `../oh-my-pi/docs/` (discrete tools, MCP exposure, marketplace,
  memory backends, natives).
- Prime Agent behavior → `../prime-agent/docs/` (ipython/RLM single-tool model,
  daemon/heartbeat/goal/schedule).

pi is the root; the forks diverge deliberately. When they disagree, the fork's
own docs + source win for that fork.

_Snapshot 2026-08-18 from `raw.githubusercontent.com/badlogic/pi-mono/main`.
`telemetry-schema.md` is generated upstream and absent from the source tree._
