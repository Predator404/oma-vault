---
type: confluence-page
page: Roadmap
project: oh-my-pi-agents
maintained-by: phi
status: draft — for review
updated: 2026-08-24
---

# OMA — Roadmap

> Confluence-bound page. "Shipped" is drawn from merged work on the `oma` branch;
> "Ideas" from the open items in [[follow-ups]]. internal refactors, test-coverage gaps, and
> housekeeping are omitted.

## How to read this

- **✅ Shipped** — merged and running live.
- **🔜 Next** — decided, in flight, or the obvious near-term step.
- **💡 Idea** — proposed in the follow-up register, not yet committed.
- **🧊 Deferred** — deliberately parked, non-blocking.

---

## ✅ Shipped

**Persistence & runtime**
- Persistent multi-entity daemon: entities survive terminal close, detach and
  reattach with state intact.
- Resident workers wake on a one-shot `oma entity prompt` / steer / follow-up
  even when detached and idle.
- Config-root isolation: `oma` runs entirely under `~/.oma`, side by side with a
  stock `omp` install without colliding on the broker socket.
- Daemon lifecycle CLI: `oma entity daemon status | shutdown | restart`.
- Fast-fail on an already-active session lease instead of a 30s daemon timeout.

**Entities & memory**
- Two role policies (agent = episodic memory, persona = curated lessons), each
  with its own model, tool grants, memory bank, and vault section.
- Bank-scoped memory MCP so an entity addresses its own memory (`recall` /
  `retain` / `forget`) — including resident entities surfacing their own memory
  and vault tools onto their session.
- Per-domain registry isolation (`oma` public, `capitec` private): hermetic
  entities, directional read rules, path-jailed vault access.

**Interaction**
- `@@<name>:` direct addressing of an in-session advisor entity.
- `@@` entity picker in the composer with attach-on-address (`@@@` escapes back
  to the file picker); entity `thinkingLevel` carried through on attach.
- Direct-address answers surface reliably even when the reply carries no severity.
- Per-entity icon and colour across the advisor card, Agent Hub roster, and the
  daemon-wire snapshot.

**Knowledge vault**
- Shared human-browsable Markdown vault with per-entity and per-project sections.
- Hybrid retrieval: semantic (vector) search plus the Obsidian wikilink graph,
  local and privacy-preserving.
- Project-pointer mechanism: a repo carries a thin `@~/vault/...` stub, content
  lives once in the vault, resolved at read time through the `~/vault` symlink.

**State continuity**
- Live Python-kernel state manifest re-injected after auto-compaction, so kernel
  variable names/types survive a compaction pass.

**Identity / brand**
- `oma`/`0.1.0` identity: `--version` / `--help` render OMA, warm faded logo,
  `πA` status-line marker to tell an OMA instance apart from stock `omp`.

## 🔜 Next

- Moving UI into a T3-code fork to allow multiple workflows simultaneously. This is already implemented but not yet ready for general use.
- Using sandcastle to transparently handle agent sandboxing - creating worktrees and isolating agents automatically as they are spun up - so paralell development streams can run autonomously without risking the host machine. 
- Ability to dictate instead of type into the interface
- **Sticky default entity** — promote an entity to the session's default
  interlocutor for the rest of the session (no per-message `@@` prefix) until
  reverted, with a visible TUI indicator. The across-turns counterpart to
  today's per-message addressing.
- **`oma last`** — resume the most recent session instead of starting fresh, with
  a way to pick the primary entity (`oma last --agent <name>`).
- **Incremental advisor attach** — attach one entity without tearing down and
  reseeding the whole advisor roster (today's attach drops peers' accumulated
  context).
- **Self-updating model alias fix** — a supported broker-restart path so entities
  can use the self-updating `anthropic/opus` alias instead of a pinned concrete
  id.
## 💡 Ideas (proposed, not committed)

- **Cross-session peer bridge** — register a resident entity as an addressable
  peer in another session's Agent Hub, so `hub send to=<entity>` works across
  project boundaries. Needs a deliberate architectural decision.
- **Worker crash-recovery** — retry backoff, replacement-supervisor election via
  lease, and generation fencing of adopted workers (porting Prime Agent's
  hardening).
- **Refinement / rollback governance** — versioned snapshot+rollback over an
  entity's own config (system prompt, memories, skills, subagent specs). Lean on
  git first; the genuinely missing pieces are memory-bank snapshot/rollback and
  an in-runtime checkpoint/rollback verb.
- **Smarter compaction** — a summarizer that preserves the active plan/goal/todo
  verbatim so working state need not be offloaded.
- **Agents-view TUI** — a live roster browser (model/effort/status, sort by last
  message, search).
- **Per-entity speech bubble in the primary transcript** — extend the icon/colour
  identity into the main conversation view.
- **`@@<name>` without a colon as a notation** — shorthand tied to the sticky
  default-entity switch.
- **Interactive `/heartbeat` surface** — today heartbeats are CLI-flags only.

## 🧊 Deferred (non-blocking)

- **Local-hardware entity placement** — no entity pinned to a local model yet.
  Recorded heuristic: narrow high-volume/privacy-sensitive personas run local,
  the complex-reasoning coordinator stays cloud. Workstation is Apple M5, so
  local inference is viable when this reopens.

## Known constraints (worth stating)

- **Semantic search depends on a local embedding model** that a corporate proxy
  currently blocks (HuggingFace). Falls back to `all-MiniLM-L6-v2`; until the
  proxy is resolved, in-Obsidian re-indexing is limited and the headless
  index-build script is the maintenance path.

## Links

- Full open-item detail: [[follow-ups]]
- Companion pages: [[Overview]] · [[Changelog]]
