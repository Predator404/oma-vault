# Oh-my-pi-agent — this project

The **persistent multi-entity agent system** built on OMP: the `oma` build.
Where the other three sections are *upstream/sibling harness* docs, this section
is the design record of **the thing Phi itself is part of** — the daemon/entity
subsystem ported from Prime Agent into OMP, the entity registry, the memory and
vault MCP servers, and the vault/project-pointer model.

> **Authoritative, self-contained copy.** The project's design docs, copied in
> full. This is the **home of record** — it does not point to, or depend on, the
> working folder they were authored in (`~/Work/persistentAgents/`), which may be
> removed without loss.

## Naming

- **pi** → upstream root · **OMP** (`omp`) → the base harness fork ·
  **Prime Agent** → the sibling fork the daemon comes from.
- **Oh-my-pi-agent** → *this* project = OMP + the ported persistent-agent
  feature. Ships as the **`oma`** launcher (`oma/0.1.0`, config root `~/.oma`,
  status marker `πA`), side by side with stock `omp`. Not a separate harness —
  an additive graft onto OMP that keeps OMP's discrete-tool surface and drops
  ipython/RLM.

## Files

| File | What it is | Originally from |
|------|-----------|-----------------|
| `SPEC.md` | The buildable spec: goal/non-goals, terminology, architecture, contracts list, runtime (§8), model hosting (§9), workstreams WS1–WS7 (+WS7a setup), integration stage, status. | `SPEC.md` |
| `CONTRACTS.md` | Frozen cross-workstream seams **C1/C4/C5** (grounded in codebase scouts): entity record schema, daemon control protocol, session-artifact paths + entry types. Plus the architecture decision (extend OMP's broker, don't fork a supervisor) and graft risks. | `CONTRACTS.md` |
| `HANDOFF.md` | The decision/rationale trail + **CURRENT STATE** block (what's built, where, git/build status, `oma` launcher, config-root isolation, UI rebrand, follow-ups). Read the top block first to resume. | `HANDOFF.md` |
| `registry-README.md` | Repo-2 registry: on-disk layout, **C1 record schema table**, role→retention policy (persona rejects `autoRetain: true`), the `resolveEntityConfig` seam, root resolution. | `registry/README.md` |
| `phi-entity-record.md` | Phi's own C1 record — the reference entry exercising every field. Phi's self-description. | `registry/entities/phi.md` |
| `mcp.json` | This project's `.omp/mcp.json` (MCP server wiring). | `.omp/mcp.json` |

## The five contracts (map)

- **C1** — entity registry record schema → `registry-README.md`, `CONTRACTS.md` §C1. Produced by WS2.
- **C2** — memory MCP: `recall/retain/forget(bank,…)` → `SPEC.md` §6. Produced by WS3.
- **C3** — vault MCP: `search_notes/get_note/write_note/get_connections` → `SPEC.md` §6. Produced by WS4. *(This is the tool surface that reads the very vault you are in.)*
- **C4** — daemon control protocol (spawn/attach/schedule/goal/heartbeat/send…) → `CONTRACTS.md` §C4. Produced by WS1.
- **C5** — session-artifact paths + entry types (scheduled-jobs store, goal/job-outcome custom entries, session lease) → `CONTRACTS.md` §C5. Produced by WS1.

## Status (per SPEC §13 / HANDOFF)

IMPLEMENTED v1.0 — WS1–WS7 + WS7a landed in
`~/Work/github/oh-my-pi/packages/coding-agent`, per-workstream reviewed+tested,
whole-solution Opus review verdict **READY**. Uncommitted on `main` atop
upstream `0a912cc46`; no branch/PR yet (user wants a run-in period first).

## The implementation codebase

Repository: **https://github.com/Predator404/oh-my-pi-agent** — the OMA fork
(of OMP `github.com/can1357/oh-my-pi`; local checkout `~/Work/github/oh-my-pi`).
This is code, separate from the (deletable) working folder that held the docs.

- Entity registry/loader/setup: `packages/coding-agent/src/entity/` (`schema.ts`,
  `loader.ts`, `setup.ts`, `record-writer.ts`, `mcp-wiring.ts`).
- CLI surface: `src/commands/entity.ts` + `src/cli/entity-cli.ts` (`oma entity …`).
- Daemon graft: `src/launch/` (broker + `agents/control-protocol.ts`).
- `oma` identity/shim: `src/oma.ts`, `src/oma-identity.ts`.

For upstream/harness behavior these build on, cross-read `../oh-my-pi/docs/`
(broker, sessions, MCP, memory) and `../prime-agent/docs/` (daemon,
long-running-agents, session-format).

## Durability

This section is the **home of record** for the project's design docs — there is
no external repo to re-sync from, and the authoring working folder may be
deleted. Edit these files here directly to keep them current.

_Copied into the vault 2026-08-18. Authoritative here._
