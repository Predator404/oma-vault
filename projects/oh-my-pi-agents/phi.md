---
type: project-note
project: oh-my-pi-agents
persona: phi
created: 2026-08-19
tags: [project]
---

# phi

> Area `projects/oh-my-pi-agents/phi.md` — this entity's single source of truth
> for the project. Never copied into the project repo; surfaced via a
> project-pointer skill (`.omp/skills/phi-project/SKILL.md`, a native `@`-import
> of this note).

## Goal

Further development of OMA — the persistent multi-entity agent build on OMP.
This note is Phi's working memory for that forward development: what is being
built, decided, and investigated. The design of record (how OMA functions) is
the persona corpus, not this note; link back to it rather than restating it.

## Knowledge & decisions

- **`@@name:` mention-addressing** — Case A (address the in-session advisor,
  e.g. `@@Phi:`) accepted and being built; Case B (address a different resident
  worker over the daemon) deferred. See [[decisions/0001-mention-addressing.md]].
- **Note-system layout** — persona design corpus vs project-dev split, one
  vault, connected by wikilinks. See [[decisions/0002-note-system-layout.md]].
- **Dense embeddings not contributing** — `phi` memory-bank recall is
  lexical-only; the dense voice scores 0. See
  [[investigations/dense-embeddings.md]].
- **Prompt-wake gap** — detached idle worker not woken by a C4 prompt;
  port-introduced regression; blocks mention-addressing Case B. See
  [[investigations/prompt-wake-gap.md]].
- **PR follow-up register** — outstanding follow-ups from merged PRs are
  recorded in [[follow-ups]] at merge; vault-wide convention. See
  [[decisions/0003-pr-follow-up-register.md]].

## Open questions

- Why does the `phi` bank store no dense embeddings (embedder not running / not
  wired at retain time / model-dim mismatch)? — [[investigations/dense-embeddings.md]]
- When does the peer-reply return channel (SPEC WS5) land, unblocking
  mention-addressing Case B? — [[decisions/0001-mention-addressing.md]]

## Links

- [[projects/oh-my-pi-agents/README.md|README]] — the project section index
- Design/reference corpus (how OMA functions):
  [[personas/phi/oh-my-pi-agent/SPEC.md]] ·
  [[personas/phi/oh-my-pi-agent/CONTRACTS.md]] ·
  [[personas/phi/oh-my-pi-agent/HANDOFF.md]]
