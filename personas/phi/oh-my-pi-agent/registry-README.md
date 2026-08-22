# Entity registry (repo 2)

The **roster** of the persistent multi-entity agent system: one lightweight
record per entity. This is pure data — adding, editing, or removing an entity is
a file change, never a code change (SPEC §12.2 acceptance). The loader, schema,
and record→session resolver live in **repo 1 / core** (`oh-my-pi`,
`packages/coding-agent/src/entity/`); this repo holds only records.

This is the concrete producer of **contract C1** (see `../CONTRACTS.md`).

## On-disk layout

```
registry/
  README.md            # this file
  entities/
    <name>.md          # one entity record: YAML frontmatter (C1 metadata) + Markdown body (system prompt)
  vault/               # scaffolded by `omp entity setup`; the ~/vault symlink target (SPEC §7)
    agents/  personas/  projects/
  endpoints.yaml       # optional: local model endpoints registered as OMP providers (SPEC §9)
```

- One Markdown file per entity, named `<name>.md`, where `<name>` is the
  entity's stable id and **must** equal the frontmatter `name`.
- The record extends OMP's existing agent-definition frontmatter, so the format
  is the same one OMP already loads from `~/.omp/agent/agents/*.md`.
- The **system prompt is the Markdown body**, loaded **on demand** at launch —
  never eagerly during roster discovery (SPEC §5 context-cost discipline). The
  roster scan (`discoverEntities`) reads frontmatter only.

## Registry root resolution

The loader finds this directory via, in order:

1. an explicit `registryRoot` passed by the caller (used by tests);
2. the `OMP_ENTITY_REGISTRY` environment variable;
3. the default `~/.omp/agent/registry`.

`omp entity setup` wires the default location to this repo-2 checkout (a symlink
`~/.omp/agent/registry → <this repo>/registry`) and creates + validates the
`~/vault → <this repo>/registry/vault` symlink. It is idempotent (re-running is a
no-op). Before running setup, point `OMP_ENTITY_REGISTRY` at this folder.

### `endpoints.yaml` (SPEC §9)

Optional. Declares local model endpoints (Ollama/vLLM/LM Studio, OpenAI-compatible
base URLs) that `omp entity setup` merges into `~/.omp/agent/models.yml` as
providers (additive + idempotent). Shape mirrors OMP's `models.yml` `providers`:

```yaml
providers:
  ollama-local:
    baseUrl: http://127.0.0.1:11434/v1
    api: openai-completions
    discovery: { type: ollama }
```

## Record schema (contract C1)

Frontmatter fields. Reused OMP agent-definition fields plus the formalized C1
extension.

| Field            | Type                                             | Req. | Notes |
|------------------|--------------------------------------------------|------|-------|
| `name`           | string, `^[a-z0-9][a-z0-9._-]*$`                 | yes  | Stable, filesystem-safe id; must equal the filename. |
| `description`    | string                                           | yes  | One-line roster description. |
| `role`           | `"agent" \| "persona"`                           | yes  | Formal enum. Drives the retention policy (below). No silent default. |
| `icon`           | string (single glyph)                            | no   | Display marker (emoji/char) for this entity's label; validated as exactly one grapheme cluster. |
| `color`          | ThemeColor token                                 | no   | Theme-color token (e.g. `accent`, `success`, `warning`, `error`) tinting the entity's label/marker. Adapts to the active theme. |
| `model`          | string[] (or CSV)                                | no   | OMP model selector list; may target a local endpoint. |
| `thinkingLevel`  | string                                           | no   | OMP thinking/effort selector (alias `thinking`). |
| `tools`          | string[] (or CSV)                                | no   | OMP tool grant list (built-in + MCP). |
| `autoloadSkills` | string[] (or CSV)                                | no   | Skills auto-loaded into the session. |
| `memory`         | `{ backend: "mnemopi"; bank: string; autoRetain?: boolean }` | yes | Bank binding (C2). `autoRetain` defaults `false`. |
| `vaultSection`   | string                                           | yes  | Owned vault subtree, e.g. `agents/<name>` or `personas/<name>` (§7). |
| `watchdog`       | WATCHDOG.yml advisor entry `{ name, model?, tools?, instructions?, enabled? }` | no | Optional standing advisor. |
| `hosting`        | `{ modelEndpoint?: string }`                     | no   | Optional provider-id pin (§9). Local placement deferred. |

Body: the entity's full system prompt (identity / voice / remit). Required,
non-empty.

## Role → retention policy (enforced at load)

Role sets the memory-retention policy, not a capability ceiling (SPEC §2, §4.1):

- **agent** — generalist/coordinator. Retains **episodic** memory across
  sessions. `memory.autoRetain: true` is allowed.
- **persona** — narrow specialist. Retains **curated domain lessons only**
  (deliberate writes). `memory.autoRetain: true` is **rejected at load** with a
  clear error — never silently coerced. Set `role: agent` if you need episodic
  retention.

## Resolution seam (WS2 → WS1)

The resident-worker launch calls:

```ts
resolveEntityConfig(name: string, opts?: { registryRoot?; cwd? }): Promise<ResolvedEntityConfig>
```

which loads the record body on demand, validates, enforces the retention
policy, and returns a launchable config that maps onto OMP's
`CreateAgentSessionOptions` (`model`→`modelPattern`, `systemPrompt`→
`customSystemPrompt`, `tools`→`toolNames`, `thinkingLevel`→`thinkingLevel`,
`cwd`→`cwd`).

## Records

- **`entities/phi.md`** — Phi, the resident OMP/pi expert **persona** (SPEC
  resolved-item 3). The reference record; exercises every C1 field.
