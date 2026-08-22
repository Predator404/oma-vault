---
type: reference
tags: [reference, vault, conventions, templater]
---

# Vault note conventions

Canonical rules for **how notes are created in this vault**. Humans and entities
(personas, agents, skills) MUST honor these when writing notes so the vault stays
consistent and Smart-Connections recall stays clean.

If you only read one thing: **put a new note in the right area folder and let the
Templater folder template fill its skeleton** — or, if you are writing the file
programmatically, **seed it from the matching `Templates/<Template>.md` and fill
the frontmatter** yourself.

## Areas and their templates

Every top-level area has one folder template. Creating an **empty** note anywhere
inside the area (including sub-folders) auto-applies it (see *How it fires*).

| Area | Holds | Folder template |
|------|-------|-----------------|
| `agents/` | Per-agent working notes | `Templates/Agent Note.md` |
| `personas/` | Curated, source-grounded domain knowledge | `Templates/Persona Note.md` |
| `projects/` | One `(project, persona)` note — an entity's knowledge of a project | `Templates/Project Note.md` |
| `reference/` | Tooling / library reference and doc mirrors | `Templates/Reference Doc.md` |

`Templates/Daily Log.md` is a general-purpose template (manual insert), not tied
to an area.

> Note vs record: the vault holds **notes**. An entity's C1 **record** (its
> system prompt + metadata) lives in the registry at `registry/entities/<name>.md`,
> never in the vault.

## How it fires

Templater applies a folder template to a **newly created, empty** note whose path
is inside the mapped area (deepest match wins). This is why copying files with
content into an area — e.g. the verbatim doc mirrors under
`personas/phi/oh-my-pi/` or `reference/obsidian/templater/` — does **not** clobber
them: non-empty files are skipped. Those mirror sub-trees are also listed in
Templater's *ignore folders on creation* as belt-and-suspenders.

Auto-fire on new-note creation additionally requires the per-vault toggle
**Settings → Templater → "Trigger Templater on new file creation"** to be on. It
is a security gate stored in Obsidian's local storage (not in the vault), so it
must be enabled once, in-app. Without it, use **Insert Template** manually; the
folder→template mapping still tells you which template to pick.

## Writing notes programmatically (entities / scripts / MCP)

When a note is created **outside** the Obsidian UI (an entity's `write`, a shell
copy, or a vault MCP `create_note`), Templater does **not** run. In that case:

1. Read the matching `Templates/<Template>.md` for the area.
2. Reproduce its frontmatter and section skeleton in the new note.
3. Resolve the Templater placeholders yourself:
   - `<% tp.file.title %>` → the note title (filename without extension)
   - `<% tp.file.creation_date("YYYY-MM-DD HH:mm") %>` and
     `<% tp.date.now("YYYY-MM-DD") %>` → the current date/time in that format
   - `<% tp.file.cursor() %>` → drop it (just leave the cursor position empty)
4. Fill the area-specific frontmatter (`entity`, `project`, `persona`, `type`)
   and keep claims **source-grounded** (cite the exact file/type/contract).

## Adding or changing a template

- Templates live in `Templates/` and are plain notes using Templater `<% %>`
  syntax. Add a `.md` there and it becomes available to **Insert Template**.
- To bind a new area to a template, add a `folder_templates` entry in
  `.obsidian/plugins/templater-obsidian/data.json` (`{ "folder": "<area>",
  "template": "Templates/<Template>.md" }`).
- Syntax and the full `tp.*` API: see the mirrored docs at
  [reference/obsidian/templater/index.md](templater/index.md) — in particular
  [syntax.md](templater/syntax.md), the
  [internal-functions](templater/internal-functions/overview.md) modules, and
  [settings.md](templater/settings.md). Upstream repo:
  <https://github.com/SilentVoid13/Templater>.

## PR follow-up register (per project)

Every project keeps a **PR follow-up register** at
`projects/<project>/follow-ups.md`. When a PR for that project is **merged**, its
still-open follow-ups — non-blocking code-review findings, deferred scope, TODOs,
and "verified live but not tested" gaps — are appended there **before the PR is
treated as closed-out**, so work does not vanish in the gap between *merged* and
*done*.

- One section per PR (`## PR #<n> — <title>`, with merge date + commit) of
  checkbox items; flip `[x]` when addressed (ideally by the PR that resolves it).
- Deep threads get their own `projects/<project>/investigations/<name>.md`,
  linked from the register rather than restated.
- Create `follow-ups.md` on the first merge if absent — seed it from
  `Templates/Reference Doc.md` (`type: reference`, tags
  `[reference, process, follow-ups]`).
- This is enforced across **every** project by Phi's advisor stewardship
  (`personas/phi/advisor-guidance.md`): Phi rides every OMA session, so a PR
  merge in any project prompts the capture automatically.

## General tasks (not tied to a project)

The vault root holds one **General Tasks** list at `General Tasks.md` for
cross-cutting to-dos that relate to **no existing project** — e.g. shell-profile
`PATH` cleanup, publishing a standalone tool. It is **not** an area folder and
has no Templater template; create/extend it directly.

- Frontmatter: `type: tasks`, `tags: [tasks]`, `created: <YYYY-MM-DD>`.
- Body is a table with **`# | Task | Created | Resolved | Status`** columns.
  Record the **Created** date when a task is added; on completion fill
  **Resolved** and set **Status** to `Done` (leave `Open` until then).

**Routing — decide by project attachment, not by feel.** A task or follow-up
that touches an existing project — its repo, tool, daemon, spec, or docs —
belongs in that project's `projects/<project>/follow-ups.md` (deep threads get a
linked `investigations/` note), **even when it looks like generic
"infrastructure."** `General Tasks.md` is **only** for work that no project owns.
Decision test: *can you name the project it belongs to?* If yes, it is not a
general task. In particular, the OMA entity / daemon / vault / advisor machinery
**is** the **oh-my-pi-agent** project — its findings route to
`projects/oh-my-pi-agents/follow-ups.md`, never to General Tasks.
