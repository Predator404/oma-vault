---
type: reference
created: 2026-08-19
tags: [reference, process, follow-ups]
---

# PR follow-up register

Outstanding follow-ups carried out of **merged** PRs on the OMA fork
([Predator404/oh-my-pi-agent](https://github.com/Predator404/oh-my-pi-agent)).

> This file is one instance of a **vault-wide convention**: every project keeps a
> `projects/<project>/follow-ups.md`, and when a PR is merged its still-open
> follow-ups — non-blocking review findings, deferred scope, TODOs, and
> "verified live but not tested" gaps — are recorded here **before the PR is
> considered closed-out**, so nothing falls through the gap between *merged* and
> *done*. See [[decisions/0003-pr-follow-up-register.md]] and the vault
> convention in [[reference/obsidian/vault-conventions.md]].

Mark an item `[x]` when addressed (ideally by the PR that resolves it); leave
`[ ]` while open. Deep technical threads get their own note under
`investigations/` and are linked from here rather than restated.

## PR #1 — fix(oma/entity): persist detached broker + clean CLI exit + caller cwd

Merged 2026-08-19 (`f0093bb0` → `oma`). Two-axis code review passed with zero
blocking findings. Outstanding (all non-blocking):

- [ ] **Relocate the spawner seam.** `AgentWorkerSpawner` type + `killAll` live
  in `broker.ts` (~352–356); fold `killAll` onto the `WorkerSpawner` interface
  beside its sibling in `agent-supervisor.ts`. _(Standards / placement)_
- [ ] **Refresh stale test comment.** The present-tense "red signal" comment in
  `broker-agent-persistence.test.ts` (~435–440) narrates pre-fix state, but the
  fix ships in the same diff — misleading to a future reader. _(Standards / docs
  nit)_
- [ ] **Event-loop-unpin regression test.** CLI unit tests assert
  `AgentDaemonClient.close()` is called, but the actual Bun event-loop unpin (the
  "60–120s cold-start hang") is verified only live (HANDOFF), not by a test.
  _(Spec / coverage)_
- [ ] _(minor)_ 7-param positional clump in `runRuntimeDispatch`, and a
  duplicated `#scheduleIdleShutdown(); return;` re-arm pattern in `broker.ts`.
  _(Standards / smell)_

## PR #2 — feat(advisor): direct @@<name>: addressing for in-session advisors

Merged 2026-08-19 (`ae834aa3` → `oma`). Two-axis review passed after fixes
(prompts moved to `.md` templates; test cleanup). Outstanding (all non-blocking):

- [ ] **Deferral integration test.** The parser and the advisor-side capability
  injection are tested, but the primary-deferral path (`resolveAddressedAdvisor`
  + the directive pushed into `beforeAgentStartSystemPrompt` in
  `agent-session.ts`) has no deterministic test. Add a session-level test that a
  `@@<advisor>:` user message appends the deferral directive. _(Spec / coverage)_
- [ ] **Live end-to-end unverified.** The advisor's model-driven answer to a
  `@@Phi:` address is interactive-only and was not exercised in an automated run.
  _(Spec / coverage — interactive)_
- [ ] _(minor)_ redundant `fs.mkdir` before `Bun.write` in
  `advisor-address-wiring.test.ts` (~31–37); defensible since `cwd` is also
  handed to `createAgentSession`. _(Standards / smell)_

## PR #4 — fix(mnemopi): resolve blank dbPath to persistent storage (#9360)

Merged 2026-08-22 (`eb836252` → `oma`, merge `9208a990`). Reviewed pre-merge
(diff read + independent red/green test run); minimal trim-or-default fix at
`mnemopi/config.ts:47` matching the sibling `embeddingModel` pattern. Fixes
upstream can1357/oh-my-pi#9360. Outstanding (non-blocking):

- [ ] **`embeddingApiUrl` / `llmBaseUrl` blank-string audit.** Both are read via
  the same `settings.get(...)` as `mnemopi.dbPath` and would share the identical
  empty-string → silent-degradation hazard if any consumer ever defaults them
  with `??`. Only `dbPath` was fixed (the reported P1); audit both and apply the
  same trim-or-default guard where they are consumed. _(Standards / robustness)_

## PR #5 — fix(compaction): preserve post-snapshot suffix on speculative apply (#9351)

Merged 2026-08-22 (`f12a7b4b` → `oma`, merge `56649e4f`). Reviewed pre-merge
(full diff read + independent test run: speculation 11 pass, session-context 14
pass, neighbours green). **Root cause:** an armed speculative compaction summarizes
a prefix snapshot; when the branch advances before apply, `#commitAutoCompactionResult`
used the armed `firstKeptEntryId` as-is and the V2 remote-replacement rebuild in
`buildSessionContext` suppressed the kept-message loop, so post-snapshot user/tool
turns were dropped. **Fix:** record `speculativeSuffixStartId` on the compaction
entry at apply time and replay that suffix after the remote replacement history
(shared V1/V2 apply path). Fixes upstream can1357/oh-my-pi#9351. Outstanding
(non-blocking):

- [ ] **V1 speculative-apply coverage gap.** The fix lands on the shared V1/V2
  path, but the regression test in `compaction-speculation.test.ts` exercises only
  the V2 remote-payload branch (the verified production repro). V1 was not
  independently reproduced or asserted — add a V1 speculative-apply case that
  advances the branch before apply and verifies the post-snapshot turn survives.
  _(Spec / coverage)_

## Direct push — fix(oma/advisor): surface direct-address answers regardless of severity

Pushed direct to `oma` 2026-08-22 (`719bede6fc`; lint-green chore `9b756ec6c7`
sorts imports in the PR #4 mnemopi test), NOT PR'd — user directed merge/push.
Independently verified (subagent PASS: advisor suite 167 pass / 0 fail; idle +
no-severity address now routes `preserve`, not `aside`). **Root cause:** a
`@@<name>:` reply emitted with a non-interrupting severity (nit / no severity —
the natural choice for an informational answer) routed to the `aside` channel,
whose advisor dispatcher is `skipIdleFlush:true`; after the primary deferred and
went idle the answer stranded in the yield queue until the next user prompt
("Phi will answer but nothing happens"). `concern`/`blocker` answers surfaced
via preserve/steer, masking it. **Fix:** thread a `directAddress` signal from the
defer site (`agent-session.ts` `markDirectAddress`) through `SessionAdvisors`
(`#pendingDirectAddress`, consumed in `#routeAdvice`) into
`resolveAdvisorDeliveryChannel`, which now returns `preserve` (idle) / `steer`
(streaming) for a direct-address note regardless of severity. Outstanding
(non-blocking):

- [ ] **Session-level delivery integration test.** Only the resolver branch is
  unit-tested (`advisor.test.ts`); the end-to-end route — `markDirectAddress` →
  `#routeAdvice` consume → visible preserved card while idle — has no
  deterministic test. Folds into PR #2's still-open deferral integration test:
  one session-level test should cover both the deferral directive push AND the
  advisor's no-severity answer surfacing while idle. _(Spec / coverage)_
- [ ] **Live end-to-end still unverified.** Same interactive gap as PR #2's
  "Live end-to-end unverified" — the no-severity `@@Phi:` answer surfacing while
  idle was found by code inspection, not exercised through a real model/daemon.
  _(Spec / coverage — interactive)_
- [ ] _(latent, documented)_ `directAddress` wins over `preserveOnly` in
  `resolveAdvisorDeliveryChannel`; safe today because `preserveOnly` is set only
  by the headless drain after the primary loop stops streaming (resolves to
  `preserve`), but a future caller setting `preserveOnly` during an active stream
  with a pending direct address could bypass the headless-drain intent. Guarded
  by an inline comment at the branch. _(Standards / latent)_
- [ ] **Open the PR.** Direct-pushed per instruction; going-forward convention is
  PRs (`gh pr create --repo Predator404/oh-my-pi-agent`). _(Delivery)_

## Direct push — chore(oma/process): make follow-up-register updates a binding AGENTS.md rule

Pushed direct to `oma` 2026-08-22 (repo-root `AGENTS.md`), NOT PR'd. Independently
reviewed (reviewer APPROVE, 0.86: `AGENTS.md` is the correct enforcement point).
**Root cause:** the coding agent reliably updated `CHANGELOG.md` on completion but
NOT this register, because the changelog rule lives in the binding, auto-loaded
`AGENTS.md` while the follow-up convention lived only in the vault
([[decisions/0003-pr-follow-up-register.md]]) and advisor guidance — no standing
instruction for the agent. **Fix:** added a `## Follow-ups` section to `AGENTS.md`
(after `## Changelog`, before `## Releasing`), parallel in force — a change is not
"done" until BOTH the changelog and this register are updated; marked OMA-fork-only
so upstream rebases don't carry it. Outstanding (non-blocking):

- [ ] **Efficacy unverified over time.** The rule is in place; whether it actually
  changes agent behavior across future sessions can only be confirmed by
  observation. _(Process / verification)_
- [ ] **Permanent local delta.** The `~/vault/...` path makes the section a
  fork-local delta to preserve across every upstream `AGENTS.md` rebase/merge.
  _(Standards / maintenance)_

## Branch `fix/entity-mcp-tool-registration` — surface a resident worker's own MCP tools (not yet PR'd)

Committed on a branch off `oma` (`116e7a30cb`), verified live, no PR yet.
**Root cause:** an entity worker builds its own `MCPManager`
(`buildEntityMcpManager`) and hands it to `createAgentSession` as
`options.mcpManager` — the subagent-inheritance shape, which `sdk.ts` only
registers tools for when it OWNS the manager (`enableMCP && !options.mcpManager`).
A resident entity has no parent, so its memory (C2 bank) + vault (C3 section)
servers connected but their `recall/retain/forget` + vault tools never surfaced
to the entity. **Fix:** extracted `registerEntityMcpTools()` in
`agent-worker-main.ts`, called after session creation (register `getTools()` +
wire `setOnToolsChanged → refreshMCPTools`, mirroring the owner path). Discovered
2026-08-21 while attaching to Phi; she now exposes `mcp__memory_*` +
`mcp__vault_*` alongside her builtins. Tests: `entity-mcp-registration.test.ts`
(unit, doubles) + a `getTools()` aggregation assertion in
`persistent-agents-e2e.test.ts`. Outstanding:

- [ ] **Open the PR + merge.** Branch is committed and green but not PR'd (fork
  PAT lacks "Pull requests: write" — manual PR, per HANDOFF). _(Delivery)_
- [ ] **Broker predates the fix.** The resident broker still runs pre-fix
  source; respawned workers already pick it up (each is a fresh `bun cli.ts`),
  but a broker restart is the clean cutover — shared with the stale-alias item
  below. _(Ops)_

## Project open items (not tied to a single PR)

Carried out of the persistent-agents build resume-doc (`personas/phi/oh-my-pi-agent/HANDOFF.md`)
on 2026-08-19 so open work lives in the register, not the handoff narrative.
Deep threads have their own `investigations/` note, linked here.

- [ ] **Prompt-wake gap (functional).** A one-shot `oma entity prompt <id>` to
  a detached, idle worker enqueues the message but never wakes the worker to
  run a turn (`busy` stays false, no reply). Full trace + fix direction:
  [[investigations/prompt-wake-gap.md]]. Same daemon-delivery domain as PR #1;
  not in that PR. _(Spec / functional)_
- [ ] **Attach / response-retrieval ergonomics (usability).** No non-interactive
  path to send a prompt and read an entity's reply: `prompt` is fire-and-forget,
  output is TUI-only (`attach`), `history://` doesn't resolve entity sessions,
  the transcript file is a hunt, and `prompt` takes only the UUID (not the
  name). Full detail + proposed CLI: [[investigations/attach-retrieval-ergonomics.md]].
  _(Standards / UX)_
- [ ] **Vault embed-model deviation.** The vault index uses
  `sentence-transformers/all-MiniLM-L6-v2`, not the spec's `TaylorAI/bge-micro-v2`,
  because a corporate proxy blocks HuggingFace. Re-indexing inside
  Obsidian still needs HF unblocked (Smart Connections caches its model via
  Electron's browser Cache API, unseedable from disk); the headless
  `vault-index/build-index.ts` is the proxy-proof maintenance path. Revisit if
  HF becomes reachable. _(Spec / deviation)_
- [ ] **Cleanup — scratch dir.** VaultIndex left
  `persistentAgents/vault-index/tester-scratch/`; safe to delete. _(Housekeeping)_
- [ ] **Deferred (from spec).** SPEC resolved-item 5 (local-hardware entity
  placement) remains deferred, non-blocking. Heuristic recorded:
  personas-local-first, coordinator-cloud. _(Deferred scope)_
- [ ] **Per-entity icon + color (feature).** Entities carry `icon`/`color` in
  their record. DONE: phase 1 (schema / loader / writer / CLI / tests) and
  phase 2 foundation + **advisory card** + **Agent Hub roster** + ascii
  fallback — a theme bubble-bg blend, icon/color threaded onto `AgentRef`, and
  advisor→entity resolution. Still open: (a) the **primary-transcript speech
  bubble** (touches the shared `AssistantMessageComponent`; deferred, own
  reviewed change, not headlessly verifiable); (b) the **roster across the
  daemon boundary** — icon/color aren't on the `@oh-my-pi/pi-wire`
  `AgentSnapshot`, so `oma entity attach`'s mirrored roster stays plain until
  the wire type + collab host/guest mapping are extended (cross-package). Full
  status + files: [[investigations/entity-icon-color.md]]. _(Feature / TUI)_
- [ ] **Stale model alias on the long-resident broker (functional).** The
  daemon broker (`oma.ts __omp_worker_daemon_broker`) resident since 2026-08-19
  resolves the `anthropic/opus` family alias to the retired `claude-opus-4-0`,
  which the Anthropic API now returns `404 not_found_error` for — so **every**
  turn of an entity pinned to the alias errored (`stopReason: error`, empty
  content). Newer interactive `oma` sessions resolve the alias to
  `claude-opus-4-8` correctly; only the pre-4.8 broker in memory is stale, and a
  fresh worker it spawns inherits the bad resolution. Discovered 2026-08-21
  while attaching to the live Phi entity. **Workaround applied:** pinned a
  concrete id in `entities/phi.md` (`anthropic/claude-opus-4-8`) — a concrete id
  bypasses the alias-to-newest chain and the stale broker accepts it. **Real
  fix:** provide a supported broker-restart path (there is no `oma daemon`
  command; the broker only reloads current source on process restart) and then
  revert Phi to the self-updating `anthropic/opus` alias. _(Spec / functional)_
- [ ] **Cross-session hub peering (functional / UX).** No supported way to load
  a resident entity as a peer in another omp session's Agent Hub: the hub is
  cwd-scoped and the entity is single-session, so `hub send to="phi"` from a
  different project fails `Unknown agent`. `oma entity spawn <name> --cwd
  <other>` deterministically times out at the daemon (30s, raw stack trace, no
  session created); `attach` is one-shot, not a bridge. Discovered 2026-08-21
  driving Phi from the credit-engineering-tests session. Full trace + fix
  directions: [[investigations/cross-session-hub-peering.md]]. Same
  daemon-delivery domain as PR #1 and the attach-retrieval / prompt-wake gaps;
  not in any of them. _(Spec / functional)_

## Parked — Prime↔OMA comparison (2026-08-21)

Full module-by-module comparison of Prime Agent's daemon subsystem (SOURCE,
`~/Work/github/PrimeAgent` @ `849c9211`) vs OMA's port, run this session via 4
read-only scouts. **MERGED into `oma` 2026-08-21** — retargeted onto latest
upstream (oma `a1128ff`→`c3067a3d`, 498-commit drift resolved with zero
conflicts; per-fix dev+tester pairs + independent retarget re-verification;
type-clean + 370 area tests green; backup tag `backup/worktree-snapshot`). Five
commits landed in dependency order, each also pushed as a per-fix branch:
entity-mcp tools (`8d6120bd`), per-entity icon/color incl. daemon-wire
`AgentSnapshot` (`6cdbfad9`), prompt-wake idle delivery (`b643458f`),
non-interactive retrieval + fast-fail `session_already_active` + `entity daemon
status|shutdown|restart` (`7d949a53`), post-compaction kernel state-manifest
(`c3067a3d`). **This CLOSES the open items above:** prompt-wake gap;
attach/retrieval ergonomics; stale-model-alias (a `daemon restart` path now
exists); and the icon/color roster-across-the-daemon-boundary. **Still open:**
the items below (parked by decision), plus icon/color's primary-transcript
bubble and the cross-session PEER bridge (fast-fail + name/UUID shipped; the
bridge itself did not).

- [ ] **Cross-session PEER bridge + worker crash-recovery (architectural).**
  CORRECTION (2026-08-21, verified): the **entity-runtime broker is
  machine-global** (one per config-root, keyed on the `agent-runtime` service
  name), NOT per-cwd — the earlier scout conflated it with the separate
  per-project LSP/browser broker. All entity spawns route to the one broker, so
  cross-cwd fast-fail already works (landed above). The REAL remaining gap: a
  resident entity is still not injected as an addressable **peer** into another
  omp session's cwd-scoped IRC hub (`hub send to=<entity>` across the boundary),
  and OMA lacks Prime's worker crash-recovery — retry backoff `[250,1000,5000]ms`
  (Prime `daemon-supervisor.ts:139`), replacement-supervisor election via atomic
  lease, and generation fencing of adopted workers (`agent-supervisor.ts:241-260`
  has none). Fix options: (a) a supervisor→session peer bridge that registers a
  resident entity onto a live session's hub; (b) worker-recovery backoff+fencing.
  Needs a deliberate architectural decision. Ref
  [[investigations/cross-session-hub-peering.md]] (its per-project-broker premise
  is corrected here). _(Spec / arch)_
- [ ] **Refinement / rollback governance layer.** Versioned snapshot+rollback
  over the entity's own config surface (systemPrompt, memories, skills, subagent
  specs). OMP has the targets but no versioning. **Lean on git first** — the
  registry + vault are already git repos (`~/oma-registry`), so operator-side
  rollback of prompts/skills/notes is already possible. Genuinely-missing pieces:
  (a) **memory-bank** snapshot/rollback (the one non-git surface — it's an
  index); (b) an in-runtime, entity-invocable checkpoint/rollback verb. Build (a)
  only if memory drift bites; defer (b) until an entity actually self-modifies
  enough to need mid-run rollback. Kernel-independent; do NOT port Prime's
  kernel-entangled Continual Harness wholesale. _(Feature / deferred)_
- [ ] **Compaction-quality lever (higher-leverage general fix).** The broad
  "compaction trap" is best mitigated by a smarter summarizer that preserves the
  active plan/goal/todo verbatim — so working state need not be offloaded at all.
  Overlaps with the Prime commit-ports below. _(Spec / robustness)_
- [ ] **Port applicable Prime fixes landed since the pull (verify vs current
  upstream `main` FIRST — several touch `core/agent-session.ts`, which in OMA is
  OMP's own file, so post-rebase OMP may already have equivalents).** From
  `git -C ~/Work/github/PrimeAgent log main..origin/main` (39 commits): goal
  continuation after compaction (`20b54977a`), resume interrupted work after
  auto-compaction (`114a1d6af`), hold goal continuations while a subagent is
  unsettled (`e51d2266c`), **harden resident session lifecycle** (`d98d0762c`,
  +693 lines — biggest), normalize daemon socket paths before deriving identity
  (`8189b12d6`), move supervisor authority records out of `$TMPDIR`
  (`26f7f1a14`), keep working-status timer across session re-entry (`48b6478e7`),
  typed codes for `continue` precondition failures (`35103cb42`). _(Maintenance)_
- [ ] **Lower-priority missing Prime surfaces.** Agents-view TUI (live roster
  browser: model/effort/status, sort-by-last-message, search); client-owned vs
  resident worker distinction (one-shot cleanup grace, reconnect-cancels-cleanup);
  interactive `/heartbeat` surface (OMA is CLI-flags only). _(Feature)_
- [x] **Kernel-variable state / compaction — SHIPPED (`c3067a3d`).** Confirmed
  OMP's persistent `eval` kernel survives compaction + detach (lifetime bound to
  the AgentSession owner, not the context window), so the gap was awareness-only.
  A post-compaction live-state manifest (`src/eval/py/state-manifest.ts` +
  `post-compaction-manifest.ts`, wired in `sdk.ts` on `auto_compaction_end`) now
  re-injects live kernel var names/types after each successful compaction.
  RLM/kernel machinery was NOT ported (decided — OMP `eval` already is the
  persistent kernel as a discrete tool). Optional follow-on: snapshot
  `frozenset(user_ns)` at kernel init as the manifest baseline (the module
  accepts it) instead of the hardcoded PRELUDE_BASELINE. _(Done + minor follow-on)_

## Repo state & housekeeping (2026-08-21 session close)

Snapshot for a fresh context — the repos are "sorted" to this shape.

**Git topology (both local and `origin`):** only two branches — `main` (tracks
`upstream/main` = `can1357/oh-my-pi`, kept for rebasing) and `oma` (the fork's
working branch, `origin=Predator404/oh-my-pi-agent`). All feature/fix branches
pruned. `oma` tip `c3067a3d`; `origin/oma` in sync; `main`/`origin/main` in sync.

**Merged into `oma` this session** (retargeted onto latest upstream, 498-commit
drift, zero conflicts): entity-mcp tools `8d6120bd`, per-entity icon/color incl.
daemon-wire `AgentSnapshot` `6cdbfad9`, prompt-wake idle delivery `b643458f`,
retrieval + fast-fail lease + `entity daemon status|shutdown|restart` `7d949a53`,
post-compaction kernel state-manifest `c3067a3d`.

**Backups (local-only tags — delete once confident):** `backup/oma-preupdate` =
`ae834aa` (old pre-rebase oma tip; holds the old advisor / entity-daemon /
persistent-agent branch content). `backup/worktree-snapshot` = `c76e7d7` (full
pre-split working tree incl. the old `fix/entity-mcp-tool-registration` base).
Every pruned branch's content is preserved here and/or as rebased equivalents in
`oma`.

**Process — PRs going forward.** This batch was merged directly (fast-forward)
after review; from here, land changes via PRs. `gh` PR-write works now (token has
`repo` scope). GOTCHA: for this fork `gh pr create` defaults the base repo to
UPSTREAM — always pass `--repo Predator404/oh-my-pi-agent`.

**Repo hygiene — `oma` is NOT fully green independent of our work** (pre-existing;
worth a cleanup / upstream-sync pass): `bun run check:types` reports (1)
`src/oma.ts:32` — `.ts`-extension import in the `oma` bin shim; (2)
`test/blob-provider-fallback.test.ts:8` — upstream blob-broker fixture missing
`compat` on `Model<Api>`. Plus Bucket-A biome drift in upstream `blob-broker/*`.
None are from this session's fixes.

**Verification gap — no live end-to-end smoke.** The 5 merged fixes are unit/
integration-verified (type-clean bar the 2 above; 370 area tests) but were NOT
exercised through a real model/daemon (avoided disturbing the live Phi entity).
To confirm interactively: `oma entity prompt --wait`, `oma entity daemon restart`,
and a real auto-compaction manifest injection.

**Stale-model-alias — now revertible.** `entities/phi.md` was pinned to a concrete
`anthropic/claude-opus-4-8` to dodge the stale long-resident broker. With `entity
daemon restart` shipped, restart the broker then revert Phi to the self-updating
`anthropic/opus` alias.

**Untracked:** `.omp/skills/phi-project/` (the project-pointer stub) is left
untracked by design; decide whether to commit it into `oma` (SPEC §7.3) or keep it
machine-local.
