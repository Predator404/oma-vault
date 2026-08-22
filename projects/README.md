# Vault — project sections (`projects/`)

Per-project knowledge, shared once and surfaced into thin project repos via
**project pointers** (SPEC §4.3, §7.3; produced by WS6).

## Layout

```
vault/
  agents/<name>/            # per-agent notes (WS4)
  personas/<name>/          # per-persona domain notes (WS4)
  projects/
    <project>/
      <persona>.md          # one note per entity that carries project knowledge
      <other-persona>.md
```

- `<project>` — the project id. Defaults to the basename of the project repo
  directory (overridable when generating the pointer).
- `<persona>` — the entity segment, taken from the entity's C1 `vaultSection`
  (`personas/phi` → `phi`, `agents/atlas` → `atlas`).
- One Markdown note per `(<project>, <persona>)` pair. This note is the single
  source of truth for that entity's knowledge of that project — it is **never**
  copied into the project repo.

## How a project repo references a note

The project repo carries only a pointer stub — no content:

```
<project-repo>/.omp/skills/<persona>-project/SKILL.md
---
name: <persona>-project
description: ...
---
@~/vault/projects/<project>/<persona>.md
```

The stub body is a single native OMP `@`-import. At read time OMP expands it
against `~/` → home, following the per-machine `~/vault` symlink to wherever the
vault physically lives (sibling checkout, external drive, synced folder). The
symlink is created and validated by the setup step (WS7a); the stub itself stays
byte-identical across machines and checkouts.

## Surfacing into an entity's context

List the pointer's skill name (`<persona>-project`) in the entity record's
`autoloadSkills` (C1). When the resident worker runs in the project directory,
OMP's native skill discovery finds the stub under `.omp/skills/`, and the
autoload machinery injects its (expanded) body — the entity sees the vault note
without the repo ever storing it.

## Generation

Stubs are generated programmatically (WS6), not hand-written:

```ts
import {
  generateProjectPointer,
  generateProjectPointerForEntity,
} from "@oh-my-pi/pi-coding-agent/project-pointer";

// Explicit persona/project:
await generateProjectPointer({ projectDir, persona: "phi", project: "oh-my-pi" });

// Resolve persona from an entity's C1 vaultSection:
await generateProjectPointerForEntity({ projectDir, entityName: "phi", registryRoot });
```

Both are idempotent — re-running with the same inputs is a no-op, never a
clobber.

## Note conventions

Each `projects/<project>/<persona>.md` note uses the **`Templates/Project Note.md`**
folder template (auto-applied by Templater on empty-note creation; fill the
`project` and `persona` frontmatter). Generated pointer stubs are unaffected —
they live in the project repo, not the vault. When creating these notes
programmatically, seed from that template. See [`AGENTS.md`](../AGENTS.md) and
[`reference/obsidian/vault-conventions.md`](../reference/obsidian/vault-conventions.md).
