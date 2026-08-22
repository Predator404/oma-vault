---
type: adr
created: 2026-08-19
tags: [adr, process]
---

# PR follow-up register — capture outstanding follow-ups at merge

## Status

Accepted 2026-08-19. Establishes a **vault-wide** convention (applies to every
project, not just this one).

## Context

A PR can merge with non-blocking review findings, deferred scope, TODOs, or
tests noted-but-not-written. Once merged the PR is closed, and those items live
only in a review transcript or someone's memory — they vanish in the gap between
*merged* and *fully done*. The project needs a durable, browsable, searchable
home for that gap, and it should happen for **any** project automatically, not
by remembering to do it each time.

## Decision

When a PR is merged, its still-outstanding follow-ups are recorded in that
project's `projects/<project>/follow-ups.md` (a living register — one section
per PR, checkbox items) **before the PR is treated as closed-out**.

- **Any project, going forward.** The convention is documented at the vault
  level: [[reference/obsidian/vault-conventions.md]] ("PR follow-up register")
  and vault [[AGENTS.md]] rule 5, and the `Templates/Project Note.md` template
  points at each project's register. Every project keeps its own
  `follow-ups.md`, created on first merge if absent.
- **Automatic enforcement via Phi.** Phi (advisor) rides every OMA session in
  every project, so a PR merge anywhere prompts the capture; her doctrine
  ([[personas/phi/advisor-guidance.md]]) makes it a standing cross-project duty.
  This is OMA's "automatic" — an always-in-the-loop agent — rather than a git
  hook (a hook cannot derive review findings; see Consequences).
- **Sources:** non-blocking Standards/Spec review findings, deferred scope,
  TODOs, and "verified live but not tested" gaps.
- **Deep threads** get their own `investigations/<name>.md`, linked from the
  register rather than restated.

## Consequences

Follow-ups survive the merge and stay discoverable + semantically searchable in
each project section. Closing an item is a checkbox flip, ideally by the PR that
resolves it. The register is the single source; the project README and the
`<persona>.md` note link to it. Seeded here with PR #1 — see [[follow-ups]].

A hard git/`gh` post-merge hook is a possible future hardening, but it can only
stub the PR header (number, merge time); it cannot derive the review findings
that make up the useful follow-ups, so the Phi-enforced capture stays primary.
