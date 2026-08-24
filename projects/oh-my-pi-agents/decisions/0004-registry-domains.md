---
type: adr
created: 2026-08-24
tags: [adr, decision, vault, registry, note-system, isolation]
---

# ADR 0004 — Registry domains: split the vault into per-domain registries (supersedes 0002)

## Status

Accepted 2026-08-24. **Supersedes [[decisions/0002-note-system-layout.md|ADR 0002]]** for the
multi-domain case. ADR 0002's *within-a-registry* split (design corpus in the persona section vs
forward-dev notes in the project section, connected by wikilinks) remains in force **inside each
registry**; what changes is that there is now more than one registry.

## Context

We must keep **Capitec-specific knowledge isolated from the public OMA fork**. The OMA vault and its
registry are pushed to a public repo; Capitec knowledge must never leak there.

ADR 0002 deliberately kept **one** vault (to preserve the cross-section link graph and semantic
search) and SPEC resolved-item 1 said to split only under *size / attachments / sync* pressure. The
new pressure is **confidentiality**, a driver 0002 did not weigh — hence we supersede it rather than
bend it.

Three leak vectors motivate a hard split (not folder-exclusion + discipline):

1. **Embeddings** — one Smart Connections index per vault would embed both domains together.
2. **Cross-links** — wikilinks bleed concepts across the boundary via the link graph.
3. **Persona-memory contamination** — an entity that learns from Capitec context then writes a
   lesson into the public OMA space carries Capitec-shaped patterns even when it names nothing.

Grounding (read-only scout of `packages/coding-agent/src`):

- Vault MCP resolves **one** root per process: arg > `OMP_VAULT_PATH` > `~/vault`
  (`vault-mcp/vault.ts:68`). `vaultRoot` is already a parameter at spawn wiring
  (`entity/mcp-wiring.ts:93`) but is sourced from that single global.
- Semantic index is per-vault at `<vaultRoot>/.smart-env/` (`vault-mcp/store.ts:60`) → **separate
  vaults ⇒ separate indexes ⇒ embedding leak closed structurally**.
- `section` is only a **post-query filter** (`vault-mcp/vault.ts:210`), not a hard boundary — which
  is why section-scoping in one vault is insufficient.
- C1 record has `vaultSection: string`, **no** `vaultRoot` (`entity/schema.ts:88`).
- Memory banks live at `~/.omp/banks/<bank>/` (`memory-mcp/bank-store.ts:39`), **outside any git
  repo**, regardless of `--registry` (which drives only retention policy, `memory-mcp/server.ts:86`)
  → memory cannot leak through a committed repo.

## Terminology

- **Registry** — canonical term for a knowledge **domain**: a self-contained unit of `{ vault root +
  backing git repo + entity records + memory-bank namespace }`. `oma` (public) and `capitec`
  (private) to start; first-class and extensible to N.
- **Visibility** — a per-registry property, `public | private`. A **private** registry may *read*
  `public` registries; a `public` registry may neither read nor write a `private` one. Asymmetric
  and directional.
- **Access grant (`readableBy`)** — an explicit, named, one-directional read edge into a **private**
  registry, declared **by the target** (the data owner, Q10). Private registry `Y` naming `X` in
  `readableBy` lets `X`'s entities read `Y`. **The reader must itself be private** — a `public`
  registry can never be granted read into a `private` one (Q9), so private content never routes to a
  public repo. Grants are **non-transitive** (Q11): `X→Y` and `Y→Z` do not give `X→Z`.
- **Home registry** — the single registry an entity belongs to. An entity **writes** only its home
  and **reads** its home, any `public` registry, plus any `private` registry that named its home in
  `readableBy`. No entity is home to two registries (*hermetic*).

## Decision

Split the single vault into per-registry domains. Settled design (grill rounds Q1–Q12):

1. **Hermetic partition (Q1).** One entity ↔ one home registry. Capitec work is owned by distinct
   Capitec-home entities; Phi stays `oma`-home. Closes contamination vector #3 by construction — an
   `oma` entity physically cannot read Capitec notes, so it cannot carry Capitec patterns into a
   public note.
2. **Registry is first-class (Q2).** A `registry` is the canonical unit (a knowledge domain); not a
   one-off `oma`-vs-`capitec` special case. Modeled minimally — no governance layer.
3. **Per-registry symlinks (Q3).** `~/oma-registry` and `~/capitec-registry`; existing OMA stubs
   migrate off `~/vault`.
4. **Directional read (Q4).** `visibility` is a registry property: private reads public, never the
   reverse.
5. **Records partition too (Q5).** Each registry repo owns its **own entity records**; the loader
   scans all configured registry roots. A Capitec persona's definition (`systemPrompt`,
   `autoloadSkills`) is itself domain knowledge and lives only in the private repo.
6. **Manifest + registry-prefixed banks (Q6).** An explicit registry manifest resolves `id →
   { root, visibility }`; bank ids are registry-prefixed so the shared `~/.omp/banks/` store is
   self-partitioning.
7. **Path-jail enforcement (Q7).** The vault MCP hard-rejects any path resolving outside an allowed
   root (writable = home only; readable = home + public), on top of process isolation.
8. **Cross-registry read interface (Q8 = A+i).** Read tools take an explicit `registry?` param
   (default = home); `search_notes` defaults to home-only, foreign public reads are deliberate
   opt-in.
9. **Directed access into private registries (Q9–Q12).** Beyond private→public, a private registry
   may grant *named* private registries directional read access via a `readableBy` allowlist on the
   **target** (Q10, registry granularity). Hard guard: a `public` registry can never be a reader of a
   private registry, grant or not (Q9) — the reader must be private, so nothing private reaches a
   public repo. Grants are **non-transitive** (Q11). Entities do not see private registries they
   cannot read (Q12) — the visible set is home ∪ readable set.

### Change surface (to apply on implementation — NOT yet done; pending user sign-off)

**C1 — entity record (`entity/schema.ts`):** add `registry: string` (required; the entity's home
registry id, a key into the manifest). `vaultSection` stays relative to the home registry's vault
root. Loader (`registry/persisted-agents.ts`) resolves `registry` → root via the manifest and scans
every configured registry root for records.

**Registry manifest (new; WS7a):** `~/.omp/agent/registries.json` —
`{ [id]: { root: <path>, repo?: <url>, visibility: "public" | "private", readableBy?: <id[]> } }`.
`readableBy` (private entries only) names the private registries granted read access. Per-machine,
**not committed**. The loader **rejects** any `readableBy` grant whose named reader is `visibility:
"public"` (Q9 guard) and any `readableBy` on a `public` entry. Setup creates + validates it and the
`~/<id>-registry` symlinks.

**C3 — vault MCP (`vault-mcp/`):**
- **Multi-root:** the server launches with its home root (writable) plus its **readable set** —
  every `public` root and every `private` root that named the home registry in `readableBy`;
  `vaultRoot` sourced per-entity, not from the global resolver.
- **`registry?` param** on read tools: `search_notes(query, {section?, registry?})`,
  `get_note(path, {registry?})`, `get_connections(path, {registry?})`; default = home. A passed
  `registry` must be in the reader's readable set, else the call is rejected. `write_note` ignores
  `registry` — writes are always home.
- **Default search scope = home-only;** a foreign read (public or granted-private) is explicit
  (`registry: "<id>"`), loading that registry's `.smart-env` index on demand.
- **Path-jail:** every resolved path must be contained in an allowed root — writable = home only,
  readable = home + readable set — else reject.
- **Discoverability:** registry listings surface only the reader's home ∪ readable set (Q12).

**`entity/mcp-wiring.ts`:** source `vaultRoot` from the entity's `registry` (via manifest) instead
of the global; pass the readable set (public + granted-private roots); keep `--section`. Memory:
prefix the bank id with the registry (`oma/<persona>`, `capitec/<persona>`).

**WS6 — project pointers:** stubs resolve through the per-registry symlink —
`@~/capitec-registry/projects/<project>/<persona>.md` (public: `@~/oma-registry/...`). Existing OMA
stubs are rewritten `@~/vault/...` → `@~/oma-registry/...`.

**Rename / migration (one-time):**
- Rename the existing vault checkout → **`oma-registry`**; create the `~/oma-registry` symlink; drop
  `~/vault`; rewrite every project stub `@~/vault/...` → `@~/oma-registry/...`.
- Provision **`capitec-registry`** as a private repo with the same section skeleton
  (`agents/ personas/ projects/ reference/`, `.smart-env/` gitignored).
- Move any Capitec-home entity records + notes into `capitec-registry`.

**Docs of record (revise when the code lands):** SPEC §4.3 (vault layout), §7 (vault + MCP bridge),
§10 resolved-item-1 (vault placement); CONTRACTS C1 (add `registry`), C3 (multi-root + `registry?`
+ path-jail). These describe how OMA *functions* and are updated at implementation, not now.

## Consequences

- The confidentiality boundary is **structural**, not disciplinary: separate embedding indexes,
  separate repos, hermetic entities, code-enforced path-jail. Capitec knowledge cannot reach the
  public repo via embeddings, links, entity records, or memory banks.
- Cost: revising two **frozen** contracts (C1, C3), a multi-root vault MCP, the manifest + setup
  work, and a one-time rename/stub migration.
- Reversibility is low (frozen-contract changes) — the reason this is an ADR.
- **Invariant preserved under grants:** because a `public` registry can never read a `private` one
  (Q9), no `readableBy` grant can route private content into a public repo. Grants only widen reads
  *among private registries*.
- **Accepted bleed:** a grant `A→B` (both private) deliberately accepts bounded, one-directional
  concept-bleed — an `A`-home entity reading `B` may write `B`-shaped lessons into `A`'s own bank or
  vault. That is the grant's intent; the reverse stays blocked unless separately granted. No public
  exposure either way.
- Residual risks: (a) a user who points `MNEMOPI_DATA_DIR` into a repo defeats memory isolation —
  setup should warn; (b) cross-registry wikilinks are forbidden by convention but not yet
  code-enforced — links must stay within a root.
- **Capitec contents deferred by design:** the `capitec` registry's purpose and entity roster are
  intentionally open-ended now and refined over time. A follow-up will be created **inside
  `capitec-registry`** once it is provisioned (it does not exist yet). Tracked in [[phi.md]].

## Links

- [[phi.md]] — project note · [[decisions/0002-note-system-layout.md|ADR 0002]] (superseded)
- Design corpus to revise on implementation:
  [[personas/phi/oh-my-pi-agent/SPEC.md]] · [[personas/phi/oh-my-pi-agent/CONTRACTS.md]]
