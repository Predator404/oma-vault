---
type: reference
created: 2026-08-19
tags: [reference, investigation, entity, tui, theming]
---

# Per-entity icon + color — visual identity for entity responses

Goal: make it clear which persona/agent is responding by giving each entity a
distinct icon (glyph) and color, rendered on its messages/label. Origin: user
request 2026-08-19. Chosen approach = option C ("extend the entity record +
renderer"); this note tracks the staged build.

## Design decisions

- **Storage = entity record (C1 frontmatter).** Two optional fields on each
  `entities/<name>.md`: `icon` and `color`. Keeps per-entity identity as pure
  data (repo 2), no code change to add/edit an entity — consistent with the rest
  of the C1 schema.
- **`color` = `ThemeColor` token, not hex.** Validated with the codebase's
  single source of truth `isValidThemeColor` (`modes/theme/schema.ts`). Matches
  `ModelRoleInfo.color` (`config/model-roles.ts`), so the phase-2 renderer can
  apply it directly via `theme.fg(color, …)` and it stays legible across
  dark/light themes. Rejected free hex to avoid a second render path and
  theme-contrast bugs.
- **`icon` = exactly one grapheme cluster.** Validated with `Intl.Segmenter`
  (dependency-free; correct for ZWJ emoji). A compact marker, not a text label,
  so it can prefix a name without breaking layout.

## Status — phase 1 DONE (schema + loader + writer + CLI + tests)

Landed on branch `oma` (`~/Work/github/oh-my-pi/packages/coding-agent`):

- `src/entity/schema.ts` — `icon?: string` + `color?: ThemeColor` on
  `EntityRecordMeta` and `ResolvedEntityConfig`.
- `src/entity/loader.ts` — `parseIcon` (single-grapheme) + `parseColor`
  (`isValidThemeColor`) validators; wired into `parseEntityMeta` and
  `resolveEntityConfig`.
- `src/entity/record-writer.ts` — `CreateEntityFields.icon/color`, serialized in
  `createEntityRecord`, and `icon`/`color` cases in `applyEntityFieldUpdate`
  (so `entity config --set` round-trips; re-validated on write).
- `src/cli/entity-cli.ts` + `src/commands/entity.ts` — `--icon`/`--color` create
  flags and an `icon/color` line in `entity show`.
- `test/entity-registry.test.ts` — reference record exercises both fields;
  resolution + populated-field assertions; reject-multi-glyph-icon and
  reject-bad-color-token cases.
- Reference record `entities/phi.md` now carries `icon: "🔷"`, `color: accent`.

Verified: `bun test` entity suites green (49 pass); Biome clean on the 6 changed
files; `check:types` clean for these files (the one `check:types` error is the
pre-existing `src/oma.ts` `.ts`-import issue, unrelated). Live CLI smoke:
`create --icon 🟣 --color success` → written + shown; `config --set
color=warning` persists; `color=chartreuse` and `icon=AB` both rejected.

## Status — phase 2 (renderer wiring): foundation + advisory + roster DONE

Landed on branch `oma`:

- **Theme helper.** `modes/theme/color.ts` `mixHex`; `theme-class.ts`
  `getBubbleBgAnsi(color)` — blends the entity color ~16% over `customMessageBg`
  for the speech-bubble fill.
- **Identity threading.** `AgentRef` + `RegisterInput` + `register()` carry
  optional `icon`/`color`; `CreateAgentSessionOptions.agentIcon/agentColor`
  thread them through the sdk registration (`sdk.ts`); the entity worker passes
  `config.icon/color` (`launch/agents/agent-worker-main.ts`).
- **Advisory card (surface 1).** `AdvisorNote.icon/color`; the advisor manager
  (`session/session-advisors.ts`) resolves each advisor to its entity record by
  name/slug via a cached `discoverEntities()` map and stamps icon/color onto the
  routed note; `modes/components/advisor-message.ts` renders the glyph + a
  color-tinted bold label + an entity-colored rail + a bubble-bg fill. In-process
  — works in a normal `omp` session AND an entity worker.
- **Agent Hub roster (surface 2).** `agent-hub-renderer.ts` `entityGlyph()`;
  `agent-hub.ts` prepends the glyph and tints the name with the entity color
  (dim fallback) on the roster row and the detail header.
- **Ascii fallback.** Both surfaces drop the glyph under the `ascii` symbol
  preset (`theme.getSymbolPreset()`), keeping the color tint.
- **Tests.** `test/advisor/advisor.test.ts` — two cases assert the rendered
  bytes carry the glyph, the accent fg escape, and the bubble-bg escape for an
  entity note, and none for an entity-less note.

Verified: `check:types` clean for the changed files (only the pre-existing
`src/oma.ts` error remains); Biome clean; 182 advisor+entity tests green, 351
across the broader affected suites (theme/hub/advisor). Render proven by the
byte-level test assertions (a standalone stdout dump is blocked only by the
`pi-natives` addon not being built in a bare `bun` process; the test runner
renders fine).

## Remaining — deferred

1. **Primary transcript bubble (surface 3).** Wrapping an attached entity's
   assistant turns in a bubble touches the shared `AssistantMessageComponent`
   (all sessions), and the attached-entity identity isn't reachable at that
   render site. Deferred by decision — its own reviewed change; can't be
   visually verified in the current headless environment.
2. **Roster across the daemon boundary.** The roster glyph/tint is populated on
   the LOCAL `AgentRegistry` (sdk registration). `oma entity attach` mirrors the
   worker's agents to the client via the `@oh-my-pi/pi-wire` `AgentSnapshot`,
   which does NOT carry icon/color — so the cross-process attach roster stays
   plain until the wire type + `collab/host.ts`/`guest.ts` mapping are extended
   (cross-package change). The advisory surface is unaffected (in-process).

## Links

- [[follow-ups]] — tracked as a project open item
- `config/model-roles.ts` — the `{ tag, name, color }` pattern reused
- `modes/theme/schema.ts` — `isValidThemeColor` / `ThemeColor`
