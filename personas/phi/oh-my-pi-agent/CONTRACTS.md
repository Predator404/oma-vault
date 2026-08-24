---
title: Frozen Cross-Workstream Contracts (C1, C4, C5)
status: FROZEN v1 — grounded in codebase scouts (PrimeDaemonScout, OmpGraftScout)
companion: SPEC.md §6 (contract list), §8 (runtime), §12 (workstreams)
sources:
  port-source: ~/Work/github/PrimeAgent  (Prime Agent, MIT)
  port-target: ~/Work/github/oh-my-pi     (OMP fork, repo 1 / core)
---

# Frozen Contracts

These freeze the seams between parallel workstreams. Every path below is verified against the two
local codebases via read-only scouts. Wave-2+ dev agents implement AGAINST these; do not renegotiate
without updating this file first.

## Architecture decision (both scouts converged)

**Extend OMP's existing daemon broker; do NOT stand up a parallel supervisor.**
- OMP already ships `packages/coding-agent/src/launch/broker.ts` (+ `launch/protocol.ts`) — a
  process-local daemon broker for LSP/browser sharing with spawn/readiness/restart/log/signal routing.
- OMP already ships residency + peer substrate: `registry/agent-registry.ts` (running→idle→parked→released),
  `registry/agent-lifecycle.ts` (TTL park/revive: park disposes live session + keeps `sessionFile`; revive
  re-creates from file), and `irc/bus.ts` (process-global mailbox; delivery survives detach because the
  worker session is the sink, not the terminal).
- The port therefore = extend broker to spawn **resident agent-session workers** + adopt Prime's
  daemon protocol command set + scheduler/goals/autonomous + peer-message routing. OMP's tool registry,
  skills, MCP, memory backends are untouched (verified isolated from session lifecycle).

Prime source modules that port (no ipython coupling — verified): `modes/daemon/daemon-supervisor.ts`,
`modes/daemon/daemon-protocol.ts` (v7), `modes/daemon/daemon-worker-protocol.ts`, `core/session-lease.ts`,
`core/agent-session-runtime.ts`, `core/cron-jobs.ts`, `core/goals.ts`, `core/autonomous.ts`,
`core/agent-messages.ts`. DROP (ipython-coupled): `kernel/`, `rlm-runtime.ts`, RLM extension hooks,
`refinement.ts`.

---

## C1 — Entity registry record schema (produced by WS2)

Extends OMP's existing agent-def frontmatter (loaded today from `~/.omp/agent/agents/*.md`;
registry in `packages/coding-agent/src/registry/persisted-agents.ts`). Existing fields reused:
`name`, `description`, `model`, `systemPrompt`, `tools`, `autoloadSkills`.

New/formalized fields:
- `role: "agent" | "persona"` — formal enum (was inferred). Drives memory-retention policy.
- `memory: { backend: "mnemopi"; bank: <string>; autoRetain: boolean }` — bank binding (C2). `autoRetain`
  MUST be `true` only for `role: agent`; `persona` records reject `autoRetain: true` at load (curated-only).
- `vaultSection: <path>` — owned vault subtree (e.g. `agents/<name>` | `personas/<name>`), §7.
- `registry: <id>` — home registry id (ADR 0004; see [[projects/oh-my-pi-agents/decisions/0004-registry-domains.md]]). Key into the registries manifest that resolves the writable vault root, the readable set (own + public + granted-private), the records root, and the memory-bank namespace. `vaultSection` is relative to the home registry's root. Populated by the loader from the record's registry; a `private` registry's `memory.bank` MUST be namespaced `<id>/<bank>`.
- `watchdog?: <WATCHDOG.yml entry>` — optional standing-advisor config.
- `hosting?: { modelEndpoint?: <provider-id> }` — optional local/cloud pin (§9; default deferred).

Consumers: WS1 (worker launch resolves record → AgentSession config), WS3 (bank binding), WS4 (vault
section), WS7 (CLI/TUI edit). Loader lives in repo 2 (registry); worker reads resolved record.

## C4 — Daemon control protocol (produced by WS1)

Baseline types: OMP `launch/protocol.ts` (`DaemonSpec`, `DaemonSnapshot`, `DaemonOperation`,
`DaemonWireRequest/Response`). Extend `DaemonOperation` with the agent-session command set adopted from
Prime `daemon-protocol.ts` (v7):

Commands (client→broker): `spawn(entityName)`, `attach(id)`, `detach(id)`, `list()`, `stop(id)`,
`prompt(id, text)`, `steer(id, text)`, `follow_up(id, text)`, `send_message(target, text, mode)`,
`schedule_add(id, spec)`, `schedule_list(id)`, `schedule_cancel(jobId)`, `heartbeat_set/pause/resume/clear(id)`,
`goal_set/status/pause/resume/clear(id)`, `autonomous_on/off/status(id)`.

Envelopes (from Prime v7, adopt as-is):
- Command envelope: `{ type, id, protocol, clientId, command }`.
- Event envelope: `{ type, id, protocol, activeSessionId, sequence, cursor, emittedAt, event }`.
- Event cursor: `{ generation, sequence }`. Attach snapshot = summary + messages + session tree + replay
  info, streamed in 512 KiB chunks. Capability-gated (`attach_snapshot`, `event_sequence`, `slim_attach`,
  `chunked_snapshot`, `client_owned_sessions`).
- **Idempotency:** mutating commands keyed `{clientId + commandId}`, journaled before dispatch; uncertain
  results reported, never replayed.
- **Reconnect/resume:** client presents last `{generation, sequence}`; server reports replay status
  `complete | partial | unavailable`; attach snapshot is the durable baseline.

Private worker↔broker transport (adopt Prime `daemon-worker-protocol.ts`): 4B JSON-header-len + 4B
payload-len + routing header + opaque payload; per-worker tokens fenced to supervisor generation; worker
lifecycle `starting|ready|recovering|stopping|failed`.

Consumers: WS5 (peer delivery over `send_message` + `irc/bus.ts`), WS7 (CLI/TUI surface).

## C5 — Session-artifact paths + entry types (produced by WS1)

OMP session JSONL is append-only, tree-navigable via `id`/`parentId`
(`packages/coding-agent/src/session/session-entries.ts`, ~15 types: message, thinking_level_change,
model_change, service_tier_change, compaction, branch_summary, reset_boundary, custom, custom_message,
label, title_change, ttsr_injection, credential_pin, session_init, mode_change). Storage path:
`~/.omp/agent/sessions/<encoded-cwd>/<timestamp>_<sessionId>.jsonl` (`session/session-manager.ts`,
`SessionStorage`).

Adaptation (do NOT invent Prime's entry types — use OMP custom entries with namespaced `customType`):
- Scheduled-job store: `session-artifacts/<session-id>/scheduled-jobs.json` (port Prime `core/cron-jobs.ts`
  store shape: job = `{ id, type: once|cron|interval, source: cron|heartbeat, prompt, schedule, nextRunAt,
  lastRunAt, status, deliveryMode: steer|follow_up }`; claim-and-advance tick before injection).
- Goal state entry: `customType: "omp.daemon.goal_checkpoint"` (port Prime `core/goals.ts` fields: goalId,
  objective, status idle|active|paused|budget_limited|complete|error, tokenBudget, tokensUsed,
  timeUsedSeconds, continuationsUsed).
- Job outcome entry: `customType: "omp.daemon.job_outcome"` (job id, reason, final turn state, next job id
  if continued) — mirrors OMP's existing `session_exit` diagnostic pattern (`session/exit-diagnostics.ts`).
- Session ownership: port Prime `core/session-lease.ts` atomic file lease → OMP session paths
  (`~/.omp/agent/session-leases/<hash>.lock`); `SessionAlreadyActiveError` on conflict.

All scheduler modes (heartbeat/cron/goal/autonomous) inject a normal prompt into OMP's `YieldQueue`
(`session/yield-queue.ts`; `flush(mode)` is the graft boundary — detached sessions flush idle messages to
a persistent queue instead of a client stream). No kernel path — this is why dropping ipython costs nothing.

---

## Graft risks (from OmpGraftScout — carry into WS1 acceptance)

1. Extension teardown: `emitSessionShutdownEvent` runs handlers with a 30s timeout each; managed timers
   (`extensibility/extensions/managed-timers.ts`) auto-clear on dispose. Verify a detach (park) does not
   block on a long extension handler. Mitigation exists (managed timers), confirm it holds under park.
2. Signal handling: OMP `session/agent-session.ts` dispose invokes with `postmortem.Reason('signal')` on
   SIGTERM/SIGINT. Resident worker must route client-requested stop as a C4 `stop`, not a raw signal to the
   session, to preserve clean park/dispose semantics.
3. No atexit/global cleanup hooks in OMP session (verified) — safe for resident worker, but the worker
   process itself must own graceful drain before exit.

## Dispatch order (SPEC §12.8, now grounded)

- **Wave 2 (foundation, contract-producing):** WS1 (daemon-port foundation → freezes C4/C5 in code),
  WS2 (entity registry → freezes C1 in code). Parallel.
- **Wave 3 (fan-out, after C1/C4 code-frozen):** WS3 (memory MCP), WS4 (vault MCP), WS5 (peer messaging),
  WS6 (project pointer), WS7 (CLI/TUI + setup). Parallel.
- **Integration:** Opus reviewer + tester over whole solution (SPEC §12.9).
- Each workstream: dev → dedicated code reviewer → dedicated tester → iterate to acceptance. Workstream
  agents skip project-wide lint/build/test during their pass (run once at integration) and coordinate
  contract questions over `hub` rather than serializing.
