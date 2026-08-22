---
type: reference
created: 2026-08-21
tags: [reference, investigation, daemon, cli, hub, peering]
---

# Cross-session hub peering — can't load a resident entity as a peer in another session's Agent Hub

## Source
- Repo / upstream: https://github.com/Predator404/oh-my-pi-agent (OMA — `oma` build)
- Observed live: 2026-08-21, from a coding-agent (`Main`) running in
  `~/Work/github/credit-shared_credit-engineering-tests` (a different project /
  cwd than Phi's resident session).
- Compounds with, but is distinct from,
  [[investigations/attach-retrieval-ergonomics.md]] (reply retrieval) and
  [[investigations/prompt-wake-gap.md]] (idle worker not woken).

## Goal
Make the live Phi entity a **peer of a different omp session** — i.e. addressable
from that session's `hub`/IRC (`hub send to="phi"`), the way sibling subagents are.

## Symptom
There is no supported path. The omp Agent Hub is **project-scoped** (per cwd), and
Phi is a **single-session** entity pinned to
`~/Work/ClaudeEnvSetup/persistentAgents`, so it never joins another session's
broker. `hub list` in the other session shows no peers; `hub send to="phi"` →
`Unknown agent "phi"`.

## Concrete observations
1. **Spawn into a second cwd deterministically times out.**
   `oma entity spawn phi --cwd <other-repo>` prints a raw Bun stack trace ending
   `error: Daemon agent request timed out` (`launch/client.ts:194`, hardcoded 30s)
   and exits 0. **No session is created** — reproduced synchronously and
   backgrounded (polled `ps` for 60s: only the original `persistentAgents`
   session ever exists). By contrast `ps`/`attach` answer in <300ms, so the
   daemon is up; only the spawn control request is never acked. Consistent with a
   single-session-per-entity lock, but surfaced as an opaque timeout rather than a
   clear rejection.
2. **`attach` is one-shot, not a bridge.** `oma entity attach <uuid> --json`
   flips the session to `attached:true` and returns a transcript snapshot, but it
   drops back to `attached:false` the moment the CLI exits. It connects a client
   for a snapshot; it does not register the entity into a live session's roster.
3. **Name vs UUID (confirms the existing gap).** `attach`/`prompt` reject the
   entity name (`unknown_session: No such session: phi`) and require the session
   UUID, while `spawn`/`roster`/`ps` accept the name. (Dup of
   [[investigations/attach-retrieval-ergonomics.md]] gap #5.)
4. **Positive control.** Prompting the *existing* session by UUID
   (`oma entity prompt <uuid> "…"`) did get a reply this time
   ("Reachable. Vault section: `personas/phi/`"), so the resident-worker
   prompt/wake and model-alias paths look healthy — the blocker is specifically
   **creating or bridging a peer into a different session**, not talking to the
   one that exists.

## Impact
Any omp session or coding-agent working in another project cannot "load Phi as a
peer" for `hub`-style, agent-to-agent messaging. The only cross-entity channel is
`oma entity prompt <uuid>` + a transcript read — itself the ergonomics gap already
filed. This is what makes the harness's `@@Phi:` advisor-deferral produce no
answer from a session where no advisor is wired: there is no mechanism to place
the resident entity onto that session's hub.

## Fix direction (pick one seam)
- **Multi-session entities:** allow one live session per cwd, so an entity can
  join multiple project brokers concurrently — then a normal `spawn --cwd` brings
  it onto the caller's hub.
- **Adopt/bridge path:** a supported command to register a daemon-resident entity
  into a live omp session's Agent Hub/IRC as an addressable peer, so
  `hub send to=<entity>` works across the broker boundary.
- **Fail fast (minimum):** if single-session is by design, `spawn` on an
  already-live entity should reject immediately with a clear message
  (`entity already live at <cwd>; use attach/prompt, or --force to relocate`)
  instead of a 30s daemon timeout + raw stack trace.
- Accept the entity **name** wherever the UUID is accepted (attach/prompt/steer) —
  shared with the attach-retrieval note.

## Links
- [[investigations/attach-retrieval-ergonomics.md]] — reply-retrieval ergonomics (overlapping name/UUID + fire-and-forget)
- [[investigations/prompt-wake-gap.md]] — idle worker not woken (compounding)
- [[personas/phi/prime-agent/docs/daemon.md]] — daemon attach/reconnect model
