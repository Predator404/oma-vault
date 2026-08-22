# Project — OMA forward development (`projects/oh-my-pi-agents/`)

The **forward-development workspace** for OMA — the `oma` build (a persistent
multi-entity runtime on top of OMP), fork
[Predator404/oh-my-pi-agent](https://github.com/Predator404/oh-my-pi-agent).
This section holds *notes on further developing OMA*: decisions in flight,
investigations, and dev status.

> This is **not** the design corpus. How OMA is *designed* and how it *functions*
> lives in Phi's persona section as source-grounded reference:
> [[personas/phi/oh-my-pi-agent/SPEC.md]] (buildable spec — runtime §8,
> workstreams §12, status §13), [[personas/phi/oh-my-pi-agent/CONTRACTS.md]]
> (frozen seams C1/C4/C5), and [[personas/phi/oh-my-pi-agent/HANDOFF.md]]
> (decision trail + CURRENT STATE). This project section is a *separate section
> in the same vault*, so Obsidian wikilinks and Smart-Connections recall span
> both — read the persona corpus for the "how it works", edit here for the
> "what we're changing next".

## Layout

```
projects/oh-my-pi-agents/
  README.md          ← this index
  phi.md             ← Phi's single source of truth for the OMA-dev project
                       (the project-pointer target)
  follow-ups.md      ← PR follow-up register (recorded at merge; vault convention)
  decisions/         ← ADRs (Status / Context / Decision / Consequences)
    0001-mention-addressing.md
    0002-note-system-layout.md
    0003-pr-follow-up-register.md
  investigations/    ← open technical investigations
    dense-embeddings.md
    prompt-wake-gap.md
```

## Active dev threads

- **`@@name:` mention-addressing** — a `@@<agentName>: <message>` convention to
  direct a message at a specific entity. Case A (address the in-session advisor)
  being built now; Case B (address a different resident worker) deferred. See
  [[decisions/0001-mention-addressing.md]].
- **Prompt-wake gap** — a detached idle resident worker is not woken by a C4
  `oma entity prompt`. Port-introduced regression; blocks mention-addressing
  Case B. See [[investigations/prompt-wake-gap.md]].
- **Dense embeddings not contributing** — the `phi` memory bank recalls
  lexically only; the dense/vector voice scores 0. See
  [[investigations/dense-embeddings.md]].

## Links

- [[phi.md]] — the project note (Phi's knowledge of this project)
- [[personas/phi/oh-my-pi-agent/SPEC.md]] · [[personas/phi/oh-my-pi-agent/CONTRACTS.md]] · [[personas/phi/oh-my-pi-agent/HANDOFF.md]] — design/reference corpus
- [[projects/oh-my-pi-agents/follow-ups.md|follow-ups]] — PR follow-up register; convention in [[decisions/0003-pr-follow-up-register.md]]
