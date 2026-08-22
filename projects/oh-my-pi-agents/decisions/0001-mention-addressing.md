---
type: adr
created: 2026-08-19
tags: [adr, decision, mention-addressing]
---

# ADR 0001 — `@@<agentName>: <message>` addressing convention

## Status

Accepted 2026-08-19; phased.

## Context

The user wants `@@name: msg` to direct a message at a specific persona/agent so
that entity responds directly. Investigation found this is **two features under
one syntax**:

- **Case A** — addressing the in-session advisor. Phi already advises this
  session via `advisor.enabled: true`.
- **Case B** — addressing a **different** resident-worker entity over the daemon.

The earliest clean interception hook is `before_agent_start` (fires after the
user submits, before the LLM call; can inject a message/systemPrompt but
**cannot** cancel the primary turn — only the `context` event can replace
messages).

The `@@` double-sigil deliberately avoids OMP's single-`@` path/import namespace
(e.g. `@~/vault/...`). No prior `@`-parsing exists in the codebase.

## Decision

Build **Case A now**: make the advisor addressable — recognize a leading
`@@Phi:` as a direct question, prime the advisor to answer promptly rather than
on its delta-cadence, and have the main agent defer.

Defer **Case B** until (i) the prompt-wake regression is fixed and (ii) a
peer-reply return channel exists (SPEC WS5).

Implementation lives in **core**, not an extension. The advisor subsystem is
core and always-on, and both touchpoints require it: the addressable-advisor
capability is appended to the advisor's own system prompt in
`session-advisors.ts`, and the primary deferral is injected into the primary's
`before_agent_start` system prompt in `agent-session.ts` (an extension
`before_agent_start` can inject but cannot make the primary stand down cleanly).
The parser is a small pure module (`advisor/address.ts`); the prompt text lives
in `src/prompts/advisor/{direct-address,primary-deferral}.md` (Handlebars), per
the repo's "no prompts in code" standard.

Trade-off (accepted): this touches the upstream-merge surface the original plan
wanted to avoid via an extension. Judged worth it — the feature is small,
always-on for the advisor, and the deferral genuinely needs core.

## Consequences

Case A covers the common "talk to Phi" need with no daemon/protocol work. Case B
is deferred, blocked on the prompt-wake gap (see
[[investigations/prompt-wake-gap.md]]) and a new return channel.
