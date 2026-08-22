---
type: reference
created: 2026-08-19
tags: [reference, investigation, memory, embeddings]
---

# Phi memory bank — dense embeddings not contributing

## Source
- Repo / upstream: https://github.com/Predator404/oh-my-pi-agent (OMA — `oma` build)
- Docs: [[personas/phi/oh-my-pi/docs/mnemosyne-memory-backend.md]] — how the memory backend is supposed to work
- Mirrored / verified: 2026-08-19

## Symptom

A `recall` against the `phi` memory bank returns matches, but purely via lexical
voices — the recall result showed `fts_score: 1`, `keyword_score: 0.769`, but
`dense_score: 0` and `embed_text: null`, i.e. the dense/vector embedding voice
scored 0. The vault Smart Connections index, by contrast, produces working
semantic embeddings (verified: block-level hits ~0.55–0.60). So the gap is
specific to the memory-bank (mnemopi) embedding path, not the vault.

## Impact

Memory-bank recall is lexical-only until fixed; semantic recall on curated
lessons is degraded.

## Next

Investigate why the `phi` bank stores no dense embeddings (embedder not running
/ not wired at retain time / model-dim mismatch). This is an OMA-development
concern.

## Links

- [[personas/phi/oh-my-pi/docs/mnemosyne-memory-backend.md]] — how the memory backend is supposed to work
