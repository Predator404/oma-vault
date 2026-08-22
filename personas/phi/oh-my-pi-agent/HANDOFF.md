---
status: IMPLEMENTED + committed on branch `oma` (fork Predator404/oh-my-pi-agent; `main` reserved for upstream). Daemon workflow debugged live 2026-08-19 — memory/broker/vault fixes done+verified, Phi live & persistent, fix PR #1 open. See CURRENT STATE → "LATEST — 2026-08-19". Outstanding: prompt-wake gap; fork PAT lacks PR-write.
created: 2026-08-17
topic: layered persistent-agent/persona memory architecture on OMP
---

# Handoff: Persistent Agent / Persona Memory Architecture

## CURRENT STATE (read first — resume here)

Everything below in this doc is the design/decision trail. This block is where things actually stand.

### LATEST — 2026-08-19: daemon workflow made usable; Phi live; fix PR open

**Supersedes stale facts in the older entries below.** The feature is no longer "uncommitted on main": it is committed on branch **`oma`** (`origin=Predator404/oh-my-pi-agent`, `upstream=can1357/oh-my-pi`; `main` reserved for upstream). Companion docs now live at `~/Work/ClaudeEnvSetup/persistentAgents/` (`SPEC.md`, `CONTRACTS.md`, this file). The entity registry + vault are a separate checkout at `~/oma-registry` (symlinks `~/.oma/agent/registry → ~/oma-registry`, `~/vault → ~/oma-registry/vault`). Config root is `~/.oma`.

**First real end-to-end run of Phi surfaced three blocking defects; all fixed + independently verified this session (dev+tester subagents, then cross-seam review):**

1. **Memory MCP wouldn't load.** The project `.omp/mcp.json` pointed `OMP_ENTITY_REGISTRY` at a moved/nonexistent path (`~/Work/persistentAgents/registry`) and hardcoded the `omp-memory-mcp` bin (absent in a source checkout → `MCP tool load failed`). Fixed to the dev-checkout shape (`bun run …/memory-mcp/server.ts --registry ~/oma-registry --bank phi`, `OMP_ENTITY_REGISTRY=~/oma-registry`). Local project config, NOT oh-my-pi source. Verified: memory tools load; hermetic retain→recall round-trip green.
2. **Entities didn't survive terminal close (violated SPEC §1.1).** Two bugs: (a) the broker's idle-shutdown reaped resident agent workers once the launching client disconnected; (b) the `oma entity` CLI never closed its daemon socket, pinning Bun's event loop — this WAS the "60–120s cold start"/hang, not a slow broker. Fixed in oh-my-pi source: `broker.ts` re-arms idle-shutdown while `AgentSupervisor.hasResidentSessions()` is true; `entity-cli.ts` connect→try/finally→close + new `AgentDaemonClient.close()`; `spawn` now defaults the worker cwd to the caller's `process.cwd()`. Verified LIVE: `oma entity spawn phi` exits 1.3s, detached broker runs at ppid=1, `oma entity ps` returns 0.2s and shows the worker still `ready` after the spawn client exited. Tests: new `broker-agent-persistence.test.ts` + `detach-reattach` + `entity-cli` = 28 pass / 0 fail.
3. **Vault semantic search returned nothing.** No Smart Connections index existed. Built `.smart-env` offline (214 sources / 6109 blocks). NB: HuggingFace is blocked here by a corporate proxy, so the index + query embedder use `sentence-transformers/all-MiniLM-L6-v2` (384-dim, in the reader's `KNOWN_MODEL_DIMS`) instead of the spec's `TaylorAI/bge-micro-v2` — index and query share the one model, so cosine ranking is coherent. Verified offline via the built `vault-mcp` server: block-level hits (0.79 "Daemon supervisor", 0.60 SPEC §8.1), and a hard error instead of any network call when the local model is removed. Tooling at `persistentAgents/vault-index/{build-index.ts,verify.ts,reseed-model.sh,README.md}`; `.smart-env/` is gitignored.

**Delivery.** The two oh-my-pi source fixes (broker persistence + CLI) are in **PR #1** on the fork — base `oma` ← `fix/entity-daemon-persistence-cli`, OPEN, awaiting user review+merge (user chose no auto-merge): https://github.com/Predator404/oh-my-pi-agent/pull/1 . The memory `.omp/mcp.json` fix and the vault index are local (non-source), applied directly. `gh pr create` was blocked (fork PAT lacks "Pull requests: write"), so the PR was created manually in the browser.

**Phi is live and persistent** (role persona, `anthropic/claude-opus-4-8`,
surviving across client exits). Per-spawn session ids are ephemeral — never
hardcode one; read the current id from `oma entity ps` (or `oma entity spawn
phi` if none is resident), then `oma entity attach <id>`. Her advisor form also
runs in-session (`~/.oma/agent/config.yml` `advisor.enabled: true`).

**Outstanding work items (open):** moved to the project register —
`~/vault/projects/oh-my-pi-agents/follow-ups.md` → "Project open items (not tied
to a single PR)". Deep threads: `investigations/prompt-wake-gap.md`,
`investigations/attach-retrieval-ergonomics.md`. Per ADR 0003, the register is
the durable home for open follow-ups; this doc keeps only the design/decision
trail.


**What exists, where:** the full feature is landed in `~/Work/github/oh-my-pi/packages/coding-agent` (the *merged* codebase = stock OMP fork + our agents feature). Reference source for the port: `~/Work/github/PrimeAgent` (MIT). Companion docs live in `~/Work/persistentAgents/`: `SPEC.md`, `CONTRACTS.md` (frozen C1–C5), and this file.

**Git state:** all work is UNCOMMITTED on branch `main` (atop upstream commit `0a912cc46` "bump version 17.3.7"). Nothing committed, no branch cut, no PR. The `oma` launcher runs the working tree live from source.

**Build/verify status:**
- TS gates: `fmt`/`lint`/`check`/`test` all green. 3 pre-existing residuals (zsh-leak in bash-executor test, missing csharp-ls plugin cache, changelog VERSION-vs-CHANGELOG drift) — verified on pristine main, not ours.
- Rust gates: `check:rs` green; `test:rs` = 2202/2204 pass. 2 failures are `pi-builtins touch::tests` atime tests — filesystem/env-dependent (macOS/APFS atime), feature touched zero Rust.
- Feature suite: 193 targeted + e2e tests pass, 0 tsc errors. Opus integration review verdict: READY.

**Toolchain installed this machine (was absent):** rustup + pinned `nightly-2026-07-28` (via `~/.cargo/env`), `cmake` (brew, for native dep audiopus_sys), `cargo-nextest` (prebuilt). Needed to run Rust gates.

**`oma` — the agents build on PATH:** `/opt/homebrew/bin/oma` (executable) runs this checkout via `bun` from source, preserving caller cwd. Entry is `packages/coding-agent/src/oma.ts` (a thin bin shim), NOT `src/cli.ts` directly — see the isolation entry below. `omp` (brew 17.3.5) is untouched = stock, side by side. `bin` entries in `packages/coding-agent/package.json`: `omp`→`src/cli.ts`, `oma`→`src/oma.ts`. First launch slower (bun transpiles); `bun run build` would make a compiled binary later.

**Config-root isolation (2026-08-18) — `oma` no longer collides with stock `omp`:** the daemon broker is a per-project singleton whose runtime dir is `<configRoot>/run/daemons/<wyhash(projectDir)>` and whose socket (`broker.sock`) is keyed on that path. Both builds resolved the same `~/.omp` configRoot, so in any project the first launched build owned the broker and the other adopted it; this build's *additive* daemon protocol (new `agent` op + `agent-event` in `launch/protocol.ts`) then broke whichever side did not spawn the broker (root cause of the VS Code `byteforce.omp-vscode-bridge` breakage). Fix: `src/oma.ts` shim sets `process.env.PI_CONFIG_DIR ??= ".oma"` before importing `cli.ts`, so `oma` lives entirely under `~/.oma` (own broker socket, sessions, mnemopi banks, auth) while stock `omp` stays under `~/.omp`. Set in the shim (not the launcher) so the dev build and a future compiled `oma` binary behave identically. `cli.ts` entry detection changed from `import.meta.main || PI_COMPILED` to `moduleIsProcessEntry()` (basename-match against `Bun.main`) so a sibling shim importing `cli.ts` neither double-runs, fails to run, nor loses the worker host; `runCli(argv, { isProcessEntry })` gains the override the shim passes. Files: `src/oma.ts` (new), `src/cli.ts`, `packages/coding-agent/package.json` (bin), `/opt/homebrew/bin/oma` (launcher → `src/oma.ts`). Verified: `oma --version` exit 0; `getDaemonRuntimeDir` resolves `~/.oma/...` vs `~/.omp/...`; live `oma mcp list` created `~/.oma/{agent,run,logs}` and left `~/.omp/run/daemons` untouched. Dead broker residue under `~/.omp/run/daemons` pruned; the live stock broker (pid 61121, this project, hosting the current session) preserved. NOTE for a compiled `oma`: point `scripts/build-binary.ts` at `src/oma.ts` (currently compiles only `src/cli.ts`→`omp`); the shim already makes such a binary isolate correctly.

**OMA identity — `oma`/`0.1.0` (2026-08-18):** `--version`/`--help`/usage now render `oma` and its own version, not `omp/17.3.7`. `src/oma-identity.ts` (new) holds `OMA_APP_NAME="oma"` + `OMA_VERSION="0.1.0"` (independent of upstream omp numbering; bump per OMA release). Threaded via `runCli(argv, { isProcessEntry, appName, version })` — the `oma.ts` shim passes them; stock/dev `omp` (no options) still renders `omp`/`VERSION`. Verified: `oma --version`→`oma/0.1.0`, `oma --help`→`oma v0.1.0`, `bun src/cli.ts --version`→`omp/17.3.7`. Not changed: internal `setProcessName(APP_NAME)` stays `omp` (herdr foreground-name validation); welcome-screen title still `omp v… · agents` (rebrand item below).

**Status-line brand marker — `π` → `πA` under OMA (2026-08-18):** the fixed bottom-left brand segment (`status-line/segments.ts` `piSegment`, `"pi"` — first `leftSegment` in every preset) now renders `πA` for OMA vs `π` for stock `omp`, so two instances running side by side are tellable apart from one always-present spot. Marker is `process.env.OMA_BUILD="1"`, set by the `oma.ts` shim (inherited by workers), read via `isOmaBuild()`/`OMA_BUILD_ENV` in `src/oma-identity.ts`. Falls back to a literal `π`/`πA` if the active theme hides `icon.pi`. No Unicode π+A ligature exists, so the `πA` composite is used (user-approved). Verified by rendering `piSegment`: OMA→`"πA "`, omp→`"π "`. Live TTY visual not captured (interactive-only surface); render smoke stands in.

**UI rebrand (agents build identity):** `modes/components/welcome.ts` — warm faded logo palette (brick-red→orange→amber→gold→straw; replaced pink/purple/blue) in `GRADIENT_STOPS` + `GRADIENT_RAMP_256` (single shared source → recolors welcome screen AND all setup-splash scenes); title `omp v… · agents`; `persistent agents` subtitle under logo; right-column note "Persistent Agents build / Fork of OMP · long-running agents" + pointers `oma entity setup`, `oma entity spawn <name>`. `modes/setup-wizard/scenes/splash.ts` — matching `persistent agents` subtitle. Visual confirm needs a real TTY (`oma` interactive); headless render can't (theme singleton is runtime-only).

**Known non-blocking follow-ups (not done):** (1) DONE — `oma --version`/`--help` now print `oma/0.1.0` (see OMA identity entry above); welcome-screen logo title still shows `omp v… · agents` if fuller rebrand wanted. (2) full monorepo suite/lint already run green; (3) commit + branch + PR not started — user wants a personal run-in period first. (4) live-model end-to-end not run (model boundary faked per SPEC §12.9). (5) changelog/version-bump is a PR-time hygiene item.

**To resume the actual project work:** the build is done and green; next real decisions are user-driven (run-in period, then commit/PR). The buildable plan + acceptance is in `SPEC.md`; frozen seams in `CONTRACTS.md`.


## Context

Started from: "does OMP have a persistent-agent setup like Prime Agent?" Answer: no native always-on daemon. From there, conversation designed a 3-tier memory architecture (Agent → Persona → Project) mapped onto existing OMP primitives. This doc is a findings inventory, not a spec — nothing built yet.

## Terminology

**agent** = generalist/coordinator role: broad remit, full cross-session episodic memory, own state store. A *role class*, not a single instance — the roster can hold multiple named agents. **persona** = specialist role: narrow domain, curated lessons only. Plain terms, no invented spelling — decided over the earlier "aGent" affectation (settled this session; both terms get precise operational definitions in the formal spec, not redefined here).

**Process note:** naming/definition decisions and other standing conclusions get written to this file as they're made, without being asked each time — this file is the durable record, not the chat.

## Core idea (revised — see "Pivot" below)

Not a fixed 1-generalist + N-specialists hierarchy anymore. A **flat roster of long-running named entities**, each individually named, modeled, hosted, and memory-scoped, tagged with one of two role-policies:

1. **agent** role — generalist/coordinator policy: broad remit, full cross-session episodic memory, can delegate. Multiple agents can exist (Prime's insight); "agent" is a role class, not one singleton instance.
2. **Persona** role — specialist policy: narrow domain, cross-session but curated *domain lessons only* (no raw episodic noise).

Both roles share a third, project-scoped layer: a lightweight in-repo pointer (not a copy) into the shared vault section that entity owns for that project.

Coordination between entities (either role, in any combination) happens over a messaging layer, not a fixed org chart — same-role and cross-role entities can work independently or hand off, symmetrically.

## Pivot from single-helper to roster (this session)

Original framing: one agent (singular helper, does everything) hands off to N personas when a task needs domain depth. Reconsidered after reviewing Prime Agent's multi-long-running-agent model — one fixed generalist is an artificial constraint, not a requirement. Resolution: keep the two *role policies* (agent=broad/episodic, persona=narrow/curated) but stop assuming exactly one agent instance. Roster can hold N agents and M personas; any entity can be addressed directly or coordinate with others, symmetric to task/user need. This matches the industry-standard 2026 multi-agent pattern (below), not a novel risk.


## Findings: what OMP already has (no build needed)

- **No native cron/daemon-triggered agent.** Sessions are process-lifetime only. Closest: `hub start ... persist:true/detached:true` keeps arbitrary *processes* alive across omp exits, but that's process supervision, not agent turns. Would need external cron + `omp -p` (print mode) to fake it.
- **agent tier → `memory.backend: mnemopi`.** Already vector-based (BAAI/bge-base-en-v1.5 embeddings, local SQLite, `BankManager` for multiple banks). `mnemopi.scoping: global` gives one shared cross-project bank. Tools: `recall`, `retain`, `reflect`, `memory_edit`. This *is* the "own vector memory store across sessions" — Obsidian not required unless human-browsable notes specifically wanted.
- **Persona identity → custom agent definition.** `~/.omp/agent/agents/<persona>.md`, frontmatter `name`/`description`/`model`/`systemPrompt` body = persona voice. Dispatched via `task` tool by name, from any project (`omp://task-agent-discovery.md`).
- **Persona domain lessons → `learn` tool's `skill` param.** Writes a *managed skill* at `~/.omp/agent/managed-skills/<persona>/SKILL.md` — user-level, keyed by name not cwd, persists across every project. Deliberate calls only (curated), unlike mnemopi's automatic per-turn retain (noisy/episodic). Pair with agent-def `autoloadSkills: [<persona>-lessons]` so it's always in context.
- **Project layer → project skill directory.** `.omp/skills/<persona>-project/SKILL.md`, committed to repo. Native provider auto-discovers one-level-under-`skills/` per project — no manual read needed, no reliance on model choosing to look for it.
- **Watchdog/advisor persona variant.** `WATCHDOG.yml` roster entry can give a persona a standing passive-reviewer role with its own model/instructions, persisted per-project file (`omp://advisor-watchdog.md`).
- **Full per-persona isolation (heavier option).** OMP *profiles* (`omp --profile <name>`) isolate an entire user-level agent dir: own `mnemopi.db`, own `mcp.json`, own agent defs. Clean separation but = separate process/session per persona, not N personas live inside one session.

## Real gap identified (needs custom build if pursued)

`recall`/`retain` tools bind to **one `memory.backend` per session**. No native way for "agent uses global bank, persona-A subagent uses domain-bank-A, persona-B uses domain-bank-B" *simultaneously inside one live session*. Two paths, not mutually exclusive:

1. **Profiles** (above) — zero build, but separate processes.
2. **Custom MCP server** wrapping `@oh-my-pi/pi-mnemopi`'s `BankManager` directly, exposing bank-scoped `recall(bank, query)` / `retain(bank, memory)` tools. Each persona subagent passes its own bank name. Registered via `.omp/mcp.json` (`omp://mcp-config.md`). This is the same build needed for real Obsidian vector search (below) — one server could cover both.

## Vault design (refined with tool research)

**The vector-vs-vault "overlap" question, answered:** not overlap, different jobs, industry-standard pattern is to run both together. 2026 memory-architecture consensus (Atlan, Mem0, Cognee writeups): vector search answers "what's similar to this" (fast, unstructured, good for episodic/semantic recall); a graph answers "what's related to this and how" (deterministic, multi-hop, explainable). Production systems embed the query to find entry nodes, then traverse the graph from there for relational context. Obsidian's wikilinks/backlinks graph *is* a lightweight knowledge graph already — no separate graph DB needed. Adding vector search over the same vault gets the hybrid pattern for free, on top of infrastructure you'd want anyway (human-browsable, git-diffable notes).

**Don't build the embedding server from scratch — mature tooling exists:**
- **Smart Connections** (Obsidian community plugin) computes embeddings locally, stores them locally, no cloud calls. Several MCP wrappers exist exposing `search_notes` (semantic, block-level, matches individual sections not just whole files), `get_similar_notes`, `get_connection_graph` (walks similarity links — the hybrid vector+graph pattern above, already built), `get_note_content`. Read-only, privacy-preserving, minimal setup. Candidates: `msdanyg/smart-connections-mcp`, `Rudelius/smart-connections-mcp`.
- **Obsidian Local REST API** (+ optional "Second Brain MCP Extension") gives full CRUD including binary files, surgical section/heading patching, full-text + JsonLogic structured queries, plus semantic ranking and graph exploration via the extension. Two-piece setup (plugin + MCP server) — more capable, more moving parts.
- Recommendation: start with Smart Connections MCP for the agent/persona *read* path (recall-style semantic search), since it's the lower-setup option and matches the mostly-read access pattern of "consult the vault." Add Local REST API only if entities need to *write* structured patches back into the vault programmatically rather than through plain `write`/`edit` file access (which works today with zero plugins, vault is just markdown on disk).

**Vault structure:** subsections owned per entity (`vault/agents/<name>/`, `vault/personas/<name>/`), plus a project-specialization area (`vault/projects/<project>/`) that a persona's project layer points into. This is where the "single vault, subsections owned by personas/agents, project docs mirrored or pointed-to" idea lands structurally.

**Project pointer mechanism — already native, no new tooling needed.** OMP context files and skills support `@path` imports, including `~/`-relative resolution (`omp://context-files.md`). A project's `.omp/skills/<persona>-project/SKILL.md` can be a thin stub whose body is `@~/vault/projects/<project>/<persona>.md` — the full content lives once in the vault, the project repo carries only a pointer that resolves at read time. No duplication, no sync step, works today. (Confirm this resolves correctly if the vault repo is a sibling checkout rather than literally under `~` — may need an absolute-path import or a symlink from `~/` into the actual vault location.)


## Prime Agent fit review

Cloned `PrimeIntellect-ai/prime-agent` (official org; a decoy repo `prime-RLM-agent/prime-agent` exists with "download" phrasing in its description — avoided) to `~/Work/github/PrimeAgent` and read its architecture/RLM/long-running-agents docs.

**Shared lineage, not an unrelated tool.** Its package is `@earendil-works/pi-coding-agent`, depending on `@mariozechner/pi-agent-core` — same upstream `pi` project (`badlogic/pi-mono`) that OMP itself forks (`omp://porting-from-pi-mono.md`). Prime Agent and OMP are sibling forks, not competitors. Same `packages/coding-agent` layout, same `SKILL.md`/skill-discovery conventions, same session-JSONL persistence idiom.

**What it has that directly answers the original "no daemon/cron" gap:**
- Daemon supervisor + resident session worker process, separate from any terminal client. Closing the terminal detaches; the worker keeps running. `prime-agent attach <id>` reattaches. This is the literal "own server instance that stores state" OMP lacks natively for *agent sessions* (OMP's daemon broker exists only for LSP/browser, not agent turns).
- Native scheduling: `/heartbeat` (user, recurring), `rlm_heartbeat` (agent-managed, programmatic, multiple), `prime-agent schedule` (one-time or cron, `0 9 * * 1-5` syntax). Solves the original ask directly — no external cron + `omp -p` workaround needed.
- `/goal` — durable objective that persists across turns/detach until explicitly completed/paused/cleared.
- `/autonomous` — bounded continuation policy (turn/token/wall-clock limits, quality gates) for unattended runs.
- `agent_message` skill — direct agent-to-agent messaging between daemon-resident sessions, survives detach (stronger than OMP's `hub`, which is live-session-only).

**What it does NOT have — checked, not assumed:**
- No vector store. Grepped `packages/coding-agent` for `embedding|vector store|vectordb|cosine` — zero hits in source. The "Continual Harness" (`/refine`) is plain durable state (prompts/memories/skill descriptions/subagent specs) with rollback-by-snapshot, not embeddings-backed recall. It's closer to OMP's `learned.md` + managed skills *combined with a governance/rollback layer OMP lacks*, not a Mnemopi-equivalent. If vector-backed recall stays a requirement, still need to bring that layer regardless of which harness hosts the agent tier.
- No persona/multi-identity concept beyond task-scoped `rlm(...)` children (subagents). Personas (tier 2/3 of this design) would need building on Prime Agent too — same amount of net-new work as on OMP.

**Execution-model difference to weigh, not just a config change:** Prime Agent's only model-facing tool is `ipython` — files, shell, tool use, and subagents are all Python calls inside a persistent kernel (RLM). OMP's discrete-tool-call model (`read`/`edit`/`bash`/`task`/etc.) is a different paradigm. Adopting Prime Agent for the agent tier means running a second harness with its own tool surface alongside OMP, not a drop-in library import.

**Verdict:** Prime Agent's daemon+heartbeat+goal+schedule stack is a stronger native fit for the agent tier's cross-session/own-server requirement than anything buildable on OMP without writing a custom daemon. Vector-backed memory is a wash — neither harness has it out of the box; Mnemopi (OMP) or a bolted-on embedding layer (Prime Agent) would be needed either way. Not a slam-dunk swap: choosing it means the agent tier runs as its own process/harness that personas/OMP sessions talk to (via `agent_message`/schedule/attach), not something absorbed into OMP's process.

## OMP integration feasibility (fact-checked this session)

User's stated direction: bring Prime Agent's daemon/heartbeat/goal/schedule capability *into* OMP as an attributed integration, possibly upstreamed as a PR, rather than adopting Prime Agent wholesale as a separate harness. Checked what that actually requires before committing the spec to it.

**License: no blocker.** Both MIT. Prime Agent: `Copyright (c) 2025 Mario Zechner, Copyright (c) 2026 Prime Intellect`. OMP itself (`can1357/oh-my-pi`): `Copyright (c) 2025 Mario Zechner, Copyright (c) 2025-2026 Can Bölük`. Same Mario Zechner root copyright in both — confirms the sibling-fork finding above at the legal-file level, not just architecturally. MIT-to-MIT porting needs only preserved notice + attribution, no license conflict either direction.

**Mechanism: this is a core-runtime feature, not an OMP "plugin."** Read `omp://extensions.md` and `omp://extension-loading.md` in full. OMP's extension API (`pi.on(...)`, `registerTool`, `ctx.setInterval`/`setTimeout`) runs **in-process, tied to one session, cleared on `session_shutdown`**, with explicit warning that unmanaged background work crashes the whole session if it throws. There is no extension hook for "keep this session's process alive after the terminal detaches" — that's exactly the capability gap identified earlier, and it's structural: OMP's daemon broker (used today for LSP/browser sharing) is core-runtime infrastructure, not something an extension can stand up for arbitrary agent sessions. Marketplace plugins (`omp://marketplace.md`) ship skills/commands/agents/hooks/MCP servers/LSP servers — none of those categories can add a resident session-worker daemon either.

**Consequence for the PR idea:** a literal "OMP sessions survive terminal close, reattach later" feature — the actual capability wanted — requires a genuine core-runtime change inside OMP (a new daemon-supervisor + resident-worker layer analogous to Prime Agent's, added alongside OMP's existing per-session process model), not an installable extension/plugin in OMP's current sense of the word. That's consistent with "I could create a PR for it" — it's PR-to-core scope, not marketplace-plugin scope. The alternative that *would* ship as an actual plugin today, with zero core changes: an external always-on MCP server (durable state + heartbeat/schedule triggers) that ephemeral OMP sessions call into — gets durable state and scheduled wake-ups, but not mid-session reattach, since the OMP session process itself still exits with the terminal either way.

## Fork-Prime-as-base vs port-daemon-into-OMP (analysis, this session)

User floated inverting direction: fork Prime Agent's codebase and integrate OMP's capabilities into it, rather than porting Prime's daemon subsystem into OMP. Read Prime Agent's `extensions.md`, `daemon.md`, `mcp-integrations.md`, `session-format.md` to check where the real mismatches are before answering.

**Good news first — shared lineage pays off structurally.** Prime Agent's extension API is near-identical to OMP's: same event names (`session_start`, `tool_call`, `before_agent_start`, `context`, `turn_start`/`turn_end`...), same `pi.registerTool()`/`registerCommand()` shapes, same tool-wrapper pre/post model. Porting *code* between the two forks is low-friction wherever both sides touch shared subsystems (provider calls, event bus, tool wrapping). This cuts both ways below.

**The daemon/scheduler subsystem is already cleanly isolated — confirmed, not assumed.** `daemon.md`: "The supervisor... does not execute providers, tools, compaction, bash, kernels, schedules, or transcript scans." Ticks/goals/autonomous continuations don't touch the kernel at all — `architecture.md`'s own diagram states "From the session queue onward, the same execution and persistence path is used when a prompt comes from a heartbeat, cron schedule, goal continuation, autonomous mode, or another agent instead of an attached user." Scheduling is generic prompt-injection into the normal session queue, not RLM-specific. This means the four things actually worth porting — (1) supervisor+resident-worker process wrapping, (2) session lease/attach/reconnect protocol, (3) scheduler/heartbeat/goal state + tick-to-prompt injection, (4) `agent_message` cross-session peer delivery — do not require adopting ipython/RLM at all. They're bolted onto "the session accepts a prompt," which OMP's `AgentSession` equally does.

**Other mismatches beyond ipython-vs-discrete-tools (asked for, found these):**
- **MCP exposure philosophy, not just mechanism.** Prime deliberately does *not* expose MCP servers as agent tools — "Consistent with Prime Agent's single-tool design" — every MCP integration is a Python skill wrapping the `mcp` SDK inside the kernel. OMP exposes MCP tools natively as discrete tool calls (`omp://mcp-config.md`: stdio/http/sse, OAuth broker, cross-tool config discovery). Forking Prime as base means either losing OMP's native MCP exposure or fighting the framework's own convention to add it back.
- **Skills are a different medium.** Prime skills are installable Python packages (pyproject.toml, typed callables imported into the kernel). OMP skills are markdown-first (`SKILL.md`, the exact mechanism behind the `mattpocock-skills` plugin installed this session) with optional Python-backed variants layered on top. Forking Prime as base orphans OMP's whole markdown-skill/marketplace ecosystem unless it's rebuilt Prime-side.
- **Session file schema differs** (entry types, role names — `bashExecution`, `child_usage_attributed`, etc. are Prime-specific vs OMP's own schema in `session.md`). Real but bounded: both are JSONL trees with `id`/`parentId`, so it's data-shape adaptation, not logic rewrite, wherever the daemon/scheduler needs to read/write session state.
- **Separate auth/credential storage** (`~/.prime/agent/auth.json` vs OMP's own auth-broker-gateway) and **separate TUI package** — each fork owns its own, another thing forking Prime as base would require re-bridging that porting the daemon into OMP sidesteps entirely (OMP's existing auth/TUI keep working untouched).

**Opinion: port the daemon subsystem into OMP; don't fork Prime Agent as the base.** Two reasons, both load-bearing:
1. **Surface-area asymmetry.** OMP's ecosystem (marketplace/plugins, native MCP, memory backends, skills-interop across 7+ provider conventions, advisor/watchdog, LSP, Agent Hub) is larger and more mature than what forking Prime would require rebuilding on its narrower, Python-first shell. The daemon/scheduler subsystem being ported the other direction is comparatively small and — per the architecture doc's own stated boundary — already isolated from the parts that would otherwise be hard to detach (providers/tools/compaction/kernels).
2. **Preserves already-settled design decisions.** This handoff's vault/MCP/skill/project-pointer design all assumes OMP's native tool surface (Smart Connections MCP as a tool, managed skills, `@` imports, WATCHDOG advisors). Forking Prime as base would silently undo those by routing everything back through Python, undermining work already done this session.

Named risk, not hidden: the port is still real infrastructure work — a new supervisor process, a versioned wire protocol (daemon.md's "Public Daemon Protocol v4": command envelopes, capability negotiation, generation-aware cursors, reconnect/resume, snapshot streaming). Smaller than the alternative, not small.

## Model-per-entity and local hardware (validated, not novel)

Per-entity model choice and mixed local/cloud hosting is standard 2026 multi-agent practice, not a special ask: "hybrid — both local and cloud, depending on task" is the current consensus (route privacy-sensitive/high-volume work local, complex reasoning to cloud). Orchestration/routing is CPU-bound and cheap (<50ms measured overhead); inference is the GPU-bound, expensive part (2-15s/call) and scales on separate infrastructure. Practical read for this project: routing logic (which entity, which model) can live anywhere cheap; only inference needs to sit near the GPU for locally-hosted entities. Already compatible with the OMP-native design (per-agent-def `model` field) and Prime Agent (explicit multi-provider/local-model support incl. Ollama/vLLM per its README) — neither harness blocks this, no extra research needed before spec-writing.

## Repo layout (refined)

Industry precedent (`agent-persona` project, GitOps-for-agents writeups) converges on one rule: **split by artifact type and change cadence, not by a single "code vs data" line.** Concretely for this project:

- **Repo 1 — core/harness.** Runtime choice (OMP-native vs Prime-Agent-hosted agent tier), shared MCP servers (vault bridge), orchestration glue. Code, CI, versioned releases.
- **Repo 2 — registry.** Entity definitions only: name, role (agent/persona), model, system prompt, tool grants, WATCHDOG entries. Lightweight text/YAML, matches the "persona registry holds names + invocation pattern, full definition loaded on demand" pattern from `agent-persona` — keeps per-session context cost down since not every entity's full prompt loads every time.
- **Vault — open call, not settled.** Notes are text and diff well in git; the *embedding index* (Smart Connections' local store) does not — regenerable, model-tied, would bloat history if committed. Two viable shapes: (a) vault lives inside repo 2 with the index gitignored, or (b) vault gets its own repo 3 once it grows attachments/size or needs a different sync cadence (e.g. Obsidian Sync/iCloud instead of git). Start with (a); split out only when it hurts.
- **Project repos** (this repo included) stay thin: a pointer stub per relevant entity, not a copy. See vault design section above.

This nets to the "at least 2 repos" instinct being right, with the vault question left explicitly open rather than forced into either repo.

## Naming — settled (agent/persona, plain terms)

Dropped the "aGent" affectation — "persona" was already a loaded-enough term on its own, no need to invent a spelling trick for the other side. Plain **agent** / **persona**, precise operational definitions deferred to the formal spec rather than pinned down here. No further naming decision pending.


## Decisions log (all resolved — see SPEC.md for the settled forms)

- **Naming** — settled: plain agent/persona; precise operational defs in `SPEC.md` §2.
- **Vault repo placement** — settled: start inside repo 2, index gitignored; split out only on size/attachments/sync-cadence trigger. `SPEC.md` §5, resolved-item 1.
- **Vault backing** — settled: Obsidian + Smart Connections (human-browsable wanted, not mnemopi-only). `SPEC.md` §7, resolved-item 2.
- **First persona** — settled: Phi (OMP/pi expert), WS2 pilot. Resolved-item 3.
- **Project-pointer resolution** — settled: `~/vault` symlink convention, created/validated by a setup process (WS7). `SPEC.md` §7.3, §12.7a, resolved-item 4.
- **Runtime direction** — settled: port daemon/scheduler subsystem into OMP as core-runtime PR; drop ipython; keep OMP tool model. Spec carries full architectural detail (Q3 option (i)). `SPEC.md` §8, WS1.
- **Local-hardware entity placement** — fully deferred (post-spec, non-blocking). Heuristic recorded: personas-local-first, coordinator-cloud. Resolved-item 5.

## Suggested skills for next session

- `codebase-design` — when starting WS1 (daemon port) or WS3/WS4 (memory/vault MCP servers), for interface/seam decisions before writing code.
- `prototype` — validate Smart Connections MCP against a small real vault, and bank-scoped mnemopi recall, before committing WS3/WS4 structure.
- `tdd` — WS1's crash-recovery/reconnect protocol and WS3's bank-isolation are exactly the invariants worth building test-first.

## Non-duplication note

`SPEC.md` (DRAFT v0.2) now exists alongside this file — the formal spec, decomposed into parallel workstreams (WS1–WS7 + WS7a setup) with per-section model/effort and a per-workstream draft→review→test→iterate loop plus a final Opus-level whole-solution review stage. All §10 items resolved (item 5 deferred, non-blocking). This handoff is the rationale/decision record; the spec is the buildable plan. No ADRs/issues yet.
