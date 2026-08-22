---
type: reference
created: 2026-08-19
tags: [reference, investigation, daemon, prompt-wake]
---

# Prompt-wake gap — detached idle worker not woken by C4 prompt

## Source
- Repo / upstream: https://github.com/Predator404/oh-my-pi-agent (OMA — `oma` build)
- Docs: [[personas/phi/prime-agent/docs/daemon.md]] — daemon delivery model (Prime Agent lineage)
- Mirrored / verified: 2026-08-19

## Symptom

A one-shot `oma entity prompt <id> "..."` to a **detached, idle** resident
worker enqueues the message but does not wake the worker to run a turn (busy
stays false, no assistant reply). Interactive `attach` and the
scheduler-injection path (heartbeat/goal) both work.

## Root cause / origin

**Port-introduced regression**, not inherited from Prime Agent. Prime's delivery
path (`modes/daemon/daemon-mode.ts`) deliberately resumes idle sessions —
`session.followUp(..., { resumeIfIdle: true })` and an awaited
`session.prompt(...)`. OMA re-implemented the C4 command dispatch in
`packages/coding-agent/src/launch/agents/agent-worker.ts` (~lines 191–203) with
a hand-written branch that fires a **non-awaited** `void session.prompt(text)`
on the idle case, bypassing OMA's own correct abstraction
`SessionPromptInjector.injectPrompt()`
(`launch/agents/resident-session.ts:171–185`, which awaits `session.prompt` on
idle and is what the working scheduler path uses). `resumeIfIdle` is a
Prime-only option (zero matches in OMA source), so the port rewrote the seam and
lost the idle-wake.

## Confidence / still to trace

Confirmed: the **origin** is the fork-written C4 dispatch seam — it diverges
from both Prime's resume-idle delivery and OMA's own working
`SessionPromptInjector` (the scheduler path). Evidence is the code itself
(`agent-worker.ts` idle branch vs `resident-session.ts:171–185`) plus the
observed asymmetry (scheduler injection wakes; C4 `prompt` does not).

Not yet traced to a single line: *why* a non-awaited `void session.prompt()`
fails to actually run a turn on a **detached, idle** worker, when the awaited
injector call (`await session.prompt()`) on the same session does. `await` vs
`void` alone should not change whether the turn starts — so there is a further
detail (e.g. an early throw swallowed by `.catch()`, or a pump/attach
precondition that only the injector path satisfies) to pin down when the fix is
written. Do not assume the one-line `void → injectPrompt` swap is sufficient
until that turn-start path is confirmed under a detached idle worker.

## Fix direction

Route the C4 `prompt` command through `SessionPromptInjector.injectPrompt()` (or
at least await it) instead of the bespoke `void prompt()` branch. Same
daemon-delivery domain as PR #1 but **not** in that PR.

## Links

- [[personas/phi/prime-agent/docs/daemon.md]] — daemon delivery model
- [[decisions/0001-mention-addressing.md]] — this is the Case B blocker
