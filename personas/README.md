# Vault — persona sections (`personas/`)

Per-persona domain knowledge (WS4). One subtree per persona entity
(`personas/<name>/`), matching the entity's C1 `vaultSection`. Personas retain
**curated domain lessons only** — deliberate, source-grounded writes.

Current sections:

- [`phi/`](phi/README.md) — Phi, the OMA/OMP harness expert (see its README for
  the doc-mirror + codebase layout).

> A persona's C1 **record** lives in the registry at
> `registry/entities/<name>.md`, **not** here. This area holds the persona's
> *domain notes*, not its definition.

## Note conventions

New notes in this area use the **`Templates/Persona Note.md`** folder template
(auto-applied by Templater on empty-note creation). The verbatim upstream doc
mirrors under `phi/` (`oh-my-pi/`, `prime-agent/`, `pi/`, `oh-my-pi-agent/`) are
excluded from templating. See [`AGENTS.md`](../AGENTS.md) and
[`reference/obsidian/vault-conventions.md`](../reference/obsidian/vault-conventions.md).
