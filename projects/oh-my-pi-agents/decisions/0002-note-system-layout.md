---
type: adr
created: 2026-08-19
tags: [adr, decision, vault, note-system]
---

# ADR 0002 — Vault note-system layout: persona-docs vs project-dev split

## Status

Accepted 2026-08-19.

## Context

We need to separate "documentation of how OMA functions" from "notes on further
developing OMA", while keeping links between them.

## Decision

Keep the design/reference corpus (SPEC/CONTRACTS/HANDOFF + upstream doc mirrors)
in the persona section `personas/phi/oh-my-pi-agent/`; put forward-development
notes (decisions, investigations, dev status) in a **new** project section
`projects/oh-my-pi-agents/` in the **same** vault; connect them with Obsidian
wikilinks.

Do **not** split into a separate vault/repo — that would break the cross-section
link graph and semantic search (SPEC resolved-item 1: split the vault out only
on size/attachments/sync pressure).

Three stores, divided by job:

- **vault** — long-form prose (both sections)
- **`phi` memory bank** — terse curated lessons
- **project repos** — thin pointer stubs only

## Consequences

Working-dir doc duplicates collapsed to pointer stubs; the vault is the single
source of truth. Semantic search and the link graph span docs + dev.

See [[projects/oh-my-pi-agents/README.md|README]] for the section layout and [[phi.md]] for the project note.
