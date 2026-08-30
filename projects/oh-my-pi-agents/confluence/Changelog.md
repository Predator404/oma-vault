---
type: confluence-page
page: Changelog
project: oh-my-pi-agents
maintained-by: phi
status: draft — for review
updated: 2026-08-30
---

# OMA — Changelog

> Confluence-bound page. Dated from the `oma` branch commit and PR history
> ([Predator404/oh-my-pi-agent](https://github.com/Predator404/oh-my-pi-agent)).
> Appended to as work lands.

## Two version tracks

OMA carries **two independent version numbers**:

- **`omp` (upstream-synced)** — the OMP release OMA is rebased onto. It moves
  **only on an upstream rebase**, not on OMA's own work. Current base:
  **`omp 18.0.11`**. Pure OMP-core fixes that we contribute back upstream (e.g.
  the mnemopi and compaction fixes below) belong to this track and ride the OMP
  changelog, not the agent one.
- **`oma-agent`** — SemVer keyed to **agent-portion PR merges**: a feature bumps
  the minor, a fix bumps the patch. This is the number that describes the
  persistent-agent build itself. Current: **`oma-agent 0.8.0`**.

Every entry below is tagged `oma-agent X.Y.Z · on omp A.B.C`.

## Legend

- 🚀 feature · 🐞 fix · 🔧 internal/process
- Upstream rebases are called out as distinct blocks:

> ### 🔄 UPSTREAM REBASE
> Marks a point where OMA was rebased onto a newer OMP release. Links go to the
> OMP changelog for the pulled-in range.

---

## [Unreleased]

_Nothing pending._

## 2026-08-30 — `oma-agent 0.8.0 · on omp 18.0.11`

- 🔧 **`rebaseForkOnUpstream` skill.** New OH-MY-PI skill encapsulating the
  fork-rebase procedure this work uses: map remotes, find the true fork point
  (merge-base, not `origin/main`), fold in WIP and unpulled fork commits, replay
  onto `upstream/main` with a fork-features-win / upstream-renames-supersede
  conflict policy, verify, and `--force-with-lease` push. Generalized to any
  fork with downstream changes. (`.omp/skills/rebaseForkOnUpstream`)
- 🔧 **`omaChangelogUpdate` skill.** Encapsulates the phi OMA-changelog flow:
  OMA's own work gets its own `oma-agent`-versioned dated entry, upstream
  rebases stay in separate block quotes, the `omp` base is read from the
  fork's rebase target, and the vault commit/push is done alone (never the
  repo CHANGELOG). (`.omp/skills/omaChangelogUpdate`)
- 🐞 **Dev launchers resolve Bun off PATH.** The `omp`/`oma` launchers now find
  Bun in `~/.bun/bin` or `$BUN_INSTALL/bin` when it is not on PATH, fixing
  `bun: not found` from GUI/cron/non-interactive contexts. (`02052e8a`)
- 🔧 **CHANGELOG reconciled after rebase.** Removed the released
  `[18.0.4]–[18.0.1]` sections the rebase had duplicated from upstream and
  moved the OMA-specific entries into the coding-agent `[Unreleased]`, so the
  released sections now match upstream. (`3abcdda0`)

> ### 🔄 UPSTREAM REBASE — `omp 18.0.4` → `omp 18.0.11` (2026-08-30)
>
> OMA retargeted from its previously recorded base (`omp 18.0.4`) onto current
> upstream `main` (**OMP 18.0.11**), resolving roughly **~1479 commits** of
> upstream drift. The fork's own commits were replayed onto the new base using
> the `rebaseForkOnUpstream` conflict policy (fork features win; upstream
> renames supersede). The OMA-side work that came out of this rebase is tracked
> in its own `oma-agent 0.8.0` entry above, kept separate from this upstream
> pull.
>
> **OMP changelog for this range:**
> - Release notes: [omp v18.0.11](https://github.com/can1357/oh-my-pi/releases/tag/v18.0.11)
> - Full diff: [v18.0.4...v18.0.11](https://github.com/can1357/oh-my-pi/compare/v18.0.4...v18.0.11)
> - Changelog file at the tag: [CHANGELOG.md @ v18.0.11](https://github.com/can1357/oh-my-pi/blob/v18.0.11/packages/coding-agent/CHANGELOG.md)
## 2026-08-24 — `oma-agent 0.7.0 · on omp 18.0.0`

- 🚀 **Per-domain vault registry isolation.** Split the vault into per-domain
  registries (`oma` public, `capitec` private) with hermetic entities,
  directional read rules (private→public plus named private→private grants), a
  registry manifest, and path-jailed multi-root vault access. Hard
  confidentiality isolation for the regulated environment. (ADR 0004,
  `ea0a65be`)

## 2026-08-23 — `oma-agent 0.6.1 · on omp 18.0.0`

- 🐞 **Entity `thinkingLevel` carried on attach**, and the `@@` entity picker is
  gated to message start (a mid-message `@@` falls through to the file picker),
  matching the address parser.
  ([PR #7](https://github.com/Predator404/oh-my-pi-agent/pull/7), `233db0b7`)

## 2026-08-22 — `oma-agent 0.6.0 · on omp 18.0.0`

- 🚀 **`@@` entity picker + attach-on-address.** A single `@` stays the file
  picker; `@@` opens an entity picker over the OMA registry and inserts
  `@@<name>: `; `@@@` escapes back to the file picker. Addressing an entity
  attaches it as an advisor (persona body → advisor instructions) and starts the
  advisor subsystem if it was off.
  ([PR #6](https://github.com/Predator404/oh-my-pi-agent/pull/6), `26774b63`)

---

> ### 🔄 UPSTREAM REBASE — `omp 17.3.7` → `omp 18.0.0` (2026-08-21 → 08-22)
>
> The fork was retargeted from its original base onto **OMP 18.0.0**, resolving
> roughly **498 commits** of upstream drift with zero conflicts and crossing the
> 18.0 major version bump. The pre-rebase OMA commits were replayed onto the new
> base; the agent features dated 2026-08-22 below landed directly on 18.0.0.
>
> What the rebase pulled in from upstream OMP (highlights): the `omp render`
> session-replay command, a live `omp bench` dashboard, `/shake thinking`,
> richer edit-tool input formats, in-place transcript-tail rewinds, and a large
> batch of streaming/render/provider fixes.
>
> **OMP changelog for this range:**
> - Release notes: [omp v18.0.0](https://github.com/can1357/oh-my-pi/releases/tag/v18.0.0)
> - Full diff: [v17.3.7...v18.0.0](https://github.com/can1357/oh-my-pi/compare/v17.3.7...v18.0.0)
> - Changelog file at the tag: [CHANGELOG.md @ v18.0.0](https://github.com/can1357/oh-my-pi/blob/v18.0.0/packages/coding-agent/CHANGELOG.md)

---

## 2026-08-22 — `oma-agent 0.5.1 · on omp 18.0.0`

- 🐞 **Direct-address answers always surface.** A `@@<name>:` reply emitted with
  no severity used to strand on the idle `aside` queue until the next prompt
  ("Phi will answer but nothing happens"). Direct-address replies now route to a
  visible channel regardless of severity. (`719bede6`)

## 2026-08-22 — `oma-agent 0.5.0 · on omp 18.0.0`

- 🚀 **Kernel state survives compaction.** After each auto-compaction, a live
  Python-kernel state manifest re-injects the kernel's variable names and types,
  so working state is not lost across a compaction pass. (`69cc939a`)

## 2026-08-22 — `oma-agent 0.4.0 · on omp 18.0.0`

- 🚀 **Non-interactive retrieval + daemon lifecycle CLI.** Read an entity's reply
  without the TUI, a fast-fail on an already-active session lease (instead of a
  30s daemon timeout), and `oma entity daemon status | shutdown | restart`.
  (`a54a7b96`)

## 2026-08-22 — `oma-agent 0.3.1 · on omp 18.0.0`

- 🐞 **Wake detached idle workers.** A one-shot `oma entity prompt` / steer /
  follow-up to a detached, idle resident worker now wakes it to run a turn
  instead of enqueuing silently. (`0357d573`)

## 2026-08-22 — `oma-agent 0.3.0 · on omp 18.0.0`

- 🚀 **Per-entity icon & colour** across the advisor card, the Agent Hub roster,
  and the daemon-wire `AgentSnapshot`, with an ascii fallback. (`e39283e5`)

> **OMP-core fixes contributed upstream this day** (tracked on the `omp` track,
> not `oma-agent` — they are OMP fixes, merged to OMA and filed upstream):
> - 🐞 Blank/whitespace `mnemopi.dbPath` resolves to persistent storage instead
>   of a volatile in-memory bank.
>   ([PR #4](https://github.com/Predator404/oh-my-pi-agent/pull/4) · upstream
>   [#9360](https://github.com/can1357/oh-my-pi/issues/9360), `eb836252`)
> - 🐞 Compaction preserves the post-snapshot suffix on speculative apply.
>   ([PR #5](https://github.com/Predator404/oh-my-pi-agent/pull/5) · upstream
>   [#9351](https://github.com/can1357/oh-my-pi/issues/9351), `f12a7b4b`)

## 2026-08-21 — `oma-agent 0.2.1 · on omp 17.3.7`

- 🐞 **Resident workers surface their own MCP tools.** A resident entity now
  registers its memory (`recall`/`retain`/`forget`) and vault tools onto its own
  session, which previously only happened for the owner path. (`74e5012a`)

## 2026-08-19 — `oma-agent 0.2.0 · on omp 17.3.7`

- 🚀 **`@@<name>:` direct addressing** of an in-session advisor entity, so a
  message can be aimed at a specific entity.
  ([PR #2](https://github.com/Predator404/oh-my-pi-agent/pull/2), `8828df2b`;
  prompts moved to `.md` templates in `8cb0227f`)

## 2026-08-19 — `oma-agent 0.1.1 · on omp 17.3.7`

- 🐞 **Entities survive terminal close.** Fixed the broker reaping resident
  workers on client disconnect, and the `oma entity` CLI never closing its
  daemon socket (the "60–120s cold-start hang"). Spawn now defaults the worker
  cwd to the caller's cwd.
  ([PR #1](https://github.com/Predator404/oh-my-pi-agent/pull/1), `3a22e862`)

## 2026-08-18 — `oma-agent 0.1.0 · on omp 17.3.7`

- 🚀 **Initial persistent multi-entity agent build.** The daemon subsystem
  (supervisor + resident workers + scheduler + protocol) ported from Prime Agent
  into OMP, entity registry and role model, bank-scoped memory MCP, vault + MCP
  bridge, project pointers, and the CLI/TUI surface. OMA identity (`oma`/`0.1.0`,
  `~/.oma` config root, `πA` status marker). (`cb03ed38`, docs `4a8cd4ca`)

---

> ### 🔄 UPSTREAM BASE — `omp 17.3.7` (2026-08-17)
>
> The project's original base. OMA v0.1.0 was built on **OMP 17.3.7**.
> - Release notes: [omp v17.3.7](https://github.com/can1357/oh-my-pi/releases/tag/v17.3.7)

## Links

- Companion pages: [[Overview]] · [[Roadmap]]
- Open follow-ups per change: [[follow-ups]]
