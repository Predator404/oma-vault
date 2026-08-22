# Phi — vault knowledge base

Owned vault section for the **Phi** persona (`registry/entities/phi.md`,
`vaultSection: personas/phi`). Phi is the resident expert on the **Oh My Pi
(OMP)** / **pi** coding-agent harness and its sibling fork **Prime Agent**.

This section is Phi's source-of-truth corpus: verbatim mirrors of the three
upstream/sibling harnesses' documentation **plus this project's own design
docs**, indexed by Smart Connections for semantic recall (the vault MCP
`search_notes` / `get_note` path). Phi's working method requires grounding
every harness claim in real source — this is that source.

> Verbatim, self-contained copies — not curated prose, and not pointers. This
> vault is the durable keeper of this information and does not depend on any
> working folder. When docs lag code, consult the repos in **Codebases** below.

## Lineage (read this first)

All three harnesses descend from one upstream project by Mario Zechner:

```
              pi  (earendil-works/pi, formerly badlogic/pi-mono)
              │   packages: @earendil-works/pi-{coding-agent,agent-core,ai,tui}
              │   MIT · © 2025 Mario Zechner
      ┌───────┴────────┐
      ▼                ▼
  Oh My Pi          Prime Agent
  can1357/          PrimeIntellect-ai/
  oh-my-pi          prime-agent
  © Can Bölük       © Prime Intellect
  (this project's   (@earendil-works/
   base harness)     pi-coding-agent fork)
```

- **pi** is the shared root. OMP and Prime Agent are **sibling forks**, not
  competitors — same `packages/coding-agent` layout, same `SKILL.md` / skill
  discovery, same session-JSONL persistence idiom.
- **OMP** keeps pi's discrete-tool model (`read`/`edit`/`bash`/`task`/…) and
  adds a large ecosystem (marketplace, native MCP tool exposure, memory
  backends, advisor/watchdog, LSP, Agent Hub).
- **Prime Agent** replaced the tool surface with a single `ipython` kernel
  (RLM) and added the daemon/heartbeat/goal/schedule stack. This project's
  persistent-entity runtime **ports that daemon subsystem into OMP** while
  keeping OMP's tool model and dropping ipython/RLM (see `oh-my-pi-agent/SPEC.md` §8).
- **This project (Oh-my-pi-agent, the `oma` build)** is that port made real:
  OMP + the persistent-entity runtime. Its design/spec/contracts live in the
  `oh-my-pi-agent/` section below — the one section that is *not* upstream, but
  the system Phi is itself part of.

## Codebases (consult the source when docs lag)

The vault is self-contained — nothing here requires these repos to be present.
They are where Phi goes to verify a claim against real code:

| Section | Project | Repository |
|---------|---------|------------|
| `pi/` | pi — upstream root | https://github.com/badlogic/pi-mono → https://github.com/earendil-works/pi · https://pi.dev |
| `oh-my-pi/` | Oh My Pi (OMP) — base harness | https://github.com/can1357/oh-my-pi |
| `prime-agent/` | Prime Agent — daemon source | https://github.com/PrimeIntellect-ai/prime-agent |
| `oh-my-pi-agent/` | OMA — this project's code (fork of OMP) | https://github.com/Predator404/oh-my-pi-agent |

## Layout

```
personas/phi/
  README.md          ← this index
  oh-my-pi/          ← OMP docs (the fork this project builds on)
    README.md          project README
    CHANGELOG.md       coding-agent changelog
    docs/              full docs mirror (~129 md; tools/, toolconv/, skills/…)
  prime-agent/       ← Prime Agent docs (the daemon-subsystem source)
    README.md          project README
    docs/              full docs mirror (~35 md; daemon.md, long-running-agents.md,
                       rlm.md, architecture.md, session-format.md, extensions.md…)
  pi/                ← upstream pi (authoritative root)
    README.md          lineage note (this fork's relationship to pi)
    pi-upstream-README.md
    agent-core-docs/   pi-agent-core specs unique to upstream
      harness.md         AgentHarness durable-runtime spec (the storage/lane/
                         operation state-machine model everything inherits)
      search.md, agent-core-README.md, containerization.md
  oh-my-pi-agent/    ← THIS project (oma build) — design/spec/contracts
    README.md          section index + contract map (C1–C5)
    SPEC.md            buildable spec (runtime §8, workstreams §12, status §13)
    CONTRACTS.md       frozen seams C1/C4/C5 + architecture decision
    HANDOFF.md         decision trail + CURRENT STATE (resume here)
    registry-README.md C1 record schema + role→retention policy
    phi-entity-record.md  Phi's own reference C1 record
    mcp.json           project MCP wiring
```

## Writing notes in this vault

Honor the vault note conventions when creating notes here. Each area has a
Templater folder template (`personas/` → `Templates/Persona Note.md`); the
verbatim upstream mirrors listed above are excluded from templating. When writing
notes programmatically (not via the Obsidian UI), seed from the matching
`Templates/<Template>.md` and fill its frontmatter. Full rules:
[`../../AGENTS.md`](../../AGENTS.md) and
[`../../reference/obsidian/vault-conventions.md`](../../reference/obsidian/vault-conventions.md).
The Templater docs are mirrored at
[`../../reference/obsidian/templater/index.md`](../../reference/obsidian/templater/index.md).

## High-value entry points for Phi's remit

- **Harness runtime / durability** — `pi/agent-core-docs/harness.md` (the three
  stores, lanes, operation state machine, crash recovery), OMP
  `docs/session.md`, `docs/compaction.md`.
- **Tool surface** — OMP `docs/tools/*.md`, `docs/custom-tools.md`,
  `docs/bash-tool-runtime.md`, `docs/task-agent-discovery.md`.
- **Skills / rules** — OMP `docs/skills.md`, `docs/context-files.md`,
  `docs/rulebook-matching-pipeline.md`, `docs/marketplace.md`.
- **MCP** — OMP `docs/mcp-config.md`, `docs/mcp-runtime-lifecycle.md`,
  `docs/mcp-protocol-transports.md`, `docs/mcp-server-tool-authoring.md`.
- **Extensibility** — OMP `docs/extensions.md`, `docs/extension-loading.md`,
  `docs/hooks.md`; Prime `docs/extensions.md`.
- **Daemon / persistent agents** — Prime `docs/daemon.md`,
  `docs/long-running-agents.md`, `docs/architecture.md`,
  `docs/session-format.md`; OMP `docs/advisor-watchdog.md`, `docs/agent-hub.md`.
- **Memory** — OMP `docs/memory.md`, `docs/mnemosyne-memory-backend.md`,
  `docs/tools/{recall,retain,reflect}.md`.
- **pi ↔ OMP porting** — OMP `docs/porting-from-pi-mono.md`,
  `docs/porting-to-natives.md`.
- **This project (Oh-my-pi-agent)** — `oh-my-pi-agent/README.md` (contract map
  C1–C5), `oh-my-pi-agent/SPEC.md` (§8 runtime, §12 workstreams),
  `oh-my-pi-agent/CONTRACTS.md` (C4 daemon protocol, C5 session artifacts),
  `oh-my-pi-agent/HANDOFF.md` (CURRENT STATE — resume point).

## Origin & durability

This vault is the **authoritative, self-contained keeper** of everything here.
All content was copied in on **2026-08-18**; it does **not** point to or depend
on the working folder it was authored in (`~/Work/persistentAgents/`), which may
be deleted without losing anything in this vault.

Originally copied from (historical provenance, not a live dependency):

| Section | Copied from |
|---------|-------------|
| `oh-my-pi/` | local OMA checkout `~/Work/github/oh-my-pi` — docs + README + coding-agent CHANGELOG |
| `prime-agent/` | `~/Work/github/PrimeAgent/packages/coding-agent` — docs + README (images dropped) |
| `pi/` | `github.com/badlogic/pi-mono@main` — root README + `packages/agent` README/docs + containerization |
| `oh-my-pi-agent/` | project design docs — SPEC / CONTRACTS / HANDOFF / registry schema / Phi record / mcp.json |

Attribution (SPEC §11): all MIT — pi © 2025 Mario Zechner, OMP © 2025–2026 Can
Bölük, Prime Agent © 2026 Prime Intellect.

### Optional: refresh the upstream doc mirrors

Only the three upstream sections track external repos (see **Codebases**); the
Smart Connections embedding index is regenerable and git-ignored, so only the
markdown is durable. If those checkouts are present and you want newer docs:

```bash
cp -R ~/Work/github/oh-my-pi/docs ~/vault/personas/phi/oh-my-pi/docs
cp -R ~/Work/github/PrimeAgent/packages/coding-agent/docs ~/vault/personas/phi/prime-agent/docs
```

`oh-my-pi-agent/` has no external source to refresh from — this vault copy is its
home of record.
